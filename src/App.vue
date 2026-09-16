<script setup lang="ts">
import { ref, computed } from 'vue'


import '@vue-flow/core/dist/style.css'
import '@vue-flow/core/dist/theme-default.css'

// these components are only shown as examples of how to use a custom node or edge
// you can find many examples of how to create these custom components in the examples page of the docs
import Home from './Home.vue'
import Graph from './Graph.vue'
import Chart from './Chart.vue'
import Topology from './topology/topology.vue'


const routes: Record<string, unknown> = {
  '/': Home,
  '/graph': Graph,
  '/chart': Chart,
  '/topology': Topology,
}

const currentPath = ref(window.location.hash)

window.addEventListener('hashchange', () => {
  currentPath.value = window.location.hash
});

const currentView = computed(() => {
  return (routes as Record<string, unknown>)[(currentPath.value).slice(1) || '/']
})

</script>


<template>
  <div id="container">
    <a href="#/"> Home </a> |
    <a href="#/graph"> Graph</a> |
    <a href="#/chart"> Chart</a> |
    <a href="#/topology"> Topology</a>
    <component :is="currentView"></component>
  </div>
</template>

<style>
/* import the necessary styles for Vue Flow to work */
@import '@vue-flow/core/dist/style.css';

/* import the default theme, this is optional but generally recommended */
@import '@vue-flow/core/dist/theme-default.css';

#container {
  position: absolute;
  height: 100%;
  width: 100%;
}
</style>