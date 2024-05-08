<template>
    <div>
      <h2>Vue.js and D3 Word Cloud</h2>
      <svg ref="svg"></svg>
    </div>
  </template>
  
  <script>
  import * as d3 from "d3";
  import cloud from "d3-cloud";
  
  export default {
    data() {
      return {};
    },
    mounted() {
      const width = 800;
      const height = 500;
      const data = [
        { text: "Lorem", size: 30 },
        { text: "ipsum", size: 20 },
        { text: "dolor", size: 25 },
        { text: "sit", size: 35 },
        { text: "amet", size: 40 },
        { text: "consectetur", size: 30 },
        { text: "adipiscing", size: 45 },
        { text: "elit", size: 50 },
        { text: "sed", size: 25 },
        { text: "do", size: 30 },
      ];
  
      const svg = d3.select(this.$refs.svg)
        .attr("width", width)
        .attr("height", height);
        
      const layout = cloud()
        .size([width, height])
        .words(data.map(d => ({text: d.text, size: d.size})))
        .padding(5)
        .rotate(() => (Math.random() < 0.5 ? 0 : 90))
        .font("Impact")
        .fontSize(d => d.size)
        .on("end", draw);
  
      layout.start();
  
      function draw(words) {
        svg.append("g")
          .attr("transform", `translate(${width / 2},${height / 2})`)
          .selectAll("text")
          .data(words)
          .enter().append("text")
          .style("font-size", d => `${d.size}px`)
          .style("font-family", "Impact")
          .style("fill", (d, i) => d3.schemeCategory10[i % 10])
          .attr("text-anchor", "middle")
          .attr("transform", d => `translate(${[d.x, d.y]})rotate(${d.rotate})`)
          .text(d => d.text);
      }
    },
  };
  </script>
  