<template>
  <Motion :initial="pageVariants.initial" :animate="pageVariants.animate" :transition="pageVariants.transition as any"
    class="min-h-screen bg-gradient-to-br from-indigo-900 via-purple-900 to-pink-900 relative overflow-hidden">

    <!-- Background Effects -->
    <div class="absolute inset-0 opacity-20">
      <div class="absolute top-20 left-20 w-72 h-72 bg-blue-400 rounded-full blur-3xl animate-pulse"></div>
      <div class="absolute top-40 right-32 w-96 h-96 bg-purple-400 rounded-full blur-3xl animate-pulse delay-1000">
      </div>
      <div class="absolute bottom-20 left-1/3 w-80 h-80 bg-pink-400 rounded-full blur-3xl animate-pulse delay-2000">
      </div>
    </div>

    <!-- Floating Header -->
    <Motion :initial="{ opacity: 0, y: -30 }" :animate="{ opacity: 1, y: 0 }"
      :transition="{ duration: 0.8, delay: 0.1 }">
      <div class="relative z-10 p-8">
        <div class="backdrop-blur-xl bg-white/10 rounded-3xl border border-white/20 shadow-2xl p-8 max-w-7xl mx-auto">
          <div class="flex items-center justify-between">
            <div>
              <h1
                class="text-4xl font-bold bg-gradient-to-r from-white via-blue-100 to-purple-200 bg-clip-text text-transparent mb-3">
                工地安全帽检测系统
              </h1>
              <p class="text-white/80 text-lg font-light">基于 YOLOv11 的智能检测，实时守护施工安全</p>
            </div>
            <div class="hidden md:flex items-center space-x-4">
              <div class="w-3 h-3 bg-green-400 rounded-full animate-ping"></div>
              <span class="text-white/90 font-medium">系统运行中</span>
            </div>
          </div>
        </div>
      </div>
    </Motion>

    <div class="relative z-10 max-w-7xl mx-auto px-8 pb-8">
      <div class="grid grid-cols-12 gap-6">

        <!-- Main Content Area -->
        <div class="col-span-12 xl:col-span-9 space-y-6">

          <!-- Statistics Cards -->
          <Motion v-bind="cardVariants" :transition="{ ...cardVariants.transition, delay: 0.2 } as any">
            <div
              class="backdrop-blur-xl bg-white/10 rounded-2xl border border-white/20 shadow-2xl p-6 hover:bg-white/15 transition-all duration-300">
              <div class="flex items-center justify-between mb-6">
                <h2 class="text-xl font-semibold text-white flex items-center">
                  <div class="w-2 h-2 bg-cyan-400 rounded-full mr-3 animate-pulse"></div>
                  实时统计
                </h2>
                <div class="flex items-center space-x-2 text-white/70">
                  <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 20 20">
                    <path d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"></path>
                  </svg>
                  <span class="text-sm">实时更新</span>
                </div>
              </div>

              <div class="grid grid-cols-2 lg:grid-cols-4 gap-6">
                <div
                  class="group relative overflow-hidden backdrop-blur-md bg-gradient-to-br from-blue-500/20 to-cyan-500/20 rounded-xl p-6 border border-blue-400/30 transition-all duration-300 hover:scale-105 hover:from-blue-500/30 hover:to-cyan-500/30">
                  <div class="absolute top-0 right-0 w-16 h-16 bg-blue-400/10 rounded-full -mr-8 -mt-8"></div>
                  <div class="relative">
                    <div class="text-3xl font-bold text-white mb-2">{{ stats.totalDetections }}</div>
                    <div class="text-blue-200 text-sm font-medium">总检测数</div>
                  </div>
                </div>

                <div
                  class="group relative overflow-hidden backdrop-blur-md bg-gradient-to-br from-green-500/20 to-emerald-500/20 rounded-xl p-6 border border-green-400/30 transition-all duration-300 hover:scale-105 hover:from-green-500/30 hover:to-emerald-500/30">
                  <div class="absolute top-0 right-0 w-16 h-16 bg-green-400/10 rounded-full -mr-8 -mt-8"></div>
                  <div class="relative">
                    <div class="text-3xl font-bold text-white mb-2">{{ stats.complianceRate }}%</div>
                    <div class="text-green-200 text-sm font-medium">合规率</div>
                  </div>
                </div>

                <div
                  class="group relative overflow-hidden backdrop-blur-md bg-gradient-to-br from-orange-500/20 to-red-500/20 rounded-xl p-6 border border-orange-400/30 transition-all duration-300 hover:scale-105 hover:from-orange-500/30 hover:to-red-500/30">
                  <div class="absolute top-0 right-0 w-16 h-16 bg-orange-400/10 rounded-full -mr-8 -mt-8"></div>
                  <div class="relative">
                    <div class="text-3xl font-bold text-white mb-2">{{ stats.totalWarnings }}</div>
                    <div class="text-orange-200 text-sm font-medium">总警告数</div>
                  </div>
                </div>

                <div
                  class="group relative overflow-hidden backdrop-blur-md bg-gradient-to-br from-purple-500/20 to-pink-500/20 rounded-xl p-6 border border-purple-400/30 transition-all duration-300 hover:scale-105 hover:from-purple-500/30 hover:to-pink-500/30">
                  <div class="absolute top-0 right-0 w-16 h-16 bg-purple-400/10 rounded-full -mr-8 -mt-8"></div>
                  <div class="relative">
                    <div class="text-3xl font-bold text-white mb-2">{{ stats.activeCameras }}</div>
                    <div class="text-purple-200 text-sm font-medium">活跃摄像头</div>
                  </div>
                </div>
              </div>
            </div>
          </Motion>

          <!-- Control Panel -->
          <Motion v-bind="cardVariants" :transition="{ ...cardVariants.transition, delay: 0.3 } as any">
            <div
              class="backdrop-blur-xl bg-white/10 rounded-2xl border border-white/20 shadow-2xl p-6 hover:bg-white/15 transition-all duration-300">
              <div class="flex items-center justify-between mb-6">
                <h2 class="text-xl font-semibold text-white flex items-center">
                  <div class="w-2 h-2 bg-green-400 rounded-full mr-3 animate-pulse"></div>
                  快速操作
                </h2>
                <div class="flex items-center space-x-2 text-white/70">
                  <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 20 20">
                    <path d="M13 6a3 3 0 11-6 0 3 3 0 016 0zM18 8a2 2 0 11-4 0 2 2 0 014 0zM14 15a4 4 0 00-8 0v3h8v-3z">
                    </path>
                  </svg>
                  <span class="text-sm">控制中心</span>
                </div>
              </div>

              <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                <button @click="$router.push('/detection')"
                  class="group relative overflow-hidden backdrop-blur-md bg-gradient-to-r from-indigo-500/20 to-purple-500/20 hover:from-indigo-500/30 hover:to-purple-500/30 border border-indigo-400/30 rounded-xl px-6 py-4 text-white transition-all duration-300 transform hover:scale-105">
                  <div
                    class="absolute inset-0 bg-gradient-to-r from-indigo-400/10 to-purple-400/10 opacity-0 group-hover:opacity-100 transition-opacity duration-300">
                  </div>
                  <div class="relative flex items-center justify-center space-x-3">
                    <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 20 20">
                      <path d="M2 6a2 2 0 012-2h6l2 2h6a2 2 0 012 2v6a2 2 0 01-2 2H4a2 2 0 01-2-2V6z"></path>
                    </svg>
                    <span class="font-medium">进入实时监控</span>
                  </div>
                </button>

                <button @click="$router.push('/history')"
                  class="group relative overflow-hidden backdrop-blur-md bg-gradient-to-r from-orange-500/20 to-red-500/20 hover:from-orange-500/30 hover:to-red-500/30 border border-orange-400/30 rounded-xl px-6 py-4 text-white transition-all duration-300 transform hover:scale-105">
                  <div
                    class="absolute inset-0 bg-gradient-to-r from-orange-400/10 to-red-400/10 opacity-0 group-hover:opacity-100 transition-opacity duration-300">
                  </div>
                  <div class="relative flex items-center justify-center space-x-3">
                    <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 20 20">
                      <path d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z M12 8v4m0 4h.01"></path>
                    </svg>
                    <span class="font-medium">查看警告记录</span>
                  </div>
                </button>

                <button @click="$router.push('/statistics')"
                  class="group relative overflow-hidden backdrop-blur-md bg-gradient-to-r from-gray-500/20 to-gray-700/20 hover:from-gray-500/30 hover:to-gray-700/30 border border-gray-400/30 rounded-xl px-6 py-4 text-white transition-all duration-300 transform hover:scale-105">
                  <div
                    class="absolute inset-0 bg-gradient-to-r from-gray-400/10 to-gray-600/10 opacity-0 group-hover:opacity-100 transition-opacity duration-300">
                  </div>
                  <div class="relative flex items-center justify-center space-x-3">
                    <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 20 20">
                      <path
                        d="M3 4a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zm0 4a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zm0 4a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1z">
                      </path>
                    </svg>
                    <span class="font-medium">统计分析</span>
                  </div>
                </button>
              </div>
            </div>
          </Motion>

          <!-- System Information -->
          <Motion v-bind="cardVariants" :transition="{ ...cardVariants.transition, delay: 0.4 } as any">
            <div
              class="backdrop-blur-xl bg-white/10 rounded-2xl border border-white/20 shadow-2xl p-6 hover:bg-white/15 transition-all duration-300">
              <h2 class="text-xl font-semibold text-white mb-6 flex items-center">
                <div class="w-2 h-2 bg-blue-400 rounded-full mr-3 animate-pulse"></div>
                系统信息
              </h2>

              <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
                <div
                  class="group relative overflow-hidden backdrop-blur-md bg-gradient-to-br from-slate-500/20 to-gray-600/20 rounded-xl p-6 border border-slate-400/30 transition-all duration-300 hover:scale-105 hover:from-slate-500/30 hover:to-gray-600/30">
                  <div class="absolute top-0 right-0 w-16 h-16 bg-slate-400/10 rounded-full -mr-8 -mt-8"></div>
                  <div class="relative">
                    <div class="text-sm text-slate-200 mb-2">模型名称</div>
                    <div class="text-lg font-bold text-white">{{ systemInfo?.name || '加载中...' }}</div>
                  </div>
                </div>

                <div
                  class="group relative overflow-hidden backdrop-blur-md bg-gradient-to-br from-teal-500/20 to-cyan-600/20 rounded-xl p-6 border border-teal-400/30 transition-all duration-300 hover:scale-105 hover:from-teal-500/30 hover:to-cyan-600/30">
                  <div class="absolute top-0 right-0 w-16 h-16 bg-teal-400/10 rounded-full -mr-8 -mt-8"></div>
                  <div class="relative">
                    <div class="text-sm text-teal-200 mb-2">置信度阈值</div>
                    <div class="text-lg font-bold text-white">{{ systemInfo?.confidence_threshold || '加载中...' }}</div>
                  </div>
                </div>

                <div
                  class="group relative overflow-hidden backdrop-blur-md bg-gradient-to-br from-amber-500/20 to-orange-600/20 rounded-xl p-6 border border-amber-400/30 transition-all duration-300 hover:scale-105 hover:from-amber-500/30 hover:to-orange-600/30">
                  <div class="absolute top-0 right-0 w-16 h-16 bg-amber-400/10 rounded-full -mr-8 -mt-8"></div>
                  <div class="relative">
                    <div class="text-sm text-amber-200 mb-2">模型路径</div>
                    <div class="text-xs font-semibold text-white truncate">{{ systemInfo?.model_path || '加载中...' }}
                    </div>
                  </div>
                </div>

                <div
                  class="group relative overflow-hidden backdrop-blur-md rounded-xl p-6 border transition-all duration-300 hover:scale-105"
                  :class="systemInfo?.is_active ?
                    'bg-gradient-to-br from-green-500/20 to-emerald-600/20 border-green-400/30 hover:from-green-500/30 hover:to-emerald-600/30' :
                    'bg-gradient-to-br from-red-500/20 to-rose-600/20 border-red-400/30 hover:from-red-500/30 hover:to-rose-600/30'">
                  <div class="absolute top-0 right-0 w-16 h-16 rounded-full -mr-8 -mt-8"
                    :class="systemInfo?.is_active ? 'bg-green-400/10' : 'bg-red-400/10'"></div>
                  <div class="relative flex items-center space-x-3">
                    <div class="w-3 h-3 rounded-full animate-pulse"
                      :class="systemInfo?.is_active ? 'bg-green-400' : 'bg-red-400'"></div>
                    <div>
                      <div class="text-sm mb-1" :class="systemInfo?.is_active ? 'text-green-200' : 'text-red-200'">系统状态
                      </div>
                      <div class="font-bold text-white">
                        {{ systemInfo?.is_active ? '正常运行' : '已停用' }}
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </Motion>
        </div>

        <!-- Sidebar -->
        <div class="col-span-12 xl:col-span-3 space-y-6">

          <!-- Warning Timeline -->
          <Motion v-bind="cardVariants" :transition="{ ...cardVariants.transition, delay: 0.5 } as any">
            <div
              class="backdrop-blur-xl bg-white/10 rounded-2xl border border-white/20 shadow-2xl p-6 hover:bg-white/15 transition-all duration-300">
              <div class="flex items-center justify-between mb-6">
                <h3 class="text-lg font-semibold text-white flex items-center">
                  <div class="w-2 h-2 bg-red-400 rounded-full mr-3 animate-ping"></div>
                  最新警告信息
                </h3>
                <div
                  class="w-6 h-6 rounded-full bg-red-500/20 border border-red-400/30 flex items-center justify-center">
                  <span class="text-xs text-red-300 font-bold">{{ warnings.length }}</span>
                </div>
              </div>

              <div class="space-y-3 max-h-96 overflow-y-auto custom-scrollbar">
                <div v-for="warning in warnings" :key="warning.id"
                  class="group relative backdrop-blur-md rounded-xl p-4 border transition-all duration-300 hover:scale-[1.02]"
                  :class="getWarningLevelInfo(warning.warning_level).cardClass">

                  <div class="flex items-start justify-between">
                    <div class="flex-1">
                      <div class="flex items-center gap-2 mb-2">
                        <div class="rounded-full px-2 py-1 text-xs font-semibold backdrop-blur-sm"
                          :class="getWarningLevelInfo(warning.warning_level).badgeClass">
                          {{ getWarningLevelInfo(warning.warning_level).text }}
                        </div>
                        <span class="text-xs text-white/60 font-medium">{{ warning.camera_name }}</span>
                      </div>
                      <p class="text-sm text-white/90 font-medium mb-2">{{ warning.title }}</p>
                      <p class="text-xs text-white/50">{{ formatDateTime(warning.created_at) }}</p>
                    </div>
                  </div>

                  <!-- Accent bar -->
                  <div class="absolute left-0 top-0 bottom-0 w-1 rounded-l-xl"
                    :class="getWarningLevelInfo(warning.warning_level).accentClass"></div>
                </div>
              </div>

              <div class="mt-6 pt-6 border-t border-white/10">
                <button @click="$router.push('/history')"
                  class="group w-full relative overflow-hidden backdrop-blur-md bg-gradient-to-r from-slate-600/20 to-gray-600/20 hover:from-slate-600/30 hover:to-gray-600/30 border border-slate-500/30 rounded-xl p-3 text-white transition-all duration-300 transform hover:scale-[1.02]">
                  <div
                    class="absolute inset-0 bg-gradient-to-r from-slate-400/10 to-gray-400/10 opacity-0 group-hover:opacity-100 transition-opacity duration-300">
                  </div>
                  <div class="relative flex items-center justify-center space-x-2">
                    <svg class="w-4 h-4" fill="currentColor" viewBox="0 0 20 20">
                      <path
                        d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z M3 4a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1z">
                      </path>
                    </svg>
                    <span class="text-sm font-medium">查看全部警告</span>
                  </div>
                </button>
              </div>
            </div>
          </Motion>
        </div>
      </div>
    </div>
  </Motion>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { Motion } from 'motion-v'
import { getWarnings, getModels } from '@/api/inspection/API'
import type { Warning, ModelInfo } from '@/types/apis/inspection_T'

// Animation variants
const pageVariants = {
  initial: { opacity: 0, y: 40 },
  animate: { opacity: 1, y: 0 },
  transition: { duration: 0.8, ease: 'easeOut' }
}

const cardVariants = {
  initial: { opacity: 0, scale: 0.95, y: 20 },
  animate: { opacity: 1, scale: 1, y: 0 },
  transition: { duration: 0.6, ease: 'easeOut' }
}

// Data
const warnings = ref<Warning[]>([])
const systemInfo = ref<ModelInfo | null>(null)
let warningInterval: any = null

const stats = computed(() => {
  const totalWarnings = warnings.value.length
  const totalDetections = 1250
  const complianceRate = 96.8
  const activeCameras = 8
  return { totalDetections, complianceRate, totalWarnings, activeCameras }
})

const getWarningLevelInfo = (level: 'info' | 'warning' | 'critical') => {
  switch (level) {
    case 'critical':
      return {
        text: '严重警告',
        cardClass: 'bg-red-500/10 border-red-400/30 hover:bg-red-500/20',
        badgeClass: 'bg-red-500/30 text-red-200 border border-red-400/40',
        accentClass: 'bg-gradient-to-b from-red-400 to-red-600'
      }
    case 'warning':
      return {
        text: '一般警告',
        cardClass: 'bg-amber-500/10 border-amber-400/30 hover:bg-amber-500/20',
        badgeClass: 'bg-amber-500/30 text-amber-200 border border-amber-400/40',
        accentClass: 'bg-gradient-to-b from-amber-400 to-amber-600'
      }
    default:
      return {
        text: '通知',
        cardClass: 'bg-blue-500/10 border-blue-400/30 hover:bg-blue-500/20',
        badgeClass: 'bg-blue-500/30 text-blue-200 border border-blue-400/40',
        accentClass: 'bg-gradient-to-b from-blue-400 to-blue-600'
      }
  }
}

const formatDateTime = (dateString: string) =>
  new Date(dateString).toLocaleString('zh-CN', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit'
  })

const fetchWarnings = async () => {
  try {
    const res = await getWarnings({ page: 1, page_size: 5 })
    warnings.value = res.data.warnings
  } catch (error) {
    console.error('获取警告信息失败:', error)
  }
}

const fetchSystemInfo = async () => {
  try {
    const res = await getModels({ page: 1, page_size: 10 })
    if (res.data.models.length > 0) {
      systemInfo.value = res.data.models[res.data.models.length - 1]
    }
  } catch (error) {
    console.error('获取系统信息失败:', error)
  }
}

onMounted(() => {
  fetchWarnings()
  fetchSystemInfo()
  warningInterval = setInterval(() => {
    fetchWarnings()
    fetchSystemInfo()
  }, 60000)
})

onUnmounted(() => warningInterval && clearInterval(warningInterval))
</script>

<style scoped>
/* Custom scrollbar for glassmorphism theme */
.custom-scrollbar::-webkit-scrollbar {
  width: 6px;
}

.custom-scrollbar::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 3px;
  backdrop-filter: blur(10px);
}

.custom-scrollbar::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.3);
  border-radius: 3px;
  backdrop-filter: blur(10px);
}

.custom-scrollbar::-webkit-scrollbar-thumb:hover {
  background: rgba(255, 255, 255, 0.5);
}

/* Enhance glassmorphism effect */
@supports (backdrop-filter: blur()) {
  .backdrop-blur-xl {
    backdrop-filter: blur(20px);
  }

  .backdrop-blur-md {
    backdrop-filter: blur(12px);
  }
}

/* Smooth animations */
* {
  transition-property: all;
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
}

/* Gradient text enhancement */
.bg-gradient-to-r {
  background-attachment: fixed;
}
</style>
