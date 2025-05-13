<template>
  <div class="dashboard-container">
    <!-- 顶部标题栏 -->
    <div class="header-bar">
      <h1>节能减排可视化电子看板</h1>
      <div class="header-info">
        <span class="weather">当前温度：22℃</span>
        <span class="time">{{ currentTime }}</span>
      </div>
    </div>

    <!-- 主体内容 -->
    <div class="main-content">
      <!-- 左侧统计区 -->
      <div class="left-panel">
        <!-- 校区选择 -->
        <div class="building-selector">
          <button 
            v-for="building in buildings" 
            :key="building.id"
            :class="{ active: activeBuilding === building.id }"
            @click="switchBuilding(building.id)"
          >
            {{ building.name }}
          </button>
        </div>

        <!-- 基础统计 -->
        <div class="basic-stats">
          <div class="stat-item">
            <h3>总建筑面积</h3>
            <div class="stat-value">113,020 m²</div>
            <div class="sub-stats">
              <span>A栋：45,670</span>
              <span>B栋：38,900</span>
              <span>C栋：28,450</span>
              <!-- <span>D栋：10,436</span> -->
            </div>
          </div>
          
          <div class="stat-item">
            <h3>人均能耗指标</h3>
            <div class="indicator-grid">
              <div class="indicator-item">
                <label>用电</label>
                <div class="value">2896.2 m³</div>
              </div>
              <div class="indicator-item">
                <label>用水</label>
                <div class="value">14.69 万吨</div>
              </div>
              <div class="indicator-item">
                <label>能耗率</label>
                <div class="value">56.56%</div>
              </div>
            </div>
          </div>
        </div>

        <!-- 实时排名 -->
        <div class="ranking">
          <h3>教学楼能耗排名</h3>
          <table>
            <thead>
              <tr>
                <th>排名</th>
                <th>教学楼</th>
                <th>能耗指数</th>
                <th>同比变化</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(item, index) in energyRanking" :key="index">
                <td>{{ index + 1 }}</td>
                <td>{{ item.name }}</td>
                <td>{{ item.value }}</td>
                <td :class="item.change >= 0 ? 'positive' : 'negative'">
                  {{ item.change }}%
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <!-- 中间图表区 -->
      <div class="center-panel">
        <!-- 能耗趋势 -->
        <div class="chart-card">
          <h3>本周能耗趋势</h3>
          <component 
            :is="currentChartComponent" 
            :chart-data="trendData" 
          />
        </div>

        <!-- 能耗分布 -->
        <div class="chart-card">
          <h3>本周能耗类型分布</h3>
          <component :is="PieChartComponent" :data="energyDistribution" />
        </div>
      </div>

      <!-- 右侧监控区 -->
      <div class="right-panel">
        <!-- 设备状态 -->
        <div class="status-card">
          <h3>设备在线率</h3>
          <div class="status-grid">
            <div 
              v-for="device in deviceStatus"
              :key="device.name"
              class="status-item"
            >
              <div class="progress">
                <div 
                  class="progress-bar"
                  :style="{ width: device.rate + '%' }"
                ></div>
              </div>
              <div class="label">{{ device.name }}</div>
              <div class="value">{{ device.rate }}%</div>
            </div>
          </div>
        </div>

        <!-- 实时监控 -->
        <div class="monitor-card">
          <h3>实时用电监控</h3>
          <div class="monitor-bars">
            <div 
              v-for="building in realtimeUsage"
              :key="building.name"
              class="bar-item"
            >
              <div class="building">{{ building.name }}</div>
              <div class="bar-container">
                <div 
                  class="bar"
                  :style="{ width: building.percent + '%' }"
                ></div>
              </div>
              <div class="value">{{ building.value }}万m³/h</div>
            </div>
          </div>
        </div>

        <div class="monitor-card">
          <h3>实时用水监控</h3>
          <div class="monitor-bars">
            <div 
              v-for="building in realtimeUsage"
              :key="building.name"
              class="bar-item"
            >
              <div class="building">{{ building.name }}</div>
              <div class="bar-container">
                <div 
                  class="bar"
                  :style="{ width: building.percent + '%' }"
                ></div>
              </div>
              <div class="value">{{ building.value }}万吨/h</div>
            </div>
          </div>
        </div>

      </div>
    </div>

    <!-- 底部信息 -->
    <div class="footer">
      <div class="data-source">数据来源：校园能源管理系统</div>
      <div class="update-time">最后更新：{{ currentTime }}</div>
    </div>
  </div>
</template>

<script>
import { defineAsyncComponent } from 'vue';
import axios from 'axios';
// 动态导入组件，实现懒加载

export default {
  components: {
    TrendChart: defineAsyncComponent(() => import('../../components/TrendChart.vue')),
    'b-trend-chart': defineAsyncComponent(() => import('../../components/B-TrendChart.vue')),
    'c-trend-chart': defineAsyncComponent(() => import('../../components/C-TrendChart.vue')),
    PieChart: defineAsyncComponent(() => import('../../components/PieChart.vue'))
  },
  data() {
    return {
      currentTime: '',
      activeBuilding: 'A',
      buildings: [
        { id: 'A', name: 'A栋教学楼' },
        { id: 'B', name: 'B栋教学楼' },
        { id: 'C', name: 'C栋教学楼' },
        // { id: 'D', name: 'D栋教学楼' }
      ],
      energyRanking: [
        { name: 'A栋', value: 3456.7, change: -1.3 },
        { name: 'B栋', value: 2345.6, change: -23.3 },
        { name: 'C栋', value: 1890.4, change: 12.5 },
        // { name: 'D栋', value: 1567.8, change: 5.6 }
      ],
      deviceStatus: [
        { name: '空调系统', rate: 95 },
        { name: '照明系统', rate: 88 },
        { name: '供水系统', rate: 92 },
        { name: '电梯系统', rate: 98 }
      ],
      realtimeUsage: [
        { name: 'A栋', value: 4.8, percent: 75 },
        { name: 'B栋', value: 3.2, percent: 50 },
        { name: 'C栋', value: 2.4, percent: 38 },
        // { name: 'D栋', value: 1.6, percent: 25 }
      ],
      trendData: [], // 从API获取
      energyDistribution: [], // 从API获取
      apiDataCache: {} // 数据缓存对象
    }
  },
  computed: {
    currentChartComponent() {
      const componentMap = {
        A: 'trend-chart',
        B: 'b-trend-chart',
        C: 'c-trend-chart'
      };
      return componentMap[this.activeBuilding];
    },
    PieChartComponent() {
      return 'PieChart';
    }
  },
  mounted() {
    this.updateTime()
    setInterval(this.updateTime, 1000)
    this.fetchData()
  },
  methods: {
    updateTime() {
      const now = new Date()
      this.currentTime = now.toLocaleString('zh-CN', {
        year: 'numeric',
        month: '2-digit',
        day: '2-digit',
        hour: '2-digit',
        minute: '2-digit',
        second: '2-digit',
        weekday: 'long'
      })
    },
    switchBuilding(buildingId) {
      this.activeBuilding = buildingId
      this.fetchBuildingData(buildingId)
    },
    async fetchData() {
      try {
        const cachedTrendData = this.apiDataCache['/api/trend'];
        const cachedDistributionData = this.apiDataCache['/api/distribution'];

        let trendResponse;
        let distributionResponse;

        if (cachedTrendData) {
          trendResponse = { data: cachedTrendData };
        } else {
          trendResponse = await axios.get('/api/trend');
          this.apiDataCache['/api/trend'] = trendResponse.data;
        }

        if (cachedDistributionData) {
          distributionResponse = { data: cachedDistributionData };
        } else {
          distributionResponse = await axios.get('/api/distribution');
          this.apiDataCache['/api/distribution'] = distributionResponse.data;
        }

        this.trendData = trendResponse.data;
        this.energyDistribution = distributionResponse.data;
      } catch (error) {
        console.error('数据获取失败:', error)
      }
    },
    async fetchBuildingData(buildingId) {
      try {
        const cachedBuildingData = this.apiDataCache[`/api/buildings/${buildingId}`];

        if (cachedBuildingData) {
          // 使用缓存数据进行更新操作
          // 这里假设更新操作是根据数据更新某些组件的状态
          // 例如更新左侧面板的某些显示信息
        } else {
          const response = await axios.get(`/api/buildings/${buildingId}`);
          this.apiDataCache[`/api/buildings/${buildingId}`] = response.data;
          // 根据获取的数据进行更新操作
        }
      } catch (error) {
        console.error('楼栋数据获取失败:', error)
      }
    }
  }
}
</script>

<style scoped>
/* 这里的样式保持不变，根据需要可以进一步优化 */
.dashboard-container {
  background: #ffffff; /* 改为白色背景 */
  color: #000; /* 字体颜色改为黑色，和白色背景搭配更清晰 */
  min-height: 100vh;
  padding: 20px;
  display: flex;
  flex-direction: column;
}

.header-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px 30px;
  background: #fff; /* 改为白色背景 */
  border-radius: 8px;
  margin-bottom: 20px;
  border-radius: 5px;
  box-shadow: var(--el-box-shadow-light);
}

.header-info {
  display: flex;
  gap: 30px;
  font-size: 16px;
}

.main-content {
  flex: 1;
  display: grid;
  grid-template-columns: 1fr 2fr 1fr;
  gap: 20px;
}

/* 左侧面板样式 */
.left-panel {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.building-selector {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 10px;
}

.building-selector button {
  padding: 12px;
  background: #f0f0f0; /* 改为浅灰色背景，和白色背景区分，也可按需调整 */
  border: none;
  color: #000;
  border-radius: 6px;
  cursor: pointer;
}

.building-selector button.active {
  background: #e0e0e0; /* 选中状态的浅灰色，可按需调整 */
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1); /* 调整阴影颜色和透明度 */
}

.basic-stats {
  background: #f9f9f9; /* 改为接近白色的浅灰色背景，可按需调整 */
  padding: 20px;
  border-radius: 8px;
}

.stat-item {
  margin-bottom: 20px;
}

.stat-value {
  font-size: 24px;
  font-weight: bold;
  color: #007bff; /* 可按需调整强调色 */
  margin: 10px 0;
}

.sub-stats {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 10px;
  font-size: 14px;
}

.indicator-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 15px;
}

.indicator-item {
  background: #f0f0f0; /* 改为浅灰色背景 */
  padding: 15px;
  border-radius: 6px;
  text-align: center;
}

/* 中间图表区 */
.center-panel {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.chart-card {
  background: #f9f9f9; /* 改为接近白色的浅灰色背景 */
  padding: 20px;
  border-radius: 8px;
  height: 60vh;
}
.chart-card h3{
  margin-top: 0px;
  font-size: 18px;
}
/* 右侧面板 */
.right-panel {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.status-card, .monitor-card {
  background: #f9f9f9; /* 改为接近白色的浅灰色背景 */
  padding: 20px;
  border-radius: 8px;
}

.status-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 15px;
}

.progress {
  height: 8px;
  background: #e0e0e0; /* 改为浅灰色背景 */
  border-radius: 4px;
  overflow: hidden;
}

.progress-bar {
  height: 100%;
  background: #007bff; /* 可按需调整进度条颜色 */
  transition: width 0.3s;
}

.monitor-bars {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.bar-item {
  display: grid;
  grid-template-columns: 80px 1fr 80px;
  align-items: center;
  gap: 10px;
}

.bar-container {
  height: 20px;
  background: #e0e0e0; /* 改为浅灰色背景 */
  border-radius: 10px;
  overflow: hidden;
}

.bar {
  height: 100%;
  background: #28a745; /* 可按需调整条形颜色 */
  transition: width 0.3s;
}

/* 表格样式 */
table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  padding: 12px;
  text-align: center;
  border-bottom: 1px solid #e0e0e0; /* 改为浅灰色边框 */
}

.positive { color: #28a745; }
.negative { color: #dc3545; }

.footer {
  display: flex;
  justify-content: space-between;
  padding: 15px 30px;
  background: #f0f0f0; /* 改为浅灰色背景 */
  border-radius: 8px;
  margin-top: 20px;
  font-size: 14px;
  color: #333; /* 调整底部文字颜色 */
}
</style>