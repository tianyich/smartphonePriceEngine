<script>
    import { onMount } from "svelte";
    import * as d3 from "d3";

    let data = [];

    onMount(async () => {
        const csvUrl =
            "https://raw.githubusercontent.com/tianyich/smartphonePriceEngine/main/data/smartphone_cleaned.csv";
        const raw = await d3.csv(csvUrl);
        data = raw.map(d => +d.price);
        console.log(data)
        createHistogram(data);
    });


    function createHistogram(data) {
        const svg = d3.select(svgElement);
        const margin = { top: 10, right: 30, bottom: 30, left: 40 };
        const width = 4000;
        const height = 400;

        const x = d3
            .scaleLinear()
            .domain([0, d3.max(data)])
            .range([0, width]);

        const histogram = d3.bin().domain(x.domain()).thresholds(x.ticks(200))(
            data,
        );

        const y = d3
            .scaleLinear()
            .range([height, 0])
            .domain([0, d3.max(histogram, (d) => d.length)]);

        svg.append("g")
            .attr("transform", `translate(100, ${height + 10})`)
            .call(d3.axisBottom(x).ticks(100).tickSizeOuter(0))
            .call((g) => g.append("text")
            .attr("x", width)
            .attr("y", margin.bottom -4)
            .attr("fill", "currentColor")
            .attr("text-anchor", "end")
            .text("Unemployment rate (%) →"));
        
        svg.selectAll("rect")
            .data(histogram)
            .enter()
            .append("rect")
            .attr("x", (d) => x(d.x0))
            .attr("width", (d) => x(d.x1) - x(d.x0))
            .attr("y", (d) => y(d.length))
            .attr("height", (d) => height - y(d.length))
            .attr("transform", `translate(100, 0)`)
            .style("fill", "#69b3a2");
    }

    let svgElement;
</script>


<main>
    <h1> Smartphone Price Engine</h1>
    <svg bind:this={svgElement} width="1000" height="1000"></svg>
</main>

<style>
    .h1 {
        text-align: center;
    }
</style>
