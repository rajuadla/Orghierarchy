<script setup lang="ts">
import { Ref, ref, onMounted, watch, defineComponent } from 'vue';
import { OrgChart } from 'd3-org-chart';
import { Organizaion } from '@/types/Organization';


const finalRes: Ref<Organizaion[]> = ref();
const selectedNode: Ref<Organizaion> = ref(null);
const chatReference: Ref<OrgChart> = ref();

async function getData() {
  const res = await fetch("http://localhost:3000/");
  finalRes.value = await res.json();
};

onMounted(() => {
  getData();
});

function onNodeSelection(data: Organizaion): any {
  const attrs = chatReference.value.getChartState();
  
  const nodes = attrs.allNodes.map((d: any) => {
    if (attrs.nodeId(d.data) === data.name) {
      d.data._highlighted = true;
      selectedNode.value = data;
    } else {
      d.data._highlighted = false;
    }
    return d;
  })
  chatReference.value.data(nodes);
  chatReference.value.update(attrs);
}

function closeNode(): any {
  selectedNode.value = null;
  const attrs = chatReference.value.getChartState();
  const nodes = attrs.allNodes.map((d: any) => {
    d.data._highlighted = false
    return d;
  })
  chatReference.value.data(nodes);
  chatReference.value.update(attrs);
}

watch(finalRes, () => {

  chatReference.value = new OrgChart()
    .initialExpandLevel(3)
    .svgWidth(500)
    .nodeId((dataItem: Organizaion) => dataItem.name)
    .parentNodeId((dataItem: Organizaion) => dataItem.parent)
    .nodeWidth((dataItem: any) => 60)
    .nodeHeight((dataItem: any) => 60)
    .nodeContent((dataItem: any) => {
      return `<div 
        style="border:1px solid #CCC;width:${dataItem.width}px;height:${dataItem.height}px;border-radius: 5px;text-align:center"
      > 
           <h4>${dataItem.data.name}</h4>
       </div>`;
    })
    .container('.chart-container')
    .layout('left')
    .data(finalRes.value)
    .setActiveNodeCentered(false)
    .compact(false)
    .onNodeClick((dataItem: { data: Organizaion }) => onNodeSelection(dataItem.data))
    .render();

})
</script>

<template>
  <div id="flex-container">
    <div class="chart-container" />
    <div class="details-container" v-if="selectedNode">
      <div @click="closeNode" class="alertClose">&times;</div>
      <div id="details">
        <h4>{{ selectedNode.name }}</h4>
        <p>{{ selectedNode.description }}</p>
      </div>
    </div>
    <div></div>
  </div>
</template>

<style>
#flex-container{
  display: flex;
  flex-wrap: wrap;
}
.details-container{
  margin: auto;
  position:relative;
}
#details{
  padding: 10px;
  background-color: beige;
}
.alertClose {
 cursor: pointer;
 position:absolute;
 top:-18px;
 right:-5px;
 padding: 2px;
 font-size: 18px;
 background-color: black;
 color: #FFF;
 border-radius: 50%;
}
</style>
