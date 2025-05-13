<template>
  <div class="energy-trend-container">
    <div v-if="loading" class="loading">数据加载中...</div>
    <div v-show="!loading" ref="chart" class="chart"></div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue';
import * as echarts from 'echarts';
import axios from 'axios';

// 默认数据（本周数据）
const defaultData = {
  dates: ['周一', '周二', '周三', '周四', '周五', '周六', '周日'],
  electricity: [1200, 1350, 1420, 1580, 1460, 1320, 1250],
  water: [80, 85, 92, 105, 98, 88, 82]
};

// 响应式数据
const chart = ref(null);
const loading = ref(true);
const chartData = ref({...defaultData});
let chartInstance = null;

// 图表配置
const getOption = (data) => ({
  title: {
    text: 'A教学楼本周能耗趋势',
    left: 'center',
    textStyle: {
      color: '#fff',
      fontSize: 18
    }
  },
  tooltip: {
    trigger: 'axis',
    formatter: function(params) {
      return params[0].name + '<br/>' +
        params.map(item => {
          return `<span style="display:inline-block;margin-right:5px;border-radius:10px;width:10px;height:10px;background-color:${item.color};"></span>
            ${item.seriesName}: ${item.value}${item.seriesId === 'electricity' ? 'kWh' : '吨'}`
        }).join('<br/>')
    }
  },
  legend: {
    top: 30,
    data: ['用电量', '用水量']
  },
  grid: {
    left: '3%',
    right: '4%',
    bottom: '10%',
    containLabel: true
  },
  xAxis: {
    type: 'category',
    boundaryGap: false,
    data: data.dates
  },
  yAxis: [
    {
      type: 'value',
      name: '用电量 (kWh)',
      axisLine: {
        lineStyle: {
          color: '#5470c6'
        }
      }
    },
    {
      type: 'value',
      name: '用水量 (吨)',
      axisLine: {
        lineStyle: {
          color: '#91cc75'
        }
      }
    }
  ],
  series: [
    {
      name: '用电量',
      type: 'line',
      smooth: true,
      data: data.electricity,
      yAxisIndex: 0,
      itemStyle: {
        color: '#5470c6'
      },
      areaStyle: {
        color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
          { offset: 0, color: '#5470c633' },
          { offset: 1, color: '#5470c610' }
        ])
      }
    },
    {
      name: '用水量',
      type: 'line',
      smooth: true,
      data: data.water,
      yAxisIndex: 1,
      itemStyle: {
        color: '#91cc75'
      },
      areaStyle: {
        color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
          { offset: 0, color: '#91cc7533' },
          { offset: 1, color: '#91cc7510' }
        ])
      }
    }
  ]
});

// 获取数据
const fetchData = async () => {
  try {
    const response = await axios.get('/api/energy-trend/a-building');
    chartData.value = response.data;
  } catch (error) {
    console.warn('使用默认数据:', error);
    chartData.value = {...defaultData};
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
  window.addEventListener('resize', resizeHandler);

  // 清理事件
  onBeforeUnmount(() => {
    window.removeEventListener('resize', resizeHandler);
    chartInstance.dispose();
  });
};

onMounted(async () => {
  await fetchData();
  initChart();
});
</script>

<style scoped>
.energy-trend-container {
  width: 100%;
  height: 90%;
  position: relative;
  /* background: #2a3f6f; */
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.1);
  padding: 5px;
  color: beige;
}

.chart {
  width: 100%;
  height: 100%;
  color: beige;
}


.loading {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  padding: 12px 24px;
  background: rgba(0,0,0,0.7);
  color: #fff;
  border-radius: 4px;
  font-size: 14px;
}
</style>