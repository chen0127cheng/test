<!-- ProfilePage.vue 个人中心 -->
<script setup>
import { ref } from "vue";
import { User, Message, Key } from "@element-plus/icons-vue";

// 用户信息（示例数据）
const userInfo = ref({
  username: "admin",
  avatar: "https://via.placeholder.com/100",
  department: "后勤管理处",
  role: "系统管理员",
  lastLogin: "2025-05-20 09:30:00",
});

// 登录记录（示例数据）
const loginHistory = ref([
  { date: "2025-05-20", ip: "192.168.1.100", location: "北京" },
  { date: "2025-05-19", ip: "192.168.1.101", location: "北京" },
]);

// 修改密码功能
const passwordForm = ref({
  oldPassword: "",
  newPassword: "",
  confirmPassword: "",
});
</script>

<template>
  <div class="profile-container">
    <!-- 用户信息卡 -->
    <el-card class="info-card">
      <template #header>
        <div class="card-header">
          <el-avatar :size="100" :src="userInfo.avatar" />
          <div class="user-info">
            <h2>{{ userInfo.username }}</h2>
            <p>{{ userInfo.department }}</p>
          </div>
        </div>
      </template>

      <el-descriptions :column="1" border>
        <el-descriptions-item label="角色">
          <el-tag type="success">{{ userInfo.role }}</el-tag>
        </el-descriptions-item>
        <el-descriptions-item label="上次登录">
          {{ userInfo.lastLogin }}
        </el-descriptions-item>
      </el-descriptions>
    </el-card>

    <!-- 登录历史 -->
    <el-card class="history-card">
      <template #header>
        <span class="header-title"
          ><el-icon><Message /></el-icon> 登录记录</span
        >
      </template>
      <el-timeline>
        <el-timeline-item
          v-for="(log, index) in loginHistory"
          :key="index"
          :timestamp="log.date"
        >
          {{ log.location }} (IP: {{ log.ip }})
        </el-timeline-item>
      </el-timeline>
    </el-card>

    <!-- 修改密码 -->
    <el-card class="password-card">
      <template #header>
        <span class="header-title"
          ><el-icon><Key /></el-icon> 修改密码</span
        >
      </template>
      <el-form :model="passwordForm" label-width="100px">
        <el-form-item label="原密码">
          <el-input type="password" v-model="passwordForm.oldPassword" />
        </el-form-item>
        <el-form-item label="新密码">
          <el-input type="password" v-model="passwordForm.newPassword" />
        </el-form-item>
        <el-form-item label="确认密码">
          <el-input type="password" v-model="passwordForm.confirmPassword" />
        </el-form-item>
        <el-button type="primary">提交修改</el-button>
      </el-form>
    </el-card>
  </div>
</template>

<style scoped>
.profile-container {
  padding: 20px;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 20px;
}

.card-header {
  display: flex;
  align-items: center;
  gap: 20px;
  padding: 20px;
}

.user-info h2 {
  margin: 0;
  color: #2c3e50;
}

.header-title {
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: bold;
}

.el-timeline {
  padding-left: 20px;
}

.password-card {
  grid-column: 1 / -1;
}
</style>
