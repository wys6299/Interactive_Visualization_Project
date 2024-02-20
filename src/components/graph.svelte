<script lang="ts">
    import { onMount, afterUpdate } from 'svelte';
    import * as d3 from 'd3';
    import Slider from './Slider.svelte';

    export let data;

    let svg;
    let circle;
    let xAxis;
    let yAxis;
    let grid;
    let update;
    let selectedYear;
    
    let years = [];

    const margin = { top: 20, right: 20, bottom: 35, left: 40 };
    const width = 960;
    const height = 560;

    let x;
    let y;
    let radius;
    let color;

    const dataAt = year => {
        return data.filter(d => d.Year === year);
    };

    onMount(() => {
        x = d3.scaleLog([200, 1e5], [margin.left, width - margin.right]);
        y = d3.scaleLinear([14, 86], [height - margin.bottom, margin.top]);
        radius = d3.scaleSqrt([0, 5e8], [0, width / 24]);
        color = d3.scaleOrdinal(data.map(d => d.Continent), d3.schemeCategory10).unknown("black");

        const svgNode = d3.select("#chart").append("svg")
            .attr("viewBox", [0, 0, width, height]);

        xAxis = g => g
            .attr("transform", `translate(0,${height - margin.bottom})`)
            .call(d3.axisBottom(x).ticks(width / 80, ","))
            .call(g => g.select(".domain").remove())
            .call(g => g.append("text")
                .attr("x", width)
                .attr("y", margin.bottom - 4)
                .attr("fill", "currentColor")
                .attr("text-anchor", "end")
                .text("GDP →"));

        yAxis = g => g
            .attr("transform", `translate(${margin.left},0)`)
            .call(d3.axisLeft(y))
            .call(g => g.select(".domain").remove())
            .call(g => g.append("text")
                .attr("x", -margin.left)
                .attr("y", 10)
                .attr("fill", "currentColor")
                .attr("text-anchor", "start")
                .text("↑ Death"));

        grid = g => g
            .attr("stroke", "currentColor")
            .attr("stroke-opacity", 0.1)
            .call(g => g.append("g")
                .selectAll("line")
                .data(x.ticks())
                .join("line")
                .attr("x1", d => 0.5 + x(d))
                .attr("x2", d => 0.5 + x(d))
                .attr("y1", margin.top)
                .attr("y2", height - margin.bottom))
            .call(g => g.append("g")
                .selectAll("line")
                .data(y.ticks())
                .join("line")
                .attr("y1", d => 0.5 + y(d))
                .attr("y2", d => 0.5 + y(d))
                .attr("x1", margin.left)
                .attr("x2", width - margin.right));

        svgNode.append("g").call(xAxis);
        svgNode.append("g").call(yAxis);
        svgNode.append("g").call(grid);

        circle = svgNode.append("g")
            .attr("stroke", "black")
            .selectAll("circle")
            .data(dataAt(1991), d => d.Entity)
            .join("circle")
            .sort((a, b) => d3.descending(a.Population, b.Population))
            .attr("cx", d => x(d.GDP))
            .attr("cy", d => y(d.Death))
            .attr("r", d => radius(d.Population))
            .attr("fill", d => color(d.Continent))
            .call(circle => circle.append("title")
                .text(d => [d.Entity, d.Continent].join("\n")));

        update = newData => {
            circle.data(newData, d => d.Entity)
                .sort((a, b) => d3.descending(a.Population, b.Population))
                .attr("cx", d => x(d.GDP))
                .attr("cy", d => y(d.Death))
                .attr("r", d => radius(d.Population));
        };

        // Calculate the range of years from your data
        years = Array.from(new Set(data.map(d => d.Year)));
        console.log(years)

        return svgNode.node();
    });

    afterUpdate(() => {
        circle.data(dataAt(selectedYear), d => d.Entity)
            .sort((a, b) => d3.descending(a.Population, b.Population))
            .attr("cx", d => x(d.GDP))
            .attr("cy", d => y(d.Death))
            .attr("r", d => radius(d.Population))
            .attr("fill", d => color(d.Continent));
    });
</script>

<div id="chart"></div>
<Slider bind:selectedYear />

<style>
  /* Add your styles here */
</style>