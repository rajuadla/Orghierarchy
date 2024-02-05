<script setup lang="ts">
import { Ref, ref, onMounted, watch, defineComponent } from 'vue';
import { OrgChart } from 'd3-org-chart';
import { Organizaion } from '@/types/Organization';


const finalRes: Ref<Organizaion[]> = ref();
const selectedNode: Ref<Organizaion> = ref(null);

async function getData() {
  const res = await fetch("http://localhost:3000/");
  finalRes.value = await res.json();
};

onMounted(() => {
  getData();
});

function onCardSelection(chatReference: OrgChart, data: Organizaion): any {
  const attrs = chatReference.getChartState();
  const nodes = attrs.allNodes.map((d: any) => {
    if (attrs.nodeId(d.data) === data.name) {
      d.data._highlighted = true;
      selectedNode.value = data;
    }
    return d;
  })
  chatReference.data(nodes);
  chatReference.update(attrs);
}

watch(finalRes, () => {

  const chatReference = new OrgChart()
    .initialExpandLevel(3)
    .svgWidth(500)
    .nodeId((dataItem: Organizaion) => dataItem.name)
    .parentNodeId((dataItem: Organizaion) => dataItem.parent)
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
    .data(finalRes.value)
    .setActiveNodeCentered(false)
    .compact(false)
    .onNodeClick((dataItem: { data: Organizaion }) => onCardSelection(chatReference, dataItem.data))
    .render();

})
</script>

<template>
  <div id="overlay">
    <div v-if="selectedNode">
      {{ finalRes }}</div>
  </div>
  <div class="chart-container" />
</template>

<style>
#overlay {
  position: fixed;
  /* Sit on top of the page content */
  display: block;
  /* Hidden by default */
  width: 100%;
  /* Full width (cover the whole page) */
  height: 100%;
  /* Full height (cover the whole page) */
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  /* Black background with opacity */
  z-index: 2;
  /* Specify a stack order in case you're using a different order for other elements */
  cursor: pointer;
  /* Add a pointer on hover */
}
</style>
