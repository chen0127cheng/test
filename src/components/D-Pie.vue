<template>
  <div class="chart-container">
    <!-- 加载状态 -->
    <div v-if="loading" class="loading">数据加载中...</div>
    
    <!-- 图表容器 -->
    <div v-show="!loading" ref="chartRef" class="chart"></div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue';
import * as echarts from 'echarts';
import axios from 'axios';

// 默认数据配置
const defaultData = [
  { value: 20, name: 'A教用水' },
  { value: 20, name: 'A教用电' },
  { value: 20, name: 'B教用水' },
  { value: 20, name: 'B教用电' },
  { value: 10, name: 'C教用水' },
  { value: 10, name: 'C教用电' },
];

// 响应式数据
const chartRef = ref(null);
const loading = ref(true);
const chartData = ref([...defaultData]); // 初始化时使用默认数据
let myChart = null;

// 基础配置（不含数据部分）
const baseOption = {
  color: [
    "#5470c6", "#91cc75", "#fac858", "#ee6666",
    "#73c0de", "#3ba272", "#fc8452", "#9a60b4",
    "#ea7ccc", "#c23531", "#2f4554", "#61a0a8", "#c4ccd3"
  ],
  title: {
    text: "用能结构",
    left: "center"
  },
  tooltip: {
    trigger: "item",
    formatter: "{a} <br/>{b} : {c} ({d}%)"
  },
  legend: {
    bottom: 10,
    left: "center",
    data: defaultData.map(item => item.name) // 默认图例数据
  },
  series: [{
    name: "用能结构",
    radius: "65%",
    type: "pie",
    avoidLabelOverlap: false,
    center: ["50%", "50%"],
    selectedMode: "single",
    data: chartData.value
  }]
};

// 获取数据
const fetchData = async () => {
  try {
    const response = await axios.get('/api/energy-structure');
    // 成功时使用API数据
    chartData.value = response.data;
    baseOption.legend.data = chartData.value.map(item => item.name);
  } catch (err) {
    console.warn('使用默认数据:', err);
    // 失败时保持默认数据
    chartData.value = [...defaultData];
    baseOption.legend.data = defaultData.map(item => item.name);
  } finally {
    loading.value = false;
  }
};

// 初始化图表
const initChart = () => {
  if (!chartRef.value) return;

  myChart = echarts.init(chartRef.value);
  
  myChart.setOption({
    ...baseOption,
    series: [{
      ...baseOption.series[0],
      data: chartData.value
    }]
  });

  window.addEventListener('resize', handleResize);
};

// 窗口调整处理
const handleResize = () => {
  myChart?.resize();
};

// 生命周期
onMounted(async () => {
  await fetchData();
  initChart(); // 无论成功失败都初始化图表
});

onBeforeUnmount(() => {
  if (myChart) {
    window.removeEventListener('resize', handleResize);
    myChart.dispose();
  }
});
</script>

<style scoped>
/* 原有样式保持不变 */
.chart-container {
  width: 600px;
  height: 400px;
  position: relative;
  left: 250px;
}

.chart {
  width: 100%;
  height: 100%;
}

.loading {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  padding: 20px;
  background: rgba(0,0,0,0.8);
  color: white;
  border-radius: 8px;
}
</style>