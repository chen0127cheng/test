<script setup>
import {
  reactive,
  onMounted,
  ref,
  onBeforeUnmount,
  shallowRef,
  computed,
} from "vue";
import { getAdmin } from "../../api";
import useAdmin from "../../stores/admin";
import { Memo, Lightning, DataAnalysis } from "@element-plus/icons-vue";
import { Watermelon } from "@element-plus/icons-vue";
import { defineAsyncComponent } from "vue";
import axios from "axios";

const { admin, updateAdmin } = useAdmin();

// 使用 shallowRef 结合 defineAsyncComponent 定义异步组件
const Dpie = shallowRef(
  defineAsyncComponent(() => import("../../components/D-Pie.vue"))
);
const TrendChart = shallowRef(
  defineAsyncComponent(() => import("../../components/TrendChart.vue"))
);
const PieChart = shallowRef(
  defineAsyncComponent(() => import("../../components/PieChart.vue"))
);
const BTrendChart = shallowRef(
  defineAsyncComponent(() => import("../../components/B-TrendChart.vue"))
);
const CTrendChart = shallowRef(
  defineAsyncComponent(() => import("../../components/C-TrendChart.vue"))
);

// 使用 shallowRef 减少响应式开销
const loadingPie = ref(true);
const loading1 = ref(true);
const loading2 = ref(true);
const loading3 = ref(true);
const loading4 = ref(true);

// 图表数据
const chartDataPie = ref([]);
const chartData1 = ref({});
const chartData2 = ref({});
const chartData3 = ref({});
const chartData4 = ref({});

// 缓存对象
const apiCache = reactive({});

// 默认数据
const defaultPieData = [
  { value: 320, name: "A教" },
  { value: 280, name: "B教" },
  { value: 260, name: "C教" },
];

const defaultChart1 = {
  xData: [
    "1月",
    "2月",
    "3月",
    "4月",
    "5月",
    "6月",
    "7月",
    "8月",
    "9月",
    "10月",
    "11月",
    "12月",
  ],
  seriesData: [60, 67, 58.5, 61, 69, 70, 75, 80, 65, 70, 72, 68],
};

const defaultChart2 = {
  xData: ["1月", "2月", "3月", "4月", "5月"],
  seriesA: [320, 300, 280, 260, 380],
  seriesB: [280, 270, 250, 240, 320],
  seriesC: [260, 240, 220, 200, 300],
};

const defaultChart3 = {
  xData: [
    "1月",
    "2月",
    "3月",
    "4月",
    "5月",
    "6月",
    "7月",
    "8月",
    "9月",
    "10月",
    "11月",
    "12月",
  ],
  seriesData: [70, 75, 80, 65, 70, 72, 68, 75, 80, 65, 70, 75],
};

const defaultChart4 = {
  xData: ["1月", "2月", "3月", "4月", "5月"],
  seriesA: [300, 320, 340, 360, 380],
  seriesB: [250, 270, 290, 310, 330],
  seriesC: [200, 220, 240, 260, 280],
};

// 带缓存的API请求
const fetchWithCache = async (url) => {
  if (apiCache[url]) {
    return apiCache[url];
  }

  const response = await axios.get(url);
  apiCache[url] = response.data;
  return response.data;
};

// API请求
const fetchData = async () => {
  try {
    // 并行请求多个API
    const [pieRes, chart1Res, chart2Res, chart3Res, chart4Res] =
      await Promise.all([
        fetchWithCache("/api/energy-pie"),
        fetchWithCache("/api/power-total"),
        fetchWithCache("/api/power-building"),
        fetchWithCache("/api/water-total"),
        fetchWithCache("/api/water-building"),
      ]);

    console.log("pieRes.data:", pieRes.data);
    console.log("chart1Res.data:", chart1Res.data);
    console.log("chart2Res.data:", chart2Res.data);
    console.log("chart3Res.data:", chart3Res.data);
    console.log("chart4Res.data:", chart4Res.data);

    chartDataPie.value = pieRes.data || defaultPieData;
    chartData1.value = chart1Res.data || defaultChart1;
    chartData2.value = chart2Res.data || defaultChart2;
    chartData3.value = chart3Res.data || defaultChart3;
    chartData4.value = chart4Res.data || defaultChart4;
  } catch (error) {
    console.error("API请求失败，使用默认数据:", error);
    chartDataPie.value = defaultPieData;
    chartData1.value = defaultChart1;
    chartData2.value = defaultChart2;
    chartData3.value = defaultChart3;
    chartData4.value = defaultChart4;
  } finally {
    // 延迟加载以确保骨架屏显示足够长时间
    setTimeout(() => {
      loadingPie.value = false;
      loading1.value = false;
      loading2.value = false;
      loading3.value = false;
      loading4.value = false;
    }, 500); // 至少显示500ms的加载状态
  }
};

onMounted(async () => {
  // 并行加载数据，不再需要等待组件导入
  await Promise.all([loadAdmin(), fetchData()]);
});

// 公用方法
const addResizeListener = (chart) => {
  const handler = () => chart.resize();
  window.addEventListener("resize", handler);
  onBeforeUnmount(() => {
    window.removeEventListener("resize", handler);
    chart.dispose();
  });
};

const loadAdmin = async () => {
  let data = await getAdmin();
  updateAdmin({
    username: data.username,
    avatar: data.avatar,
  });
};

// 用户登录信息（模拟数据）
const loginInfo = reactive({
  loginTime: "2025-05-09 09:00:00",
  loginPlace: "北京",
});
</script>

<!-- 模板部分保持不变 -->

<template>
  <el-row :gutter="20">
    <el-col :span="6">
      <el-card class="box-card">
        <template #header>
          <div class="card-header">
            <el-avatar
              class="avatar"
              :src="admin.avatar"
              shape="square"
              :size="40"
            >
            </el-avatar>
            <span style="font-size: 24px">{{ admin.username }} </span>
          </div>
        </template>
        <div class="info">
          <p>登录时间：{{ loginInfo.loginTime }}</p>
          <p>登录地点：{{ loginInfo.loginPlace }}</p>
        </div>
      </el-card>
    </el-col>
    <!-- 单月统计信息展示 -->
    <!-- 月度能耗统计信息展示 -->
    <el-col :span="18">
      <el-card class="box-card">
        <template #header>
          <div class="card-header">
            <span style="font-size: 18px">📊 2025年5月能耗统计</span>
          </div>
        </template>
        <div class="info">
          <el-row :gutter="24">
            <!-- A教学楼 -->
            <el-col :span="8">
              <div class="card-container">
                <div
                  class="card-left-container"
                  style="background-color: #ff6b6b"
                >
                  <el-icon :size="90" color="#fff">
                    <Lightning />
                  </el-icon>
                </div>
                <div class="card-right-container">
                  <p class="number">3,85</p>
                  <p>A教总耗能</p>
                </div>
              </div>
            </el-col>

            <!-- B教学楼 -->
            <el-col :span="8">
              <div class="card-container">
                <div
                  class="card-left-container"
                  style="background-color: #4ecdc4"
                >
                  <el-icon :size="90" color="#fff">
                    <Watermelon />
                  </el-icon>
                </div>
                <div class="card-right-container">
                  <p class="number">520</p>
                  <p>B教总耗能</p>
                </div>
              </div>
            </el-col>

            <!-- C教学楼 -->
            <el-col :span="8">
              <div class="card-container">
                <div
                  class="card-left-container"
                  style="background-color: #45b7d1"
                >
                  <el-icon :size="90" color="#fff">
                    <DataAnalysis />
                  </el-icon>
                </div>
                <div class="card-right-container">
                  <p class="number">8,24</p>
                  <p>C教总能耗</p>
                </div>
              </div>
            </el-col>
          </el-row>
        </div>
      </el-card>
    </el-col>
  </el-row>
  <!-- 图表区域 -->
  <el-row :gutter="20">
    <!-- 其他图表 -->
    <el-col :span="24">
      <el-card class="box-card">
        <div class="chart-container">
          <!-- 使用骨架屏替代简单的加载文本 -->
          <div v-if="loadingPie" class="skeleton-chart">
            <div
              class="skeleton-line"
              style="width: 80%; margin: 20px auto"
            ></div>
            <div
              class="skeleton-line"
              style="width: 60%; margin: 20px auto"
            ></div>
            <div class="skeleton-circle" style="margin: 40px auto"></div>
          </div>
          <component
            v-else
            :is="Dpie"
            :data="chartDataPie"
            style="height: 400px"
          />
        </div>
      </el-card>
    </el-col>
  </el-row>
  <el-row :gutter="20">
    <el-col :span="12">
      <!-- A教本周能耗趋势折线图 -->
      <el-card class="box-card">
        <h3>A教本周能耗趋势</h3>
        <div class="chart-container">
          <div v-if="loading1" class="skeleton-chart">
            <div
              class="skeleton-line"
              style="width: 80%; margin: 20px auto"
            ></div>
            <div
              class="skeleton-line"
              style="width: 60%; margin: 20px auto"
            ></div>
            <div class="skeleton-lines" style="margin: 40px auto"></div>
          </div>
          <component
            v-else
            :is="TrendChart"
            :chart-data="chartData1"
            style="height: 400px"
          />
        </div>
      </el-card>
    </el-col>
    <el-col :span="12">
      <!-- 本周能耗类型分布饼图 -->
      <el-card class="box-card">
        <h3>本周能耗类型分布</h3>
        <div class="chart-container">
          <div v-if="loading2" class="skeleton-chart">
            <div
              class="skeleton-line"
              style="width: 80%; margin: 20px auto"
            ></div>
            <div
              class="skeleton-line"
              style="width: 60%; margin: 20px auto"
            ></div>
            <div class="skeleton-bars" style="margin: 40px auto"></div>
          </div>
          <component
            v-else
            :is="PieChart"
            :data="chartData2"
            style="height: 400px"
          />
        </div>
      </el-card>
    </el-col>
  </el-row>
  <el-row :gutter="20">
    <el-col :span="12">
      <!-- B教本周能耗趋势折线图 -->
      <el-card class="box-card">
        <h3>B教本周能耗趋势</h3>
        <div class="chart-container">
          <div v-if="loading3" class="skeleton-chart">
            <div
              class="skeleton-line"
              style="width: 80%; margin: 20px auto"
            ></div>
            <div
              class="skeleton-line"
              style="width: 60%; margin: 20px auto"
            ></div>
            <div class="skeleton-lines" style="margin: 40px auto"></div>
          </div>
          <component
            v-else
            :is="BTrendChart"
            :data="chartData3"
            style="height: 400px"
          />
        </div>
      </el-card>
    </el-col>
    <el-col :span="12">
      <!-- C教本周能耗趋势折线图 -->
      <el-card class="box-card">
        <h3>C教本周能耗趋势</h3>
        <div class="chart-container">
          <div v-if="loading4" class="skeleton-chart">
            <div
              class="skeleton-line"
              style="width: 80%; margin: 20px auto"
            ></div>
            <div
              class="skeleton-line"
              style="width: 60%; margin: 20px auto"
            ></div>
            <div class="skeleton-bars" style="margin: 40px auto"></div>
          </div>
          <component
            v-else
            :is="CTrendChart"
            :data="chartData4"
            style="height: 400px"
          />
        </div>
      </el-card>
    </el-col>
  </el-row>
</template>

<style lang="scss" scoped>
.el-row {
  margin-bottom: 20px;

  &:last-child {
    margin-bottom: 0;
  }

  .el-col {
    .box-card {
      .card-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
      }

      .info {
        font-size: 14px;
      }
    }
  }
}

.card-container {
  display: flex;
  align-items: center;
  justify-content: space-between;
  border: 1px solid #e4e7ed;
  text-align: center;
  padding-right: 20px;

  .card-left-container {
    color: white;
  }

  .card-right-container {
    .number {
      font-size: 18px;
      font-weight: bold;
    }
  }
}

.chart-container {
  position: relative;
  width: 100%;
  height: 400px;
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
  z-index: 10;
}

/* 骨架屏样式 */
.skeleton-chart {
  width: 100%;
  height: 100%;
  background-color: #f5f5f5;
  border-radius: 4px;
  padding: 20px;
  box-sizing: border-box;

  .skeleton-line {
    height: 20px;
    background-color: #e0e0e0;
    border-radius: 4px;
  }

  .skeleton-circle {
    width: 120px;
    height: 120px;
    background-color: #e0e0e0;
    border-radius: 50%;
  }

  .skeleton-lines {
    height: 180px;
    display: flex;
    align-items: flex-end;
    gap: 5px;
    padding: 0 20px;

    div {
      flex: 1;
      background-color: #e0e0e0;
      border-radius: 4px;
    }
  }

  .skeleton-bars {
    height: 180px;
    display: flex;
    align-items: flex-end;
    gap: 8px;
    padding: 0 20px;

    div {
      width: 20px;
      background-color: #e0e0e0;
      border-radius: 4px;
    }
  }
}
</style>
