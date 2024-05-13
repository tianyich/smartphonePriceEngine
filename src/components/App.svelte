<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    onMount(async () => {
    const rawData = d3.csv('./data/smartphone_cleaned.csv');
    data = rawData.map(d => +d.price);

    create_hist(data);
    });
    function create_hist(data){
        const svg = d3.select(svgElement);
        const margin = {top: 10, right: 30, bottom: 30, left: 40};
        const width = 450 - margin.left - margin.right;
        const height = 400 - margin.top - margin.bottom;
        const bins = d3.bin()
        .thresholds(40)
        .value((d) => d.price)
        (data);
        const y = d3.scaleLinear()
               .range([height, 0])
               .domain([0, d3.max(histogram, d => d.length)]);
        svg.append("g")
       .attr("transform", `translate(${margin.left},${margin.top})`);

       svg.selectAll("rect")
        .data(histogram)
        .enter()
        .append("rect")
        .attr("x", d => x(d.x0))
        .attr("width", d => x(d.x1) - x(d.x0) - 1)
        .attr("y", d => y(d.length))
        .attr("height", d => height - y(d.length))
        .style("fill", "#69b3a2");
    }
    let svgElement;
</script>

<main>
    <svg bind:this={svgElement} width="500" height="500"></svg>
</main>

<style>

</style>