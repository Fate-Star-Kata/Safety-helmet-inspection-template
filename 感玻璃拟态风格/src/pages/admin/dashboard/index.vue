<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { Motion } from 'motion-v'
import { ElMessage } from 'element-plus'
import VChart from 'vue-echarts'
import { use } from 'echarts/core'
import { CanvasRenderer } from 'echarts/renderers'
import { LineChart, PieChart, BarChart } from 'echarts/charts'
import { TitleComponent, TooltipComponent, LegendComponent, GridComponent } from 'echarts/components'
import type { EChartsOption } from 'echarts'
import {
  VideoCamera, User, Warning, Monitor, ArrowUp, ArrowDown, Refresh, Setting, Connection, Bell
} from '@element-plus/icons-vue'

// Register ECharts components
use([CanvasRenderer, LineChart, PieChart, BarChart, TitleComponent, TooltipComponent, LegendComponent, GridComponent])

// Data interfaces
interface SystemStats {
  totalCameras: number; onlineCameras: number; totalUsers: number; activeUsers: number; totalWarnings: number; todayWarnings: number; systemUptime: string; cpuUsage: number; memoryUsage: number; diskUsage: number;
}
interface CameraStatus {
  id: string; name: string; location: string; status: 'online' | 'offline' | 'error'; lastUpdate: string;
}
interface RecentActivity {
  id: string; type: 'warning' | 'user' | 'system'; message: string; timestamp: string; level: 'info' | 'warning' | 'error';
}
interface WarningTrend {
  date: string; count: number;
}

// Animation variants
const pageVariants = { initial: { opacity: 0, y: 30 }, animate: { opacity: 1, y: 0 }, transition: { duration: 0.6, ease: 'easeOut' } };
const cardVariants = { initial: { opacity: 0, scale: 0.95, y: 20 }, animate: { opacity: 1, scale: 1, y: 0 }, transition: { duration: 0.6, ease: 'easeOut' } };

// Reactive data
const loading = ref(false)
const autoRefresh = ref(true)
const refreshInterval = ref<any>(null)
const systemStats = ref<SystemStats>({ totalCameras: 24, onlineCameras: 22, totalUsers: 156, activeUsers: 89, totalWarnings: 1247, todayWarnings: 23, systemUptime: '15d 8h 32m', cpuUsage: 45, memoryUsage: 68, diskUsage: 32 });
const cameraStatus = ref<CameraStatus[]>([
  { id: '001', name: '主入口摄像头', location: '工地大门', status: 'online', lastUpdate: '2024-01-15 14:30:25' },
  { id: '002', name: '施工区域A', location: '1号楼', status: 'online', lastUpdate: '2024-01-15 14:30:20' },
  { id: '003', name: '施工区域B', location: '2号楼', status: 'offline', lastUpdate: '2024-01-15 13:45:10' },
  { id: '004', name: '材料堆放区', location: '仓库', status: 'online', lastUpdate: '2024-01-15 14:30:15' },
  { id: '005', name: '办公区域', location: '办公室', status: 'error', lastUpdate: '2024-01-15 12:20:30' }
]);
const recentActivities = ref<RecentActivity[]>([
  { id: '1', type: 'warning', message: '检测到未佩戴安全帽 - 施工区域A', timestamp: '2024-01-15 14:28:30', level: 'warning' },
  { id: '2', type: 'user', message: '用户张三登录系统', timestamp: '2024-01-15 14:25:15', level: 'info' },
  { id: '3', type: 'system', message: '摄像头 #003 离线', timestamp: '2024-01-15 13:45:10', level: 'error' },
  { id: '4', type: 'warning', message: '安全帽佩戴不规范 - 施工区域B', timestamp: '2024-01-15 13:42:20', level: 'warning' },
  { id: '5', type: 'system', message: '系统自动备份完成', timestamp: '2024-01-15 13:00:00', level: 'info' }
]);
const warningTrend = ref<WarningTrend[]>([
  { date: '01-09', count: 15 }, { date: '01-10', count: 23 }, { date: '01-11', count: 18 }, { date: '01-12', count: 31 }, { date: '01-13', count: 27 }, { date: '01-14', count: 19 }, { date: '01-15', count: 23 }
]);

// Computed properties
const cameraOnlineRate = computed(() => ((systemStats.value.onlineCameras / systemStats.value.totalCameras) * 100).toFixed(1));
const userActiveRate = computed(() => ((systemStats.value.activeUsers / systemStats.value.totalUsers) * 100).toFixed(1));
const todayWarningChange = computed(() => {
  const yesterdayWarnings = 18;
  return (((systemStats.value.todayWarnings - yesterdayWarnings) / yesterdayWarnings) * 100).toFixed(1);
});

// ECharts Dark Theme
const darkChartTheme = {
  backgroundColor: 'transparent',
  textStyle: { color: 'rgba(255, 255, 255, 0.9)' },
  title: { textStyle: { color: 'rgba(255, 255, 255, 0.9)', fontSize: 16, fontWeight: 'normal' } },
  legend: { textStyle: { color: 'rgba(255, 255, 255, 0.7)' } },
  tooltip: { backgroundColor: 'rgba(20, 20, 30, 0.85)', borderColor: 'rgba(255, 255, 255, 0.2)', borderWidth: 1, textStyle: { color: '#FFF' } }
};

const darkAxisTheme = {
  xAxis: { axisLine: { lineStyle: { color: 'rgba(255, 255, 255, 0.3)' } }, axisLabel: { color: 'rgba(255, 255, 255, 0.7)' }, splitLine: { show: false } },
  yAxis: { axisLine: { lineStyle: { color: 'rgba(255, 255, 255, 0.3)' } }, axisLabel: { color: 'rgba(255, 255, 255, 0.7)' }, splitLine: { lineStyle: { color: 'rgba(255, 255, 255, 0.2)' } } }
};

// Chart Options
const warningTrendOption = computed(() => ({
  ...darkChartTheme,
  tooltip: { trigger: 'axis', formatter: (p: any) => `${p[0].axisValue}<br/>警告: ${p[0].value}次` },
  grid: { left: '3%', right: '4%', bottom: '3%', containLabel: true },
  xAxis: { ...darkAxisTheme.xAxis, type: 'category', data: warningTrend.value.map(item => item.date) },
  yAxis: { ...darkAxisTheme.yAxis, type: 'value' },
  series: [{
    name: '警告数量', type: 'line', data: warningTrend.value.map(item => item.count), smooth: true,
    lineStyle: { color: '#F87171' }, itemStyle: { color: '#F87171' },
    areaStyle: { color: { type: 'linear', x: 0, y: 0, x2: 0, y2: 1, colorStops: [{ offset: 0, color: 'rgba(248, 113, 113, 0.3)' }, { offset: 1, color: 'rgba(248, 113, 113, 0)' }] } }
  }]
} as EChartsOption));

const systemResourceOption = computed(() => ({
  ...darkChartTheme,
  tooltip: { formatter: '{b}: {c}%' },
  series: [{
    type: 'pie', radius: ['50%', '70%'], center: ['50%', '50%'], avoidLabelOverlap: false,
    label: { show: false, position: 'center' },
    emphasis: { label: { show: true, fontSize: '20', fontWeight: 'bold' } },
    labelLine: { show: false },
    data: [
      { value: systemStats.value.cpuUsage, name: 'CPU', itemStyle: { color: '#38BDF8' } },
      { value: systemStats.value.memoryUsage, name: '内存', itemStyle: { color: '#34D399' } },
      { value: systemStats.value.diskUsage, name: '磁盘', itemStyle: { color: '#FBBF24' } }
    ]
  }]
} as EChartsOption));

// Helper methods
const formatNumber = (num: number) => num.toLocaleString();
const getStatusClass = (status: string) => ({ online: 'bg-green-500', offline: 'bg-slate-500', error: 'bg-red-500' }[status] || 'bg-gray-500');
const getActivityIcon = (type: string) => ({ warning: Warning, user: User, system: Setting }[type] || Bell);
const getActivityColor = (level: string) => ({ info: '#3B82F6', warning: '#F59E0B', error: '#EF4444' }[level] || '#909399');

// Data refresh logic
const refreshData = async () => {
  loading.value = true;
  try {
    await new Promise(resolve => setTimeout(resolve, 1000));
    ElMessage({ message: '数据已刷新', type: 'success', customClass: 'glass-message' });
  } catch (error) {
    ElMessage({ message: '数据刷新失败', type: 'error', customClass: 'glass-message' });
  } finally {
    loading.value = false;
  }
};
const toggleAutoRefresh = () => {
  if (autoRefresh.value && !refreshInterval.value) {
    refreshInterval.value = setInterval(refreshData, 30000);
  } else if (!autoRefresh.value && refreshInterval.value) {
    clearInterval(refreshInterval.value);
    refreshInterval.value = null;
  }
};

// Lifecycle hooks
onMounted(() => toggleAutoRefresh());
onUnmounted(() => { if (refreshInterval.value) clearInterval(refreshInterval.value); });
</script>

<template>
  <Motion :initial="pageVariants.initial" :animate="pageVariants.animate" :transition="pageVariants.transition as any"
    class="dashboard-page">
    <div class="space-y-6">
      <!-- Page Header -->
      <Motion :initial="{ opacity: 0, y: -20 }" :animate="{ opacity: 1, y: 0 }"
        :transition="{ duration: 0.5, delay: 0.1 }">
        <div class="page-header">
          <div>
            <h2 class="page-title">管理员控制台</h2>
            <p class="page-subtitle">工地安全帽检测系统运营概览</p>
          </div>
          <div class="flex items-center gap-4">
            <el-switch v-model="autoRefresh" @change="toggleAutoRefresh" active-text="自动刷新" class="glass-switch" />
            <button class="header-btn" @click="refreshData">
              <el-icon :class="{ 'animate-spin': loading }">
                <Refresh />
              </el-icon>
              <span>刷新数据</span>
            </button>
          </div>
        </div>
      </Motion>

      <!-- Core Metric Cards -->
      <Motion v-bind="cardVariants" :transition="{ ...cardVariants.transition, delay: 0.2 } as any">
        <div class="grid grid-cols-1 md:grid-cols-2 xl:grid-cols-4 gap-4 lg:gap-6">
          <!-- Camera Stats -->
          <div class="metric-card bg-gradient-to-br from-blue-500/10 via-blue-400/5 to-cyan-500/10 border border-blue-400/20 hover:border-blue-400/40 transition-all duration-300 hover:shadow-lg hover:shadow-blue-500/10">
            <div class="flex items-center justify-between mb-3">
              <div class="metric-icon text-blue-400 bg-blue-500/10 p-2 rounded-lg">
                <VideoCamera class="w-5 h-5" />
              </div>
              <div class="text-xs text-blue-300/70 font-medium">摄像头</div>
            </div>
            <h3 class="metric-title text-gray-200 mb-2">设备总数</h3>
            <p class="metric-value text-2xl font-bold text-white mb-2">{{ formatNumber(systemStats.totalCameras) }}</p>
            <div class="flex items-center justify-between text-sm">
              <p class="metric-change text-green-400 flex items-center gap-1">
                <ArrowUp class="w-3 h-3" /> {{ cameraOnlineRate }}%
              </p>
              <span class="text-gray-400">在线率</span>
            </div>
            <div class="metric-detail mt-3 pt-3 border-t border-blue-400/10">
              <div class="flex justify-between text-xs">
                <span class="text-green-400">在线: {{ systemStats.onlineCameras }}</span>
                <span class="text-gray-400">离线: {{ systemStats.totalCameras - systemStats.onlineCameras }}</span>
              </div>
            </div>
          </div>
          <!-- User Stats -->
          <div class="metric-card bg-gradient-to-br from-green-500/10 via-green-400/5 to-emerald-500/10 border border-green-400/20 hover:border-green-400/40 transition-all duration-300 hover:shadow-lg hover:shadow-green-500/10">
            <div class="flex items-center justify-between mb-3">
              <div class="metric-icon text-green-400 bg-green-500/10 p-2 rounded-lg">
                <User class="w-5 h-5" />
              </div>
              <div class="text-xs text-green-300/70 font-medium">用户</div>
            </div>
            <h3 class="metric-title text-gray-200 mb-2">注册用户</h3>
            <p class="metric-value text-2xl font-bold text-white mb-2">{{ formatNumber(systemStats.totalUsers) }}</p>
            <div class="flex items-center justify-between text-sm">
              <p class="metric-change text-green-400 flex items-center gap-1">
                <ArrowUp class="w-3 h-3" /> {{ userActiveRate }}%
              </p>
              <span class="text-gray-400">活跃率</span>
            </div>
            <div class="metric-detail mt-3 pt-3 border-t border-green-400/10">
              <div class="flex justify-between text-xs">
                <span class="text-green-400">活跃: {{ systemStats.activeUsers }}</span>
                <span class="text-gray-400">非活跃: {{ systemStats.totalUsers - systemStats.activeUsers }}</span>
              </div>
            </div>
          </div>
          <!-- Warning Stats -->
          <div class="metric-card bg-gradient-to-br from-orange-500/10 via-orange-400/5 to-red-500/10 border border-orange-400/20 hover:border-orange-400/40 transition-all duration-300 hover:shadow-lg hover:shadow-orange-500/10">
            <div class="flex items-center justify-between mb-3">
              <div class="metric-icon text-orange-400 bg-orange-500/10 p-2 rounded-lg">
                <Warning class="w-5 h-5" />
              </div>
              <div class="text-xs text-orange-300/70 font-medium">警告</div>
            </div>
            <h3 class="metric-title text-gray-200 mb-2">总警告数</h3>
            <p class="metric-value text-2xl font-bold text-white mb-2">{{ formatNumber(systemStats.totalWarnings) }}</p>
            <div class="flex items-center justify-between text-sm">
              <p class="metric-change flex items-center gap-1" :class="parseFloat(todayWarningChange) >= 0 ? 'text-red-400' : 'text-green-400'">
                <ArrowUp v-if="parseFloat(todayWarningChange) >= 0" class="w-3 h-3" />
                <ArrowDown v-else class="w-3 h-3" /> {{ todayWarningChange }}%
              </p>
              <span class="text-gray-400">vs 昨日</span>
            </div>
            <div class="metric-detail mt-3 pt-3 border-t border-orange-400/10">
              <div class="flex justify-between text-xs">
                <span class="text-orange-400">今日: {{ systemStats.todayWarnings }}</span>
                <span class="text-gray-400">昨日: 18</span>
              </div>
            </div>
          </div>
          <!-- System Stats -->
          <div class="metric-card bg-gradient-to-br from-purple-500/10 via-purple-400/5 to-pink-500/10 border border-purple-400/20 hover:border-purple-400/40 transition-all duration-300 hover:shadow-lg hover:shadow-purple-500/10">
            <div class="flex items-center justify-between mb-3">
              <div class="metric-icon text-purple-400 bg-purple-500/10 p-2 rounded-lg">
                <Monitor class="w-5 h-5" />
              </div>
              <div class="text-xs text-purple-300/70 font-medium">系统</div>
            </div>
            <h3 class="metric-title text-gray-200 mb-2">运行时间</h3>
            <p class="metric-value system-uptime text-2xl font-bold text-white mb-2">{{ systemStats.systemUptime }}</p>
            <div class="flex items-center justify-between text-sm">
              <p class="metric-change text-green-400 flex items-center gap-1">
                <Connection class="w-3 h-3" /> 正常
              </p>
              <span class="text-gray-400">状态</span>
            </div>
            <div class="metric-detail mt-3 pt-3 border-t border-purple-400/10">
              <div class="flex justify-between text-xs">
                <span class="text-purple-400">CPU: {{ systemStats.cpuUsage }}%</span>
                <span class="text-gray-400">内存: {{ systemStats.memoryUsage }}%</span>
              </div>
            </div>
          </div>
        </div>
      </Motion>

      <!-- Charts & Info Grid -->
      <div class="grid grid-cols-1 xl:grid-cols-3 gap-6">
        <!-- Left Column (Charts) -->
        <div class="xl:col-span-2 space-y-6">
          <Motion v-bind="cardVariants" :transition="{ ...cardVariants.transition, delay: 0.3 } as any">
            <div class="glass-card">
              <div class="card-header">
                <h3>警告趋势分析 (近7天)</h3>
              </div>
              <div class="h-72">
                <VChart :option="warningTrendOption" autoresize />
              </div>
            </div>
          </Motion>
          <Motion v-bind="cardVariants" :transition="{ ...cardVariants.transition, delay: 0.4 } as any">
            <div class="glass-card">
              <div class="card-header">
                <h3>系统资源监控</h3>
              </div>
              <div class="h-72">
                <VChart :option="systemResourceOption" autoresize />
              </div>
            </div>
          </Motion>
        </div>

        <!-- Right Column (Status & Activity) -->
        <div class="space-y-6">
          <Motion v-bind="cardVariants" :transition="{ ...cardVariants.transition, delay: 0.5 } as any">
            <div class="glass-card h-full">
              <div class="card-header">
                <h3>摄像头状态</h3>
                <span class="text-sm text-white/70">{{ systemStats.onlineCameras }}/{{ systemStats.totalCameras }}
                  在线</span>
              </div>
              <div class="info-list custom-scrollbar">
                <div v-for="camera in cameraStatus" :key="camera.id" class="info-item">
                  <div class="flex items-center gap-3">
                    <span class="w-2 h-2 rounded-full flex-shrink-0" :class="getStatusClass(camera.status)"></span>
                    <div>
                      <p class="info-title">{{ camera.name }}</p>
                      <p class="info-subtitle">{{ camera.location }}</p>
                    </div>
                  </div>
                  <p class="info-time">{{ camera.lastUpdate.split(' ')[1] }}</p>
                </div>
              </div>
            </div>
          </Motion>
          <Motion v-bind="cardVariants" :transition="{ ...cardVariants.transition, delay: 0.6 } as any">
            <div class="glass-card h-full">
              <div class="card-header">
                <h3>最新活动日志</h3>
              </div>
              <div class="info-list custom-scrollbar">
                <div v-for="activity in recentActivities" :key="activity.id" class="info-item">
                  <div class="flex items-start gap-3">
                    <el-icon :color="getActivityColor(activity.level)" class="mt-1 flex-shrink-0">
                      <component :is="getActivityIcon(activity.type)" />
                    </el-icon>
                    <div>
                      <p class="info-title">{{ activity.message }}</p>
                      <p class="info-time">{{ activity.timestamp }}</p>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </Motion>
        </div>
      </div>
    </div>
  </Motion>
</template>

<style scoped>
.dashboard-page {
  padding: 1.5rem;
  max-width: 1600px;
  margin: 0 auto;
}

.page-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 1rem;
}

.page-title {
  font-size: 1.75rem;
  /* 28px */
  font-weight: 600;
  color: white;
  margin: 0;
}

.page-subtitle {
  font-size: 0.875rem;
  /* 14px */
  color: rgba(255, 255, 255, 0.7);
  margin-top: 0.25rem;
}

.header-btn {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  background-color: rgba(255, 255, 255, 0.1);
  color: rgba(255, 255, 255, 0.9);
  border: 1px solid rgba(255, 255, 255, 0.2);
  padding: 0.5rem 1rem;
  border-radius: 0.5rem;
  font-size: 0.875rem;
  cursor: pointer;
  transition: background-color 0.2s;
}

.header-btn:hover {
  background-color: rgba(255, 255, 255, 0.2);
}

.metric-card {
  position: relative;
  overflow: hidden;
  backdrop-filter: blur(12px);
  background-image: linear-gradient(to bottom right, var(--tw-gradient-stops));
  border-radius: 0.75rem;
  /* 12px */
  padding: 1.5rem;
  border: 1px solid;
  transition: all 0.3s ease;
  color: white;
}

.metric-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.2);
}

.metric-icon {
  /* The font-size is removed as requested. Sizing is now handled by classes on the icon component itself. */
  margin-bottom: 1rem;
}

.metric-title {
  font-size: 0.875rem;
  color: rgba(255, 255, 255, 0.7);
  margin-bottom: 0.5rem;
}

.metric-value {
  font-size: 1.875rem;
  /* 30px */
  font-weight: 600;
  margin-bottom: 0.25rem;
}

.metric-value.system-uptime {
  font-size: 1.125rem;
  /* 18px */
}

.metric-change {
  font-size: 0.75rem;
  /* 12px */
  display: flex;
  align-items: center;
  gap: 0.25rem;
  margin-bottom: 1rem;
}

.metric-detail {
  display: flex;
  justify-content: space-between;
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.6);
  padding-top: 0.75rem;
  border-top: 1px solid rgba(255, 255, 255, 0.2);
}

.glass-card {
  backdrop-filter: blur(20px);
  background-color: rgba(255, 255, 255, 0.05);
  border-radius: 0.75rem;
  border: 1px solid rgba(255, 255, 255, 0.2);
  padding: 1.5rem;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  color: white;
  font-weight: 600;
}

.info-list {
  max-height: 320px;
  overflow-y: auto;
  margin-right: -1rem;
  padding-right: 1rem;
}

.info-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.75rem 0;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.info-item:last-child {
  border-bottom: none;
}

.info-title {
  font-size: 0.875rem;
  color: rgba(255, 255, 255, 0.9);
  margin-bottom: 0.125rem;
}

.info-subtitle,
.info-time {
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.6);
}

.custom-scrollbar::-webkit-scrollbar {
  width: 4px;
}

.custom-scrollbar::-webkit-scrollbar-track {
  background: transparent;
}

.custom-scrollbar::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.2);
  border-radius: 2px;
}
</style>

<!-- Global styles for ElMessage and ElSwitch -->
<style>
.glass-message {
  background-color: rgba(30, 30, 45, 0.8) !important;
  border: 1px solid rgba(255, 255, 255, 0.2) !important;
  backdrop-filter: blur(10px);
  color: white !important;
}

.glass-switch .el-switch__core {
  background-color: rgba(0, 0, 0, 0.3);
  border-color: rgba(255, 255, 255, 0.3);
}

.glass-switch .el-switch__core .el-switch__action {
  background-color: #cbd5e1;
}

.glass-switch.is-checked .el-switch__core {
  background-color: #4f46e5;
  /* indigo-600 */
  border-color: #4f46e5;
}

.glass-switch .el-switch__label {
  color: rgba(255, 255, 255, 0.8);
}
</style>
