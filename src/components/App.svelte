<script>
    import { onMount } from "svelte";
    import * as d3 from "d3";

    let data = [];
    let select_5g = false;
    let select_nfc = false;
    let select_ir = false;

    onMount(async () => {
        const csvUrl =
            "https://raw.githubusercontent.com/tianyich/smartphonePriceEngine/main/data/smartphone_cleaned.csv";
        const raw = await d3.csv(csvUrl);
        data = processData(data);
        data = raw.map((d) => +d.price);
        createHistogram(data);
    });

    function processData(data) {
        return data;
    }

    function createHistogram(data) {
        const svg = d3.select(svgElement);
        const margin = { top: 10, right: 30, bottom: 30, left: 40 };
        const width = 1400;
        const height = 400;

        const x = d3.scaleLinear().domain([0, 2000]).range([0, width]);

        const histogram = d3.bin().domain(x.domain()).thresholds(x.ticks(50))(
            data,
        );

        var tooltip = d3
            .select("#graph")
            .append("div")
            .style("opacity", 0)
            .style('position', 'absolute')
            .attr("class", "tooltip")
            .style("background-color", "black")
            .style("color", "white")
            .style("border-radius", "5px")
            .style("padding", "10px");

        // A function that change this tooltip when the user hover a point.
        // Its opacity is set to 1: we can now see it. Plus it set the text and position of tooltip depending on the datapoint (d)
        var showTooltip = function (d, i) {

            console.log(i);
            d3.select(this).attr('style', 'fill: orange;');
            tooltip.transition().duration(100).style("opacity", 1);
            tooltip
                .text("Range: " + i.x0 + " - " + i.x1)
                .style("left", d.pageX + 20 + "px")
                .style("top", d.pageY + "px");
        };
        var moveTooltip = function (d,i) {
            tooltip
                .style("left", d.pageX + 20 + "px")
                .style("top", d.pageY + "px");
        };
        // A function that change this tooltip when the leaves a point: just need to set opacity to 0 again
        var hideTooltip = function (d,i) {
            d3.select(this).attr('style', 'fill: #69b3a2;');
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
                    .attr("x", -margin.left)
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
            .style("fill", "#69b3a2")
            .on('mouseover', showTooltip)
            .on("mousemove", moveTooltip)
            .on('mouseout', hideTooltip)
            
    }

    let svgElement;
</script>

<main>
    <h1>Smartphone Price Engine</h1>
    <div id="graph">
        <svg bind:this={svgElement} width="1600" height="450"></svg>
    </div>
    
    <div class="selections">
        <label class="container"
            >5G
            <input type="checkbox" />
            <span class="checkmark"></span>
        </label>

        <label class="container"
            >NFC
            <input type="checkbox" />
            <span class="checkmark"></span>
        </label>

        <label class="container"
            >IR Blaster
            <input type="checkbox" />
            <span class="checkmark"></span>
        </label>
    </div>
</main>

<style>
    h1 {
        text-align: center;
    }
    .selections {
        display: flex;
        justify-content: space-evenly;
        align-items: center;
        padding: 10px;
    }
    .container {
        display: block;
        position: relative;
        padding-left: 35px;
        margin-bottom: 12px;
        cursor: pointer;
        font-size: 22px;
        -webkit-user-select: none;
        -moz-user-select: none;
        -ms-user-select: none;
        user-select: none;
    }

    /* Hide the browser's default radio button */
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

    .container checkmark:after {
        top: 9px;
        left: 9px;
        width: 8px;
        height: 8px;
        border-radius: 50%;
        background: white;
    }
</style>
