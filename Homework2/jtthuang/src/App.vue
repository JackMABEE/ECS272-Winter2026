<script setup lang="ts">
import { ref, onMounted } from 'vue';
import * as d3 from 'd3';
import BarChart from './components/BarChart.vue';
import ScatterPlot from './components/ScatterPlot.vue';
import ParallelCoordinates from './components/ParallelCoordinates.vue';

// Define the structure of our book data
interface Book {
  title: string;
  genre: string;
  year: number;
  pages: number;
  rating: number;
}

const books = ref<Book[]>([]);

onMounted(async () => {
  try {
    const rawData = await d3.csv('/data/books.csv');

    // Clean and Format the Data
    books.value = rawData.map((d: any) => ({
      title: d.title,
      genre: d.genre,
      year: +d.publicationYear, // Convert string "2005" to number
      pages: +d.pageCount,      // Convert string "300" to number
      rating: +d.rating_average // Convert string "4.5" to number
    })).filter(d => d.genre && d.year > 0); // Remove rows with missing info

    console.log("Data ready for charts:", books.value[0]);
  } catch (error) {
    console.error("Error loading CSV:", error);
  }
});
</script>

<template>
  <VContainer fluid>
    <VRow>
      <VCol cols="12" class="text-center">
        <h1>Book Exchange Dashboard</h1>
        <p>Homework 2 - Focus + Context Visualization</p>
      </VCol>
    </VRow>

    <div v-if="books.length > 0">
      <VRow>
        <VCol cols="12">
          <BarChart :data="books" />
        </VCol>
      </VRow>

      <VRow>
        <VCol cols="12">
          <ScatterPlot :data="books" />
        </VCol>
      </VRow>

      <VRow>
        <VCol cols="12">
          <ParallelCoordinates :data="books" />
        </VCol>
      </VRow>
    </div>

    <VRow v-else>
      <VCol cols="12">
        <p>Loading data...</p>
      </VCol>
    </VRow>
  </VContainer>
</template>