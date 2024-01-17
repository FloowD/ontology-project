<script setup>
import GraphStatic from './components/GraphStatic.vue'
import Graph from './components/Graph.vue'
import {ref} from 'vue'
// const instrumentOntology = JSON.parse(jsonOntology)


</script>

<template>
  <header>
    <h1>Ontologie sur les instruments de musiques</h1>
    <div>
      <!-- {{ instrumentOntology }} -->
      <!-- <GraphStatic /> -->
      <Graph />
    </div>
  </header>

  <main>
   
  </main>
</template>

<style scoped>
  header {
    color: #fff;
    padding: 1rem;
    text-align: center;

  }
  h1 {
    color: black;
  }
</style>
