<script setup lang="ts">
import { defineProps, onMounted, computed, ref, watchEffect } from 'vue';
import * as d3 from 'd3';

// Receive the data from App.vue
const props = defineProps<{
  data: any[];
}>();

const chartContainer = ref(null);

// This function draws the chart
const drawChart = () => {
  if (!props.data || props.data.length === 0) return;

  // 1. Process Data: Count books per Genre
  const genreCounts = d3.rollups(
    props.data,
    (v) => v.length,
    (d) => d.genre
  )
  .sort((a, b) => b[1] - a[1]) // Sort descending
  .slice(0, 10); // Take Top 10

  // 2. Setup Dimensions
  const margin = { top: 40, right: 30, bottom: 40, left: 150 };
  const width = 600 - margin.left - margin.right;
  const height = 400 - margin.top - margin.bottom;

  // Clear previous chart if exists
  d3.select(chartContainer.value).selectAll('*').remove();

  // 3. Create SVG
  const svg = d3.select(chartContainer.value)
    .append('svg')
    .attr('width', width + margin.left + margin.right)
    .attr('height', height + margin.top + margin.bottom)
    .append('g')
    .attr('transform', `translate(${margin.left},${margin.top})`);

  // 4. Scales
  const x = d3.scaleLinear()
    .domain([0, d3.max(genreCounts, d => d[1]) as number])
    .range([0, width]);

  const y = d3.scaleBand()
    .domain(genreCounts.map(d => d[0]))
    .range([0, height])
    .padding(0.2);

  // 5. Draw Bars
  svg.selectAll('rect')
    .data(genreCounts)
    .enter()
    .append('rect')
    .attr('x', x(0))
    .attr('y', d => y(d[0]) as number)
    .attr('width', d => x(d[1]))
    .attr('height', y.bandwidth())
    .attr('fill', '#69b3a2');

  // 6. Add Axes
  svg.append('g')
    .call(d3.axisLeft(y));

  svg.append('g')
    .attr('transform', `translate(0,${height})`)
    .call(d3.axisBottom(x));

  // 7. Add Title
  svg.append('text')
    .attr('x', width / 2)
    .attr('y', -10)
    .attr('text-anchor', 'middle')
    .style('font-size', '16px')
    .style('font-weight', 'bold')
    .text('Top 10 Genres by Number of Books');
};

// Redraw when data changes
watchEffect(() => {
  drawChart();
});
</script>

<template>
  <div ref="chartContainer" class="chart-container"></div>
</template>

<style scoped>
.chart-container {
  display: flex;
  justify-content: center;
}
</style>