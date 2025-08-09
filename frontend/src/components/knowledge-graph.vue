<script setup lang="ts">
import { ref, onMounted, watch } from 'vue';

const props = defineProps<{ chunk: any }>();
const graphRef = ref<HTMLDivElement | null>(null);

function loadEcharts(): Promise<any> {
  return new Promise((resolve, reject) => {
    if ((window as any).echarts) {
      resolve((window as any).echarts);
      return;
    }
    const script = document.createElement('script');
    script.src = 'https://cdn.jsdelivr.net/npm/echarts@5/dist/echarts.min.js';
    script.onload = () => resolve((window as any).echarts);
    script.onerror = reject;
    document.head.appendChild(script);
  });
}

async function renderGraph() {
  if (!graphRef.value) return;
  const echarts = await loadEcharts();
  const chart = echarts.init(graphRef.value);
  const nodes: any[] = [
    {
      id: props.chunk.id,
      name: props.chunk.content?.slice(0, 20) || props.chunk.id,
      symbolSize: 60,
    },
  ];
  const links: any[] = [];
  (props.chunk.relation_chunks || []).forEach((id: string, index: number) => {
    nodes.push({ id, name: id, symbolSize: 40 });
    links.push({ source: props.chunk.id, target: id });
  });
  chart.setOption({
    tooltip: {},
    series: [
      {
        type: 'graph',
        layout: 'force',
        roam: true,
        label: { show: true },
        data: nodes,
        links: links,
        force: { repulsion: 100 },
      },
    ],
  });
}

onMounted(renderGraph);
watch(
  () => props.chunk,
  () => {
    renderGraph();
  },
  { deep: true }
);
</script>

<template>
  <div ref="graphRef" style="width: 100%; height: 300px;"></div>
</template>

<style scoped>
</style>
