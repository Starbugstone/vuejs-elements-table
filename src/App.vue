<script setup lang="ts">
import DataTable from './components/DataTable.vue'
</script>

<template>
  <div class="app-container">
    <h1>Data Table</h1>
    <DataTable />
  </div>
</template>

<style>
.app-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

body {
  margin: 0;
  min-height: 100vh;
  background-color: #f0f2f5;
}
</style>