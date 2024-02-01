<script setup lang="ts">
import { ref, onMounted, watch } from 'vue';
import { OrgChart } from 'd3-org-chart';

const finalRes = ref();
const modalref = ref(null)

async function getData() {
  const res = await fetch("http://localhost:3000/");
  finalRes.value = await res.json();
};

onMounted(() => {
  getData();
});

watch(finalRes, () => {

  const orgData = JSON.parse(JSON.stringify(finalRes.value));

  new OrgChart()
    .initialExpandLevel(3)
    .svgWidth(500)
    .nodeId((dataItem: any) => dataItem.name)
    .parentNodeId((dataItem: any) => dataItem.parent)
    .nodeWidth((node: any) => 30)
    .nodeHeight((node: any) => 30)
    .nodeContent((node: any) => {
      return `<div 
        style="background-color:aqua;width:${node.width}px;height:${node.height}px"
      > 
           ${node.data.name}
       </div>`;
    })
    .container('.chart-container')
    .layout('left')
    .data(orgData)
    .onNodeClick((dataItem: any) => alert(`${dataItem.data.name} \n${dataItem.data.description}`))
    .render();
})


</script>

<template>
  <div class="chart-container" />
</template>