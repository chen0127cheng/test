
<template>
  <div class="chart-container">
    <div v-if="loading" class="loading">数据加载中...</div>
    <div v-show="!loading" ref="chartRef" class="chart"></div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from "vue";
import * as echarts from "echarts";
import axios from "axios";

// 时间轴数据
const timePoints = [
  "第5天",
  "第10天",
  "第15天",
  "第20天",
  "第25天",
  "第30天"
];

// 默认数据结构（带时间维度）
const defaultData = [
  { time: "第5天", building: "A教", value: 40 },
  { time: "第5天", building: "B教", value: 35 },
  { time: "第5天", building: "C教", value: 20 },
  { time: "第5天", building: "D教", value: 30 },

  { time: "第10天", building: "A教", value: 45 },
  { time: "第10天", building: "B教", value: 68 },
  { time: "第10天", building: "C教", value: 80 },
  { time: "第10天", building: "D教", value: 36 },

  { time: "第15天", building: "A教", value: 28 },
  { time: "第15天", building: "B教", value: 39 },
  { time: "第15天", building: "C教", value: 41 },
  { time: "第15天", building: "D教", value: 90 },

  { time: "第20天", building: "A教", value: 60 },
  { time: "第20天", building: "B教", value: 68 },
  { time: "第20天", building: "C教", value: 75 },
  { time: "第20天", building: "D教", value: 50 },

  { time: "第25天", building: "A教", value: 44 },
  { time: "第25天", building: "B教", value: 60 },
  { time: "第25天", building: "C教", value: 41 },
  { time: "第25天", building: "D教", value: 80 },

  { time: "第30天", building: "A教", value: 55 },
  { time: "第30天", building: "B教", value: 75 },
  { time: "第30天", building: "C教", value: 63 },
  { time: "第30天", building: "D教", value: 86 },
];

const chartRef = ref(null);
const loading = ref(true);
const barData = ref({ A教: [], B教: [], C教: [], D教: [] });
const lineData = ref([]);
let myChart = null;

// 新版数据处理逻辑
const processData = (rawData) => {
  const buildingMap = {
    A教: [],
    B教: [],
    C教: [],
    D教: []
  };
  
  const timeMap = new Map();
  const lineValues = [];

  // 按时间分组数据
  rawData.forEach(item => {
    if (!timeMap.has(item.time)) {
      timeMap.set(item.time, {
        A教: 0,
        B教: 0,
        C教: 0,
        D教: 0
      });
    }
    timeMap.get(item.time)[item.building] = item.value;
  });

  // 按预定义时间顺序处理
  timePoints.forEach(time => {
    const data = timeMap.get(time) || { A教: 0, B教: 0, C教: 0, D教: 0 };
    
    // 填充柱状图数据
    Object.keys(buildingMap).forEach(building => {
      buildingMap[building].push(data[building]);
    });

    // 计算折线图数据（总和）
    lineValues.push(
      Object.values(data).reduce((sum, val) => sum + val, 0)
    );
  });

  return { barData: buildingMap, lineData: lineValues };
};

const fetchData = async () => {
  try {
    const response = await axios.get("/api/energy-monthly");
    const processed = processData(response.data);
    barData.value = processed.barData;
    lineData.value = processed.lineData;
  } catch (err) {
    console.warn("使用默认数据:", err);
    const processed = processData(defaultData);
    barData.value = processed.barData;
    lineData.value = processed.lineData;
  } finally {
    loading.value = false;
  }
};

const initChart = () => {
  if (!chartRef.value) return;
  myChart = echarts.init(chartRef.value);

  const option = {
    color: ["#5470c6", "#91cc75", "#fac858", "#ee6666"],
    title: {
      text: "月度消费统计",
      subtext: "水电总耗能 (kWh)",
      left: "center"
    },
    tooltip: {
      trigger: "axis",
      axisPointer: {
        type: "shadow"
      },
      formatter: params => {
        const total = params.reduce((sum, p) => sum + p.value, 0);
        return `
          ${params[0].name}<br/>
          ${params.map(p => `${p.marker} ${p.seriesName}: ${p.value}kWh`).join('<br/>')}
          <hr style="margin:5px 0"/>
          <strong>总耗能: ${total}kWh</strong>
        `;
      }
    },
    legend: {
      data: ["A教", "B教", "C教", "D教", "总耗能"],
      bottom: 10
    },
    xAxis: {
      type: "category",
      data: timePoints,
      axisLabel: {
        interval: 0,
        rotate: 45
      }
    },
    yAxis: {
      type: "value",
      name: "水电总耗能 (kWh)",
      axisLine: { show: true }
    },
    series: [
      ...["A教", "B教", "C教", "D教"].map(building => ({
        name: building,
        type: "bar",
        data: barData.value[building],
        barGap: "10%",
        emphasis: { focus: "series" }
      })),
      {
        name: "总耗能",
        type: "line",
        data: lineData.value,
        smooth: true,
        symbol: "circle",
        symbolSize: 8,
        lineStyle: {
          width: 3,
          type: "dashed"
        },
        label: {
          show: true,
          position: "top",
          formatter: ({ value }) => `${value}kWh`
        }
      }
    ],
    dataZoom: [{
      type: "inside",
      start: 0,
      end: 100
    }]
  };

  myChart.setOption(option);
  window.addEventListener("resize", handleResize);
};

const handleResize = () => {
  myChart?.resize();
};

onMounted(async () => {
  await fetchData();
  initChart();
});

onBeforeUnmount(() => {
  if (myChart) {
    window.removeEventListener("resize", handleResize);
    myChart.dispose();
  }
});
</script>

<style scoped>
.chart-container {
  width: 800px;
  height: 500px;
  position: relative;
  background: white;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.1);
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
  background: rgba(0, 0, 0, 0.8);
  color: white;
  border-radius: 8px;
}
</style>