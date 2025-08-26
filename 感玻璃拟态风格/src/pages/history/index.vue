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

    <div class="relative z-10 p-6 space-y-6">
      <!-- Page Header -->
      <Motion :initial="{ opacity: 0, y: -30 }" :animate="{ opacity: 1, y: 0 }"
        :transition="{ duration: 0.8, delay: 0.1 }">
        <div class="backdrop-blur-xl bg-white/10 rounded-3xl border border-white/20 shadow-2xl p-8">
          <div class="max-w-7xl mx-auto">
            <h1
              class="text-4xl font-bold bg-gradient-to-r from-white via-blue-100 to-purple-200 bg-clip-text text-transparent mb-3">
              历史记录
            </h1>
            <p class="text-white/80 text-lg font-light">查看和管理工地安全帽检测系统的历史记录信息</p>
          </div>
        </div>
      </Motion>

      <!-- Filter Area -->
      <Motion v-bind="cardVariants" :transition="{ ...cardVariants.transition, delay: 0.2 } as any">
        <div class="backdrop-blur-xl bg-white/10 rounded-2xl border border-white/20 shadow-2xl p-6">
          <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-5 gap-4 items-end">
            <!-- Form controls styled for glassmorphism -->
            <div class="form-control">
              <label class="label pb-1"><span class="label-text font-medium text-white/70">开始日期</span></label>
              <input type="date" v-model="filters.startDate"
                class="input w-full bg-black/20 border-white/20 text-white/90 placeholder-white/50 focus:border-blue-400 focus:ring-2 focus:ring-blue-400/50 transition" />
            </div>
            <div class="form-control">
              <label class="label pb-1"><span class="label-text font-medium text-white/70">结束日期</span></label>
              <input type="date" v-model="filters.endDate"
                class="input w-full bg-black/20 border-white/20 text-white/90 placeholder-white/50 focus:border-blue-400 focus:ring-2 focus:ring-blue-400/50 transition" />
            </div>
            <div class="form-control">
              <label class="label pb-1"><span class="label-text font-medium text-white/70">检测类型</span></label>
              <select v-model="filters.level"
                class="select w-full bg-black/20 border-white/20 text-white/90 focus:border-blue-400 focus:ring-2 focus:ring-blue-400/50 transition">
                <option value="" class="bg-gray-800">全部类型</option>
                <option value="wearing_hat" class="bg-gray-800">佩戴安全帽</option>
                <option value="no_hat" class="bg-gray-800">未佩戴安全帽</option>
                <option value="person_detected" class="bg-gray-800">人员检测</option>
              </select>
            </div>
            <div class="form-control">
              <label class="label pb-1"><span class="label-text font-medium text-white/70">摄像头</span></label>
              <select v-model="filters.camera"
                class="select w-full bg-black/20 border-white/20 text-white/90 focus:border-blue-400 focus:ring-2 focus:ring-blue-400/50 transition">
                <option value="" class="bg-gray-800">全部摄像头</option>
                <option v-for="camera in cameraOptions" :key="camera" :value="camera" class="bg-gray-800">
                  {{ camera }}
                </option>
              </select>
            </div>
            <button @click="applyFilters"
              class="group relative overflow-hidden backdrop-blur-md bg-gradient-to-r from-indigo-500/20 to-purple-500/20 hover:from-indigo-500/30 hover:to-purple-500/30 border border-indigo-400/30 rounded-xl h-12 text-white transition-all duration-300 transform hover:scale-105">
              <div class="relative flex items-center justify-center space-x-2">
                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                    d="M3 4a1 1 0 011-1h16a1 1 0 011 1v2.586a1 1 0 01-.293.707l-6.414 6.414a1 1 0 00-.293.707V17l-4 4v-6.586a1 1 0 00-.293-.707L3.293 7.293A1 1 0 013 6.586V4z">
                  </path>
                </svg>
                <span class="font-medium">筛选</span>
              </div>
            </button>
          </div>
        </div>
      </Motion>

      <!-- Statistics Module -->
      <Motion v-bind="cardVariants" :transition="{ ...cardVariants.transition, delay: 0.3 } as any">
        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
          <div
            class="group relative overflow-hidden backdrop-blur-md bg-gradient-to-br from-red-500/20 to-orange-500/20 rounded-xl p-6 border border-red-400/30">
            <p class="text-sm font-medium text-red-200">低置信度</p>
            <p class="text-3xl font-bold text-white">{{ stats.severe }}</p>
          </div>
          <div
            class="group relative overflow-hidden backdrop-blur-md bg-gradient-to-br from-yellow-500/20 to-amber-500/20 rounded-xl p-6 border border-yellow-400/30">
            <p class="text-sm font-medium text-yellow-200">中等置信度</p>
            <p class="text-3xl font-bold text-white">{{ stats.warning }}</p>
          </div>
          <div
            class="group relative overflow-hidden backdrop-blur-md bg-gradient-to-br from-blue-500/20 to-cyan-500/20 rounded-xl p-6 border border-blue-400/30">
            <p class="text-sm font-medium text-blue-200">高置信度</p>
            <p class="text-3xl font-bold text-white">{{ stats.info }}</p>
          </div>
          <div
            class="group relative overflow-hidden backdrop-blur-md bg-gradient-to-br from-green-500/20 to-emerald-500/20 rounded-xl p-6 border border-green-400/30">
            <p class="text-sm font-medium text-green-200">检测准确率</p>
            <p class="text-3xl font-bold text-white">{{ stats.completionRate }}%</p>
          </div>
        </div>
      </Motion>

      <!-- Action Area -->
      <Motion v-bind="cardVariants" :transition="{ ...cardVariants.transition, delay: 0.4 } as any">
        <div class="backdrop-blur-xl bg-white/10 rounded-2xl border border-white/20 shadow-2xl p-4">
          <div class="flex flex-wrap gap-3 justify-center">
            <!-- Action buttons with expanded utility classes -->
            <button @click="exportRecords"
              class="flex-1 relative overflow-hidden backdrop-blur-md bg-gradient-to-r border rounded-xl px-4 py-2 text-white/90 text-sm font-medium transition-all duration-300 transform hover:scale-105 from-green-500/20 to-emerald-500/20 border-green-400/30 hover:from-green-500/30 hover:to-emerald-500/30">
              📥 导出记录CSV</button>
            <button @click="exportWarnings"
              class="flex-1 relative overflow-hidden backdrop-blur-md bg-gradient-to-r border rounded-xl px-4 py-2 text-white/90 text-sm font-medium transition-all duration-300 transform hover:scale-105 from-blue-500/20 to-cyan-500/20 border-blue-400/30 hover:from-blue-500/30 hover:to-cyan-500/30">
              📤 导出警告CSV</button>
            <button @click="deleteSelected"
              class="flex-1 relative overflow-hidden backdrop-blur-md bg-gradient-to-r border rounded-xl px-4 py-2 text-white/90 text-sm font-medium transition-all duration-300 transform hover:scale-105 from-red-500/20 to-rose-500/20 border-red-400/30 hover:from-red-500/30 hover:to-rose-500/30">
              🗑️ 删除选中</button>
            <button @click="generateReport"
              class="flex-1 relative overflow-hidden backdrop-blur-md bg-gradient-to-r border rounded-xl px-4 py-2 text-white/90 text-sm font-medium transition-all duration-300 transform hover:scale-105 from-purple-500/20 to-pink-500/20 border-purple-400/30 hover:from-purple-500/30 hover:to-pink-500/30">
              📊 生成PDF报告</button>
          </div>
        </div>
      </Motion>

      <!-- Detection Records List -->
      <Motion v-bind="cardVariants" :transition="{ ...cardVariants.transition, delay: 0.5 } as any">
        <div class="backdrop-blur-xl bg-white/10 rounded-2xl border border-white/20 shadow-2xl">
          <div class="p-6 border-b border-white/10">
            <h2 class="text-xl font-semibold text-white">检测记录</h2>
          </div>

          <div class="grid gap-4 p-6">
            <div v-for="record in paginatedRecords" :key="record.id"
              class="group relative backdrop-blur-md rounded-xl p-4 border transition-all duration-300 hover:scale-[1.02] hover:shadow-lg"
              :class="getRecordStyle(record.detection_type).cardClass">
              <!-- Accent bar -->
              <div class="absolute left-0 top-0 bottom-0 w-1 rounded-l-xl"
                :class="getRecordStyle(record.detection_type).accentClass"></div>
              <div class="flex items-start space-x-4 ml-2">
                <input type="checkbox" v-model="selectedRecords" :value="record.id" @click.stop
                  class="checkbox-glass mt-1.5 flex-shrink-0" />
                <div class="flex-1 min-w-0">
                  <div class="flex items-start justify-between mb-3">
                    <div>
                      <h3 class="text-lg font-semibold text-white mb-1">{{ getRecordStyle(record.detection_type).title
                      }}</h3>
                      <div class="flex items-center space-x-2 text-sm">
                        <span
                          class="inline-flex items-center px-2 py-0.5 backdrop-blur-sm bg-white/10 text-white/70 rounded">
                          📹 {{ record.camera_name }}
                        </span>
                        <span
                          class="inline-flex items-center px-2 py-0.5 backdrop-blur-sm bg-white/10 text-white/70 rounded">
                          ID: {{ record.id }}
                        </span>
                      </div>
                    </div>
                    <div class="text-right flex-shrink-0">
                      <div class="text-sm font-medium text-white/90">{{ formatDateTime(record.detected_at).split(' ')[1]
                      }}</div>
                      <div class="text-xs text-white/60">{{ formatDateTime(record.detected_at).split(' ')[0] }}</div>
                    </div>
                  </div>
                  <div class="flex items-center justify-between">
                    <div class="flex items-center space-x-2">
                      <span class="text-sm text-white/70">置信度:</span>
                      <div class="flex items-center space-x-2">
                        <div class="w-24 h-2 bg-black/20 rounded-full overflow-hidden">
                          <div class="h-full rounded-full transition-all duration-300"
                            :class="getRecordStyle(record.detection_type).accentClass"
                            :style="{ width: (record.confidence * 100) + '%' }"></div>
                        </div>
                        <span class="text-sm font-medium"
                          :class="getRecordStyle(record.detection_type).confidenceClass">
                          {{ (record.confidence * 100).toFixed(1) }}%
                        </span>
                      </div>
                    </div>
                    <span class="inline-flex items-center px-2 py-1 text-xs font-semibold rounded-full backdrop-blur-sm"
                      :class="getRecordStyle(record.detection_type).badgeClass">
                      {{ getRecordStyle(record.detection_type).statusText }}
                    </span>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Pagination -->
          <div class="p-6 border-t border-white/10">
            <div class="flex items-center justify-between">
              <div class="text-sm text-white/70">
                显示第 {{ startIndex }} - {{ endIndex }} 条，共 {{ totalRecords }} 条记录
              </div>
              <div class="flex space-x-2">
                <button @click="previousPage" :disabled="currentPage === 1"
                  class="px-4 py-2 text-sm font-medium bg-white/5 border border-white/20 text-white/80 rounded-lg hover:bg-white/10 disabled:opacity-50 disabled:cursor-not-allowed transition-colors">上一页</button>
                <span class="flex items-center px-3 text-sm text-white/80">
                  第 {{ currentPage }} / {{ totalPages }} 页
                </span>
                <button @click="nextPage" :disabled="currentPage === totalPages"
                  class="px-4 py-2 text-sm font-medium bg-white/5 border border-white/20 text-white/80 rounded-lg hover:bg-white/10 disabled:opacity-50 disabled:cursor-not-allowed transition-colors">下一页</button>
              </div>
            </div>
          </div>
        </div>
      </Motion>
    </div>
  </Motion>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { Motion } from 'motion-v'
import { getDetectionRecords, getWarnings } from '@/api/inspection/API'
import type { DetectionRecord } from '@/types/apis/inspection_T'
import { jsPDF } from 'jspdf'

// Interfaces
interface Filters {
  startDate: string
  endDate: string
  level: string
  camera: string
}

interface Stats {
  severe: number
  warning: number
  info: number
  completionRate: number
}

// Animation Variants
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

// Reactive Data
const filters = ref<Filters>({ startDate: '', endDate: '', level: '', camera: '' })
const selectedRecords = ref<number[]>([])
const currentPage = ref(1)
const pageSize = ref(10)
const allRecords = ref<DetectionRecord[]>([])
const filteredRecords = ref<DetectionRecord[]>([])
const cameraOptions = ref<string[]>([])

// Computed Properties
const stats = computed<Stats>(() => {
  const result = { severe: 0, warning: 0, info: 0, completionRate: 99.8 } // Static rate for demo
  filteredRecords.value.forEach(record => {
    if (record.confidence < 0.7) result.severe++
    else if (record.confidence < 0.9) result.warning++
    else result.info++
  })
  return result
})

const totalRecords = computed(() => filteredRecords.value.length)
const totalPages = computed(() => Math.max(1, Math.ceil(totalRecords.value / pageSize.value)))
const startIndex = computed(() => totalRecords.value === 0 ? 0 : (currentPage.value - 1) * pageSize.value + 1)
const endIndex = computed(() => Math.min(currentPage.value * pageSize.value, totalRecords.value))
const paginatedRecords = computed(() => {
  const start = (currentPage.value - 1) * pageSize.value
  return filteredRecords.value.slice(start, start + pageSize.value)
})

// Methods
const getRecordStyle = (type: string) => {
  if (type === 'no_hat') {
    return {
      title: '未佩戴安全帽',
      statusText: '违规',
      cardClass: 'bg-red-500/10 border-red-400/30 hover:bg-red-500/20',
      badgeClass: 'bg-red-500/30 text-red-200 border border-red-400/40',
      accentClass: 'bg-gradient-to-b from-red-400 to-red-600',
      confidenceClass: 'text-red-300'
    }
  }
  if (type === 'wearing_hat') {
    return {
      title: '佩戴安全帽',
      statusText: '合规',
      cardClass: 'bg-green-500/10 border-green-400/30 hover:bg-green-500/20',
      badgeClass: 'bg-green-500/30 text-green-200 border border-green-400/40',
      accentClass: 'bg-gradient-to-b from-green-400 to-green-600',
      confidenceClass: 'text-green-300'
    }
  }
  return {
    title: '人员检测',
    statusText: '监测',
    cardClass: 'bg-blue-500/10 border-blue-400/30 hover:bg-blue-500/20',
    badgeClass: 'bg-blue-500/30 text-blue-200 border border-blue-400/40',
    accentClass: 'bg-gradient-to-b from-blue-400 to-blue-600',
    confidenceClass: 'text-blue-300'
  }
}

const fetchDetectionRecords = async () => {
  try {
    const res = await getDetectionRecords({ page: 1, page_size: 1000 }) // Fetch more for better filtering demo
    allRecords.value = res.data.records
    filteredRecords.value = allRecords.value
    const cameras = new Set(allRecords.value.map(record => record.camera_name))
    cameraOptions.value = Array.from(cameras)
  } catch (error) {
    console.error('获取检测记录失败:', error)
  }
}

const formatDateTime = (datetime: string): string => new Date(datetime).toLocaleString('zh-CN', {
  year: 'numeric', month: '2-digit', day: '2-digit',
  hour: '2-digit', minute: '2-digit', second: '2-digit'
})

const applyFilters = (): void => {
  const startTime = filters.value.startDate ? new Date(filters.value.startDate).getTime() : -Infinity
  const endTime = filters.value.endDate ? new Date(filters.value.endDate).setHours(23, 59, 59, 999) : Infinity

  filteredRecords.value = allRecords.value.filter(record => {
    const recordTime = new Date(record.detected_at).getTime()
    const matchCamera = !filters.value.camera || record.camera_name === filters.value.camera
    const matchType = !filters.value.level || record.detection_type === filters.value.level
    const matchDate = recordTime >= startTime && recordTime <= endTime
    return matchCamera && matchType && matchDate
  })
  currentPage.value = 1
  selectedRecords.value = []
}

const previousPage = () => { if (currentPage.value > 1) currentPage.value-- }
const nextPage = () => { if (currentPage.value < totalPages.value) currentPage.value++ }

// Export and Report Functions
const exportToCSV = (data: any[], filename: string, headers: Record<string, string>): void => {
  const headerKeys = Object.keys(headers)
  const headerLabels = Object.values(headers)
  const csvContent = [
    headerLabels.join(','),
    ...data.map(row => headerKeys.map(key => {
      let value = row[key]
      if (key === 'detection_type') value = getRecordStyle(value).title
      if (key === 'confidence') value = `${(value * 100).toFixed(1)}%`
      if (key.includes('_at')) value = formatDateTime(value)
      return `"${String(value || '').replace(/"/g, '""')}"`
    }).join(','))
  ].join('\n')
  const blob = new Blob(['\uFEFF' + csvContent], { type: 'text/csv;charset=utf-8' })
  const link = document.createElement('a')
  link.href = URL.createObjectURL(blob)
  link.download = `${filename}_${new Date().toISOString().split('T')[0]}.csv`
  link.click()
  URL.revokeObjectURL(link.href)
}

const exportRecords = () => {
  if (filteredRecords.value.length === 0) return alert('没有可导出的记录')
  exportToCSV(filteredRecords.value, '检测记录', {
    id: 'ID', detection_type: '检测类型', camera_name: '摄像头', confidence: '置信度', detected_at: '检测时间'
  })
}

const exportWarnings = async () => {
  try {
    const res = await getWarnings({ page: 1, page_size: 1000 })
    if (!res.data.warnings?.length) return alert('没有可导出的警告记录')
    exportToCSV(res.data.warnings, '警告记录', {
      id: 'ID', title: '标题', warning_level: '级别', camera_name: '摄像头', created_at: '时间'
    })
  } catch (error) { console.error('导出警告失败:', error) }
}

const deleteSelected = () => {
  if (selectedRecords.value.length === 0) return alert('请先选择要删除的记录')
  alert(`删除选中的 ${selectedRecords.value.length} 条记录 (模拟)`)
  // Logic to delete records would go here
}

const generateReport = () => {
  try {
    const doc = new jsPDF()
    doc.setFont('helvetica', 'normal'); // Basic font support
    doc.text('Safety Detection Report', 20, 20)
    doc.text(`Generated on: ${new Date().toLocaleDateString()}`, 20, 30)
    // Add more content...
    doc.save(`report_${new Date().toISOString().split('T')[0]}.pdf`)
  } catch (error) {
    console.error('生成PDF报告失败:', error)
    alert('生成报告失败。')
  }
}

// Lifecycle Hooks
onMounted(() => {
  fetchDetectionRecords()
})

onUnmounted(() => {
  // Cleanup if needed
})
</script>

<style scoped>
/* Custom scrollbar for glassmorphism theme */
.custom-scrollbar::-webkit-scrollbar {
  width: 6px;
}

.custom-scrollbar::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 3px;
}

.custom-scrollbar::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.3);
  border-radius: 3px;
}

/* Enhance glassmorphism effect */
@supports (backdrop-filter: blur()) {
  .backdrop-blur-xl {
    backdrop-filter: blur(20px);
  }
}

/* Custom styles for themed checkbox without @apply */
.checkbox-glass {
  appearance: none;
  width: 1.25rem;
  /* w-5 */
  height: 1.25rem;
  /* h-5 */
  border-radius: 0.25rem;
  /* rounded */
  background-color: rgba(0, 0, 0, 0.2);
  /* bg-black/20 */
  border-width: 2px;
  border-color: rgba(255, 255, 255, 0.3);
  /* border-white/30 */
  transition-property: background-color, border-color, background-size;
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
  transition-duration: 200ms;
  cursor: pointer;
  background-image: url("data:image/svg+xml,%3csvg viewBox='0 0 16 16' fill='white' xmlns='http://www.w3.org/2000/svg'%3e%3cpath d='M12.207 4.793a1 1 0 010 1.414l-5 5a1 1 0 01-1.414 0l-2-2a1 1 0 011.414-1.414L6.5 9.086l4.293-4.293a1 1 0 011.414 0z'/%3e%3c/svg%3e");
  background-size: 0;
  background-repeat: no-repeat;
  background-position: center;
}

.checkbox-glass:checked {
  background-color: #3b82f6;
  /* checked:bg-blue-500 */
  border-color: #60a5fa;
  /* checked:border-blue-400 */
  background-size: 100% 100%;
}

/* Remove default date/time input icon for better styling control */
input[type="date"]::-webkit-calendar-picker-indicator {
  filter: invert(0.8);
  cursor: pointer;
}
</style>
