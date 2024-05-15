<script>
    import { onMount } from "svelte";
    import * as d3 from "d3";
    import Apple from "../imgs/apple.svelte";
    import Asus from "../imgs/asus.svelte";
    import Google from "../imgs/google.svelte";
    import Honor from "../imgs/honor.svelte";
    import Huawei from "../imgs/huawei.svelte";
    import Iqoo from "../imgs/iqoo.svelte";
    import Motorola from "../imgs/motorola.svelte";
    import Nokia from "../imgs/nokia.svelte";
    import Nothing from "../imgs/nothing.svelte";
    import Oneplus from "../imgs/oneplus.svelte";
    import Poco from "../imgs/poco.svelte";
    import Samsung from "../imgs/samsung.svelte";
    import Sony from "../imgs/sony.svelte";
    import Vivo from "../imgs/vivo.svelte";
    import Xiaomi from "../imgs/xiaomi.svelte";

    let data = [];
    let select_5g = false;
    let select_nfc = false;
    let select_ir = false;
    let select_Apple = false;
    let select_Asus = false;
    let select_Google = false;
    let select_Honor = false;
    let select_Huawei = false;
    let select_Iqoo = false;
    let select_Motorola = false;
    let select_Nokia = false;
    let select_Nothing = false;
    let select_Oneplus = false;
    let select_Poco = false;
    let select_Samsung = false;
    let select_Sony = false;
    let select_Vivo = false;
    let select_Xiaomi = false;
    let buttons = {
        "Apple": select_Apple,
        "Asus": select_Asus,
        "Google": select_Google,
        "Honor": select_Honor,
        "Huawei": select_Huawei,
        "Iqoo": select_Iqoo,
        "Motorola": select_Motorola,
        "Nokia": select_Nokia,
        "Nothing": select_Nothing,
        "Oneplus": select_Oneplus,
        "Poco": select_Poco,
        "Samsung": select_Samsung,
        "Sony": select_Sony,
        "Vivo": select_Vivo,
        "Xiaomi": select_Xiaomi,
    }
    
    let raw;
    onMount(async () => {
        const csvUrl =
            "https://raw.githubusercontent.com/tianyich/smartphonePriceEngine/main/data/smartphone_cleaned.csv";
        raw = await d3.csv(csvUrl);
        data = processData(raw);
        createHistogram(data);
    });

    function processData(data) {
        if (select_nfc) {
            data = data.filter((d) => d["has_nfc"] == "True");
        }
        if (select_5g) {
            data = data.filter((d) => d["has_5g"] == "True");
        }
        if (select_ir) {
            data = data.filter((d) => d["has_ir_blaster"] == "True");
        }
        let selected_brands = []
        for (const brand in buttons) {
            if (buttons[brand]) {
                selected_brands.push(brand.toLowerCase());
            }
        }
        if (selected_brands.length > 0) {
            data = data.filter((d) => selected_brands.includes(d["brand_name"].toLowerCase()));
        }
        if (select_memory != "") {
            if (select_memory == "< 4GB") {
                console.log("here");
                data = data.filter((d) => d["ram_capacity"] < 4);
                console.log(data);
            } else if (select_memory == "4GB") {
                data = data.filter((d) => +d["ram_capacity"] == 4);
            } else if (select_memory == "6GB") {
                data = data.filter((d) => +d["ram_capacity"] == 6);
            } else if (select_memory == "8GB") {
                data = data.filter((d) => +d["ram_capacity"] == 8);
            } else if (select_memory == "12GB") {
                data = data.filter((d) => +d["ram_capacity"] == 12);
            } else if (select_memory == "16GB") {
                data = data.filter((d) => +d["ram_capacity"] == 16);
            } else if (select_memory == "> 16GB") {
                data = data.filter((d) => +d["ram_capacity"] > 16);
            }
        }
        if (select_storage != "") {
            if(select_storage) {
                if (select_storage == "16GB") {
                    data = data.filter((d) => +d["internal_memory"] == 16);
                } else if (select_storage == "32GB") {
                    data = data.filter((d) => +d["internal_memory"] == 32);
                } else if (select_storage == "64GB") {
                    data = data.filter((d) => +d["internal_memory"] == 64);
                } else if (select_storage == "128GB") {
                    data = data.filter((d) => +d["internal_memory"] == 128);
                } else if (select_storage == "256GB") {
                    data = data.filter((d) => +d["internal_memory"] == 256);
                } else if (select_storage == "512GB") {
                    data = data.filter((d) => +d["internal_memory"] == 512);
                } else if (select_storage == "> 512GB") {
                    data = data.filter((d) => +d["internal_memory"] > 512);
                }
            }
        }
        if (select_refresh != "") {
            if (select_refresh == "60Hz") {
                data = data.filter((d) => +d["refresh_rate"] == 60);
            } else if (select_refresh == "90Hz") {
                data = data.filter((d) => +d["refresh_rate"] == 90);
            } else if (select_refresh == "120Hz") {
                data = data.filter((d) => +d["refresh_rate"] == 120);
            } else if (select_refresh == "144Hz") {
                data = data.filter((d) => +d["refresh_rate"] == 144);
            } else if (select_refresh == "165Hz") {
                data = data.filter((d) => +d["refresh_rate"] == 165);
            }
        }
        if (select_core != "") {
            data = data.filter((d) => d["num_cores"] == select_core);
        }
        
        return data;
    }

    function createHistogram(data) {
        let price = data.map((d) => +d.price);
        const svg = d3.select(svgElement);
        const margin = {top: 10, right: 30, bottom: 30, left: 40 };
        const width = 1400;
        const height = 400;

        const x = d3.scaleLinear().domain([0, 2000]).range([0, width]);

        const histogram = d3.bin().domain(x.domain()).thresholds(x.ticks(50))(
            price,
        );

        var tooltip = d3
            .select("#graph")
            .append("div")
            .style("opacity", 0)
            .style("position", "absolute")
            .attr("class", "tooltip")
            .style("background-color", "black")
            .style("color", "white")
            .style("border-radius", "5px")
            .style("padding", "10px");

        var showTooltip = function (d, i) {
            d3.select(this).attr("style", "fill: orange;").style("stroke", "gray");
            tooltip.transition().duration(100).style("opacity", 1);
            tooltip
                .text("Range: " + "$"+i.x0 + " - " + "$"+i.x1)
                .style("left", d.pageX + 20 + "px")
                .style("top", d.pageY + "px");
        };
        var moveTooltip = function (d, i) {
            tooltip
                .style("left", d.pageX + 20 + "px")
                .style("top", d.pageY + "px");
        };

        var hideTooltip = function (d, i) {
            d3.select(this).attr("style", "fill: #69b3a2;").style("stroke", "gray");;
            tooltip.transition().duration(100).style("opacity", 0);
        };

        const y = d3
            .scaleLinear()
            .range([height, 0])
            .domain([0, d3.max(histogram, (d) => d.length)]);

        svg.append("g")
            .attr("transform", `translate(150, ${height + 10})`)
            .call(d3.axisBottom(x).ticks(50).tickSizeOuter(0))
            .call((g) =>
                g
                    .append("text")
                    .attr("x", width)
                    .attr("y", margin.bottom - 4)
                    .attr("fill", "currentColor")
                    .attr("text-anchor", "end")
                    .text("Price"),
            );

        svg.append("g")
            .attr("transform", `translate(150,0)`)
            .call(d3.axisLeft(y).ticks(height / 40))
            .call((g) => g.select(".domain").remove())
            .call((g) =>
                g
                    .append("text")
                    .attr("x", -margin.left - 30)
                    .attr("y", 10)
                    .attr("fill", "currentColor")
                    .attr("text-anchor", "start")
                    .text("Frequency"),
            );

        svg.selectAll("rect")
            .data(histogram)
            .enter()
            .append("rect")
            .attr("x", (d) => x(d.x0))
            .attr("width", (d) => x(d.x1) - x(d.x0))
            .attr("y", (d) => y(d.length))
            .attr("height", (d) => height - y(d.length))
            .attr("transform", `translate(150, 0)`)
            .on("mouseover", showTooltip)
            .on("mousemove", moveTooltip)
            .on("mouseout", hideTooltip)
            .on("click", function (d,i) {
                selected = true;
                let phonesInBin = data.filter((x) => x['price'] >= i.x0 && x['price'] < i.x1);
                phonesInBin = phonesInBin.filter((x) => x['model'] !== undefined);
                phonesInBin.sort((a, b) => b.rating - a.rating);
                modelNames = phonesInBin.map((x) => x['model']).slice(0, 20);
                
            })
            .style("fill", "#69b3a2")
            .style("stroke", "gray");
    }
    var UpdateHistogram = function() {
        const svg = d3.select(svgElement);
        console.log("UpdateHistogram");
        data = processData(raw);
        svg.selectAll("rect").remove();
        svg.selectAll("g").remove();
        createHistogram(data);
        selected = false;
    }

    let svgElement;
    let select_memory = "";
    const memories = [
        "< 4GB",
        "4GB",
        "6GB",
        "8GB",
        "12GB",
        "16GB",
        "> 16GB",
    ];
    let select_refresh = "";
    const refresh_rates = [
        "60Hz",
        "90Hz",
        "120Hz",
        "144Hz",
        "165Hz",
    ];
    let select_storage = "";
    const storages = [
        "16GB",
        "32GB",
        "64GB",
        "128GB",
        "256GB",
        "512GB",
        "> 512GB",
    ];
    let select_core = "";
    const cores = [
        "Quad Core",
        "Hexa Core",
        "Octa Core",
    ];
    let modelNames = [];
    let selected = false;

</script>

<main>
    <body>
    <h1>Smartphone Price Engine</h1>
    <div id="graph">
        <svg bind:this={svgElement} width="1600" height="450"></svg>
    </div>
    <div align="center">
        {#if selected}<h2>Best Rating Models for You </h2>{/if}
        {#if !selected}<h2>Please Select the Price Range</h2>{/if}
        {#if selected}<table class = "top_picks">
            <tbody>
                {#each Array(Math.ceil(modelNames.length / 5)) as _, i (i)}
                    <tr>
                        {#if modelNames[i * 5]}<td>{modelNames[i * 5]}</td>{/if}
                        {#if modelNames[i * 5 + 1]}<td>{modelNames[i * 5 + 1]}</td>{/if}
                        {#if modelNames[i * 5 + 2]}<td>{modelNames[i * 5 + 2]}</td>{/if}
                        {#if modelNames[i * 5 + 3]}<td>{modelNames[i * 5 + 3]}</td>{/if}
                        {#if modelNames[i * 5 + 4]}<td>{modelNames[i * 5 + 4]}</td>{/if}
                    </tr>
                {/each}
            </tbody>
        </table>{/if}
    </div>
    

   

    <div class="selections" align="center">
        <label class="selection">
            Select RAM:
            <select bind:value={select_memory} on:change={UpdateHistogram}>
                <option value="">-- select RAM --</option>
                {#each memories as memory (memory)}
                    <option value={memory}>{memory}</option>
                {/each}
            </select>
        </label>
        <label class="selection">
            Select Refresh Rate:
            <select bind:value={select_refresh} on:change={UpdateHistogram}>
                <option value="">-- select refreshrates --</option>
                {#each refresh_rates as rf (rf)}
                    <option value={rf}>{rf}</option>
                {/each}
            </select>
        </label>
        <label class="selection">
            Select Storages:
            <select bind:value={select_storage} on:change={UpdateHistogram}>
                <option value="">-- select storage --</option>
                {#each storages as storage (storage)}
                    <option value={storage}>{storage}</option>
                {/each}
            </select>
        </label>
        <label class="selection">
            Select Number of Cores:
            <select bind:value={select_core} on:change={UpdateHistogram}>
                <option value="">-- select Number of Core --</option>
                {#each cores as core (core)}
                    <option value={core}>{core}</option>
                {/each}
            </select>
        </label>
            
    </div>

    <div class="selections">
        <label class="container"
            >5G
            <input type="checkbox" bind:checked={select_5g} on:change={UpdateHistogram} />
            <span class="checkmark"></span>
        </label>

        <label class="container"
            >NFC
            <input type="checkbox" bind:checked={select_nfc} on:change={UpdateHistogram} />
            <span class="checkmark"></span>
        </label>

        <label class="container"
            >IR Blaster
            <input type="checkbox" bind:checked={select_ir} on:change={UpdateHistogram}/>
            <span class="checkmark"></span>
        </label>
    </div>
    

    <div class="col-sm-5 grid-container" align="center">
        <button class={buttons['Apple'] ? 'checked' : ''}
        on:click={() => buttons['Apple'] = !buttons['Apple']}
        on:click = {UpdateHistogram}>    
            <Apple></Apple>
        </button>

        <button class={buttons['Asus'] ? 'checked' : ''}
        on:click={() => buttons['Asus'] = !buttons['Asus']}
        on:click = {UpdateHistogram}>    
            <Asus></Asus>
        </button>
        
        <button class={buttons['Google'] ? 'checked' : ''}
        on:click={() => buttons['Google'] = !buttons['Google']}
        on:click = {UpdateHistogram}>    
            <Google></Google>
        </button>

        <button class={buttons['Honor'] ? 'checked' : ''}
        on:click={() => buttons['Honor'] = !buttons['Honor']}
        on:click = {UpdateHistogram}>    
            <Honor></Honor>
        </button>

        <button class={buttons['Huawei'] ? 'checked' : ''}
        on:click={() => buttons['Huawei'] = !buttons['Huawei']}
        on:click = {UpdateHistogram}>    
            <Huawei></Huawei>
        </button>

        <button class={buttons['Iqoo'] ? 'checked' : ''}
        on:click={() => buttons['Iqoo'] = !buttons['Iqoo']}
        on:click = {UpdateHistogram}>    
            <Iqoo></Iqoo>
        </button>

        <button class={buttons['Motorola'] ? 'checked' : ''}
        on:click={() => buttons['Motorola'] = !buttons['Motorola']}
        on:click = {UpdateHistogram}>    
            <Motorola></Motorola>
        </button>

        <button class={buttons['Nokia'] ? 'checked' : ''}
        on:click={() => buttons['Nokia'] = !buttons['Nokia']}
        on:click = {UpdateHistogram}>    
            <Nokia></Nokia>
        </button>

        <button class={buttons['Nothing'] ? 'checked' : ''}
        on:click={() => buttons['Nothing'] = !buttons['Nothing']}
        on:click = {UpdateHistogram}>    
            <Nothing></Nothing>
        </button>

        <button class={buttons['Oneplus'] ? 'checked' : ''}
        on:click={() => buttons['Oneplus'] = !buttons['Oneplus']}
        on:click = {UpdateHistogram}>    
            <Oneplus></Oneplus>
        </button>

        <button class={buttons['Poco'] ? 'checked' : ''}
        on:click={() => buttons['Poco'] = !buttons['Poco']}
        on:click = {UpdateHistogram}>    
            <Poco></Poco>
        </button>
        
        <button class={buttons['Samsung'] ? 'checked' : ''}
        on:click={() => buttons['Samsung'] = !buttons['Samsung']}
        on:click = {UpdateHistogram}>    
            <Samsung></Samsung>
        </button>
        <button class={buttons['Sony'] ? 'checked' : ''}
        on:click={() => buttons['Sony'] = !buttons['Sony']}
        on:click = {UpdateHistogram}>    
            <Sony></Sony>
        </button>
        <button class={buttons['Vivo'] ? 'checked' : ''}
        on:click={() => buttons['Vivo'] = !buttons['Vivo']}
        on:click = {UpdateHistogram}>    
            <Vivo></Vivo>
        </button>
        <button class={buttons['Xiaomi'] ? 'checked' : ''}
        on:click={() => buttons['Xiaomi'] = !buttons['Xiaomi']}
        on:click = {UpdateHistogram}>    
            <Xiaomi></Xiaomi>
        </button>
    </div>
    </body>
</main>

<style>
    body {
        font-family: Arial, sans-serif;
    }
    h1 {
        text-align: center;
    }
    .selections {
        display: flex;
        justify-content: space-evenly;
        align-items: center;
        padding: 10px;
        font-size: 22px;
    }
    .selection select {
        width: 200px;  
        height: 30px;  
        font-size: 18px;
        text-align: center;
        }
    .container {
        display: block;
        position: relative;
        padding-left: 35px;
        margin-bottom: 12px;
        cursor: pointer;
        font-size: 22px;
    }

    .container input {
        position: absolute;
        opacity: 0;
        cursor: pointer;
        height: 0;
        width: 0;

    }

    .checkmark {
        position: absolute;
        top: 0;
        left: 0;
        height: 25px;
        width: 25px;
        background-color: #eee;
        border-radius: 50%;
    }

    .container:hover input ~ .checkmark {
        background-color: #ccc;
    }

    .container input:checked ~ .checkmark {
        background-color: #2196f3;
    }

    .checkmark:after {
        content: "";
        position: absolute;
        display: none;
    }

    .container input:checked ~ .checkmark:after {
        display: block;
    }

    .container .checkmark:after {
        top: 9px;
        left: 9px;
        width: 8px;
        height: 8px;
        border-radius: 50%;
        background: white;
    }

    .grid-container {
        display: grid;
        grid-template-columns: repeat(5, 1fr);
        grid-template-rows: repeat(3, 1fr);
        gap: 10px;
        align-items: center;
        justify-items: center;
    }

    .selection {
        margin: 10px;
    }
    button {
        box-sizing: border-box;
        background-color: Transparent;
        background-repeat: no-repeat;
        border: 0px;
        cursor: pointer;
        overflow: hidden;
        outline: none;
    }
    button.checked {
        box-sizing: border-box;
        background-color: Transparent;
        outline: 2px solid blue;
        cursor: pointer;
        overflow: hidden;
    }
    button.checked:hover {
        box-sizing: border-box;
        background-color: #c3e0f8;
        outline: 2px solid blue;
        cursor: pointer;
        overflow: hidden;
    }
    button:hover {
        background-color: #c3e0f8;
        background-repeat: no-repeat;
        border: none;
        cursor: pointer;
        overflow: hidden;
    }
    top_picks {
        align: center;
        display: grid;
        grid-template-columns: repeat(5, 1fr);
        gap: 10px;
        width: 100%;
    }
    .top_picks td {
        padding: 10px;
        border: 1px solid black;
    }

    
</style>
