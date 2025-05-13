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
  electricity: [680, 720, 810, 920, 850, 780, 700],
  water: [38, 42, 55, 62, 58, 48, 40]
};

// 响应式数据
const chart = ref(null);
const loading = ref(true);
const chartData = ref({...defaultData});
let chartInstance = null;

// 图表配置
const getOption = (data) => ({
  title: {
    text: 'D教学楼本周能耗趋势',
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
          color: '#3ba272'
        }
      }
    },
    {
      type: 'value',
      name: '用水量 (吨)',
      axisLine: {
        lineStyle: {
          color: '#fac858'
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
        color: '#3ba272'
      },
      areaStyle: {
        color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
          { offset: 0, color: '#3ba27233' },
          { offset: 1, color: '#3ba27210' }
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
        color: '#fac858'
      },
      areaStyle: {
        color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
          { offset: 0, color: '#fac85833' },
          { offset: 1, color: '#fac85810' }
        ])
      }
    }
  ]
});

// 获取数据
const fetchData = async () => {
  try {
    const response = await axios.get('/api/energy-trend/d-building');
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
  height: 90%;  /* 修改为固定高度 */
  position: relative;
  background: #2a3f6f;
  border-radius: 8px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.1);
  padding: 5px;
}

.chart {
  width: 100%;
  height: 100%;
  color: #fff;  /* 确保文字可见 */
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