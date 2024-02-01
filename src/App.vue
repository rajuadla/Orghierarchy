<script setup>
import { RouterView } from 'vue-router'


const chartOptions = {
  chart: {
    type: 'spline'
  },
  title: {
    text: 'Sin chart'
  },
  series: [{
    data: [10, 0, 8, 2, 6, 4, 5, 5],
    color: '#6fcd98'
  }]
};


</script>

<template>
  <RouterView />
</template>
