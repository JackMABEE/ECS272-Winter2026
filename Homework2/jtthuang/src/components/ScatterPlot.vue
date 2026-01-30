<script setup lang="ts">
import { defineProps, onMounted, ref, watchEffect } from 'vue';
import * as d3 from 'd3';

const props = defineProps<{
  data: any[];
}>();

const chartContainer = ref(null);

const drawChart = () => {
  if (!props.data || props.data.length === 0) return;

  // 1. Setup Dimensions
  const margin = { top: 40, right: 150, bottom: 50, left: 60 };
  const width = 800 - margin.left - margin.right;
  const height = 450 - margin.top - margin.bottom;

  // Clear previous
  const container = d3.select(chartContainer.value);
  container.selectAll('*').remove();

  const svg = container.append('svg')
    .attr('width', width + margin.left + margin.right)
    .attr('height', height + margin.top + margin.bottom)
    .append('g')
    .attr('transform', `translate(${margin.left},${margin.top})`);

  // 2. Filter Data (Remove extreme outliers for better visibility)
  // Let's only look at books with pages < 1000 to keep the chart readable
  const filteredData = props.data.filter(d => d.pages < 1200 && d.pages > 0);

  // 3. Scales
  // X-axis: Page Count
  const x = d3.scaleLinear()
    .domain([0, d3.max(filteredData, d => d.pages) as number])
    .range([0, width]);

  // Y-axis: Rating (usually 1 to 5)
  const y = d3.scaleLinear()
    .domain([2.5, 5]) // Start from 2.5 because most books are rated 3-5
    .range([height, 0]);

  // Color Scale: Genre
  const genres = Array.from(new Set(filteredData.map(d => d.genre)));
  const color = d3.scaleOrdinal()
    .domain(genres)
    .range(d3.schemeTableau10); // Nice standard colors

  // 4. Draw Circles (Dots)
  svg.append('g')
    .selectAll('circle')
    .data(filteredData)
    .enter()
    .append('circle')
    .attr('cx', d => x(d.pages))
    .attr('cy', d => y(d.rating))
    .attr('r', 5)
    .style('fill', d => color(d.genre) as string)
    .style('opacity', 0.7)
    .style('stroke', 'white');

  // 5. Add Axes
  svg.append('g')
    .attr('transform', `translate(0,${height})`)
    .call(d3.axisBottom(x));

  svg.append('g')
    .call(d3.axisLeft(y));

  // 6. Labels
  // Title
  svg.append('text')
    .attr('x', width / 2)
    .attr('y', -10)
    .attr('text-anchor', 'middle')
    .style('font-size', '16px')
    .style('font-weight', 'bold')
    .text('Book Length vs. Rating');

  // X Label
  svg.append('text')
    .attr('x', width / 2)
    .attr('y', height + 40)
    .attr('text-anchor', 'middle')
    .text('Page Count');

  // Y Label
  svg.append('text')
    .attr('transform', 'rotate(-90)')
    .attr('y', -40)
    .attr('x', -height / 2)
    .attr('text-anchor', 'middle')
    .text('Average Rating');

  // 7. Legend (Right Side)
  // Only show top 10 genres in legend to save space
  const topGenres = genres.slice(0, 10); 
  
  const legend = svg.append('g')
    .attr('font-family', 'sans-serif')
    .attr('font-size', 10)
    .attr('text-anchor', 'start')
    .selectAll('g')
    .data(topGenres)
    .enter().append('g')
    .attr('transform', (d, i) => `translate(${width + 10},${i * 20})`);

  legend.append('rect')
    .attr('x', 0)
    .attr('width', 15)
    .attr('height', 15)
    .attr('fill', d => color(d) as string);

  legend.append('text')
    .attr('x', 20)
    .attr('y', 9.5)
    .attr('dy', '0.32em')
    .text(d => d as string);
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
}
</style>