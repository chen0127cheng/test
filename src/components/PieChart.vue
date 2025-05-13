<template>
  <div class="energy-pie-container">
    <div v-if="loading" class="loading">数据加载中...</div>
    <div v-show="!loading" ref="chart" class="chart"></div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from "vue";
import * as echarts from "echarts";
import axios from "axios";

// 默认数据配置
const defaultData = [
  { name: "A栋用电", value: 12000, type: "electricity" },
  { name: "A栋用水", value: 500, type: "water" },
  { name: "B栋用电", value: 9800, type: "electricity" },
  { name: "B栋用水", value: 450, type: "water" },
  { name: "C栋用电", value: 7500, type: "electricity" },
  { name: "C栋用水", value: 380, type: "water" },
];

// 响应式数据
const chart = ref(null);
const loading = ref(true);
const chartData = ref([...defaultData]);
let chartInstance = null;

// 图表配置
const getOption = (data) => ({
  title: {
    // text: '四校区能耗分布',
    left: "center",
    textStyle: {
      color: "#333",
      fontSize: 18,
    },
  },
  tooltip: {
    trigger: "item",
    formatter: "{b}: {c} ({d}%)",
  },
  legend: {
    bottom: 20,
    left: "center",
    data: data.map((item) => item.name),
  },
  color: [
    "#5470c6",
    "#91cc75",
    "#fac858",
    "#ee6666",
    "#73c0de",
    "#3ba272",
    "#fc8452",
    "#9a60b4",
  ],
  series: [
    {
      name: "能耗分布",
      type: "pie",
      radius: ["40%", "70%"],
      center: ["50%", "45%"],
      avoidLabelOverlap: true,
      label: {
        show: true,
        formatter: "{b|{b}}\n{c|{c}}",
        rich: {
          b: {
            fontSize: 12,
            lineHeight: 20,
          },
          c: {
            color: "#999",
            fontSize: 10,
          },
        },
      },
      data: data,
    },
  ],
});

// 获取数据
const fetchData = async () => {
  try {
    const response = await axios.get("/api/energy-consumption");
    chartData.value = response.data;
  } catch (error) {
    console.warn("使用默认数据:", error);
    chartData.value = [...defaultData];
  } finally {
    loading.value = false;
  }
};

// 初始化图表
const initChart = () => {
  if (!chart.value) return;

  chartInstance = echarts.init(chart.value);
  chartInstance.setOption(getOption(chartData.value));

  // 窗口自适应
  const resizeHandler = () => chartInstance.resize();
  window.addEventListener("resize", resizeHandler);

  // 清理事件
  onBeforeUnmount(() => {
    window.removeEventListener("resize", resizeHandler);
    chartInstance.dispose();
  });
};

onMounted(async () => {
  await fetchData();
  initChart();
});
</script>

<style scoped>
.energy-pie-container {
  width: 100%;
  height: 100%;
  position: relative;
  /* background: #fff; */
  border-radius: 8px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
  /* padding: 20px; */
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
  padding: 12px 24px;
  background: rgba(0, 0, 0, 0.7);
  color: #fff;
  border-radius: 4px;
  font-size: 14px;
}
</style>
