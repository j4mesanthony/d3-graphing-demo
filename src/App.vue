<script setup>
  import * as d3 from "d3";
  import { useTemplateRef, onMounted, reactive } from "vue";

  const svgElement = useTemplateRef('ref-id');
  const isDarkMode = window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches;

  // Declare the chart dimensions and margins.
  const width = window.innerWidth;
  const height = window.innerHeight;
  const chartWidth = width - 100;
  const chartHeight = height - 200;
  const marginTop = 20;
  const marginRight = 20;
  const marginBottom = 30;
  const marginLeft = 40;

  const data = [
    { date: new Date("2024-04-15"), max: 92.98, min: 82.52 },
    { date: new Date("2024-04-16"), max: 92.98, min: 82.52 },
    { date: new Date("2024-04-17"), max: 92.98, min: 82.52 },
    { date: new Date("2024-04-18"), max: 91.35, min: 84.52 },
    { date: new Date("2024-04-19"), max: 99.84, min: 87.52 },
    { date: new Date("2024-04-20"), max: 97.42, min: 80.52 },
    { date: new Date("2024-04-21"), max: 99.4, min: 89.52 },
    { date: new Date("2024-04-22"), max: 98.92, min: 86.52 },
    { date: new Date("2024-04-23"), max: 93.24, min: 88.52 },
    { date: new Date("2024-04-24"), max: 94.46, min: 82.52 },
    { date: new Date("2024-04-25"), max: 98.84, min: 81.52 },
    { date: new Date("2024-04-26"), max: 99.92, min: 86.52 },
    { date: new Date("2024-04-27"), max: 99.98, min: 84.52 },
    { date: new Date("2024-04-28"), max: 100, min: 91.52 },
  ]

  function RenderGraph() {
    const x = d3.scaleUtc(d3.extent(data, d => d.date), [marginLeft, chartWidth - marginRight]);

    const y = d3.scaleLinear([50, 105], [height - marginBottom, marginTop]);
        
    const svg = d3.create("svg")
        .attr("width", chartWidth)
        .attr("height", chartHeight);

    svg.append("g")
        .attr("class", "x-axis")
        .attr("transform", `translate(0,${chartHeight - marginBottom})`)
        .attr("color", isDarkMode ? d3.schemePastel1[1] : "black")
        .call(d3.axisBottom(x))
        .call(g => g.selectAll(".tick line").clone()
          .attr("y2", (chartHeight - marginTop - marginBottom) * -1)
          .attr("stroke-opacity", 0))
        // .select(".domain")
        // .attr("stroke", isDarkMode ? d3.schemePastel1[1] : "black")
        // .selectAll("text")
        // .attr("fill", isDarkMode && "white");
        

    svg.append("g")
        .attr("transform", `translate(${marginLeft},0)`)
        .attr("color", isDarkMode ? d3.schemePastel1[1] : "black")
        .call(d3.axisLeft(y))
        .attr("fill", isDarkMode && "white")
        .call(g => g.select(".domain").remove())
        .call(g => g.selectAll(".tick line").clone()
          .attr("x2", chartWidth - marginLeft - marginRight)
          .attr("stroke-opacity", 0.1))
          // .selectAll("text")
          // .attr("fill", isDarkMode && "white");

    const max = d3.line()
      .x(d => x(d.date))
      .y(d => y(d.max));

    const min = d3.line()
      .x(d => x(d.date))
      .y(d => y(d.min));

    const GetInterpolatedColor = interval => isDarkMode ? d3.interpolatePlasma(interval) : d3.interpolatePurples(interval + 0.6);
      
    svg.append("path")
      .attr("fill", "none")
      .attr("stroke", GetInterpolatedColor(0.1))
      .attr("stroke-width", 3)
      .attr("d", max(data));

    svg.append("path")
      .attr("fill", "none")
      .attr("stroke", GetInterpolatedColor(0.2))
      .attr("stroke-width", 1)
      .attr("stroke-dasharray", "3,5")
      .attr("d", min(data));

    svg.selectAll(".symbol")
      .data(data)
      .enter()
      .append("path")
      .attr("class", "symbol")
      .style("cursor", "pointer")
      .on("mouseover", function(event, d) {
        d3.select(this)
          .transition()
          .duration(200)
          .attr("d", d3.symbol().type(d3.symbolCircle).size(180))
          .attr("fill", GetInterpolatedColor(0.9));

          svg.selectAll(".x-axis line")
            .attr("stroke", t => JSON.stringify(t) == JSON.stringify(d.date) ? GetInterpolatedColor(1) : d3.schemePastel1[1])
            .attr("stroke-opacity", t => JSON.stringify(t) == JSON.stringify(d.date) ? 0.6 : 0);
      })
      .on("mouseout", function() {
        d3.select(this)
          .transition()
          .ease(d3.easeBounceOut)
          .duration(600)
          .attr("d", d3.symbol().type(d3.symbolCircle).size(80))
          .attr("fill", GetInterpolatedColor(0.2));

          svg.selectAll(".x-axis line")
            .attr("stroke", d3.schemePastel1[1])
            .attr("stroke-opacity", 0);
      })
      .attr("d", d3.symbol().type(d3.symbolCircle).size(80))
      .attr("transform", d => `translate(${x(d.date)},${y(d.max)})`)
      .attr("fill", GetInterpolatedColor(0.2));

    svgElement.value.appendChild(svg.node());
  }

  onMounted(RenderGraph);
</script>

<template>
  <div class="p-10 dark:bg-gray-950">
    <h1 class="text-3xl dark:text-white">D3 Graphing</h1>
    <p class="mb-8 text-lg text-gray-500">Multi-Axis | <span class=" small">Demo</span></p>
    <div ref="ref-id"></div>
  </div>
</template>


