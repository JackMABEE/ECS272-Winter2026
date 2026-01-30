<script setup lang="ts">
import { defineProps, onMounted, ref, watchEffect } from 'vue';
import * as d3 from 'd3';

const props = defineProps<{
  data: any[];
}>();

const chartContainer = ref(null);

const drawChart = () => {
  if (!props.data || props.data.length === 0) return;

  // 1. Select Dimensions to Visualize
  // We want to compare these three numeric columns
  const dimensions = ['year', 'pages', 'rating'];

  // 2. Setup Size
  const margin = { top: 100, right: 50, bottom: 20, left: 50 };
  const width = 800 - margin.left - margin.right;
  const height = 400 - margin.top - margin.bottom;

  // Clear previous
  const container = d3.select(chartContainer.value);
  container.selectAll('*').remove();

  const svg = container.append('svg')
    .attr('width', width + margin.left + margin.right)
    .attr('height', height + margin.top + margin.bottom)
    .append('g')
    .attr('transform', `translate(${margin.left},${margin.top})`);

  // 3. Filter Data
  // Let's filter slightly to remove extreme outliers (e.g. year < 1900) for a cleaner look
  const subset = props.data.filter(d => d.year > 1950 && d.pages < 1500).slice(0, 200); 
  // (We slice(0, 200) to keep the chart from getting too messy with 1000 lines)

  // 4. Build Y Scales (One for each dimension)
  const y = {};
  dimensions.forEach(name => {
    y[name] = d3.scaleLinear()
      .domain(d3.extent(subset, d => d[name]) as [number, number]) // Min to Max
      .range([height, 0]); // Bottom to Top
  });

  // 5. Build X Scale (Position of the axes)
  const x = d3.scalePoint()
    .range([0, width])
    .padding(1)
    .domain(dimensions);

  // 6. Color Scale (By Genre)
  const color = d3.scaleOrdinal()
    .domain(subset.map(d => d.genre))
    .range(d3.schemeTableau10);

  // 7. Draw Lines (The Paths)
  // Function to calculate the path for a single book
  const path = (d) => {
    return d3.line()(dimensions.map(p => [x(p), y[p](d[p])]));
  };

  svg.selectAll('myPath')
    .data(subset)
    .enter().append('path')
    .attr('d', path)
    .style('fill', 'none')
    .style('stroke', d => color(d.genre) as string)
    .style('opacity', 0.5)
    .style('stroke-width', 1.5);

  // 8. Draw Axes
  svg.selectAll('myAxis')
    .data(dimensions).enter()
    .append('g')
    .attr('transform', d => `translate(${x(d)})`)
    .each(function(d) { d3.select(this).call(d3.axisLeft(y[d])); })
    .append('text')
    .style('text-anchor', 'middle')
    .attr('y', -9)
    .text(d => d)
    .style('fill', 'black')
    .style('font-weight', 'bold')
    .style('text-transform', 'capitalize');

  // Title
  svg.append('text')
    .attr('x', width / 2)
    .attr('y', -20)
    .attr('text-anchor', 'middle')
    .style('font-weight', 'bold')
    .text('Multivariate Analysis (Parallel Coordinates)');
};

watchEffect(() => {
  drawChart();
});
</script>

<template>
  <div ref="chartContainer" class="chart-wrapper"></div>
</template>

<style scoped>
.chart-wrapper {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}
</style>