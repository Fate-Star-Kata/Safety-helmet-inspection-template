<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { Motion } from 'motion-v'
import { ElMessage, ElMessageBox } from 'element-plus'
import {
  Search,
  Refresh,
  Filter,
  Download,
  View,
  Edit,
  Delete,
  Warning,
  Check,
  Close,
  VideoCamera,
  Location,
  Clock,
  User
} from '@element-plus/icons-vue'

// 数据接口定义
interface WarningRecord {
  id: string
  cameraId: string
  cameraName: string
  location: string
  type: 'no_helmet' | 'improper_helmet' | 'multiple_violations'
  severity: 'low' | 'medium' | 'high' | 'critical'
  status: 'pending' | 'processing' | 'resolved' | 'ignored'
  detectedAt: string
  processedAt?: string
  processedBy?: string
  description: string
  imageUrl: string
  confidence: number
  personCount: number
  violationCount: number
  notes?: string
}

interface FilterOptions {
  dateRange: [string, string] | null
  cameraId: string
  type: string
  severity: string
  status: string
  keyword: string
}

// 动画配置
const pageVariants = {
  initial: { opacity: 0, y: 30 },
  animate: { opacity: 1, y: 0 },
  transition: { duration: 0.6, ease: 'easeOut' }
}

// 响应式数据
const tableLoading = ref(false)
const selectedRows = ref<WarningRecord[]>([])
const currentPage = ref(1)
const pageSize = ref(10)
const total = ref(0)

// 筛选条件
const filters = ref<FilterOptions>({
  dateRange: null,
  cameraId: '',
  type: '',
  severity: '',
  status: '',
  keyword: ''
})

// 警告记录数据
const warningRecords = ref<WarningRecord[]>([
  {
    id: 'W001',
    cameraId: 'CAM001',
    cameraName: '主入口摄像头',
    location: '工地大门',
    type: 'no_helmet',
    severity: 'high',
    status: 'pending',
    detectedAt: '2024-01-15 14:28:30',
    description: '检测到1名工人未佩戴安全帽',
    imageUrl: '/images/warnings/w001.jpg',
    confidence: 95.6,
    personCount: 1,
    violationCount: 1
  },
  {
    id: 'W002',
    cameraId: 'CAM002',
    cameraName: '施工区域A',
    location: '1号楼施工区',
    type: 'improper_helmet',
    severity: 'medium',
    status: 'processing',
    detectedAt: '2024-01-15 13:42:20',
    processedAt: '2024-01-15 14:10:15',
    processedBy: '张三',
    description: '检测到安全帽佩戴不规范',
    imageUrl: '/images/warnings/w002.jpg',
    confidence: 87.3,
    personCount: 1,
    violationCount: 1,
    notes: '已通知现场安全员处理'
  },
  {
    id: 'W003',
    cameraId: 'CAM003',
    cameraName: '施工区域B',
    location: '2号楼施工区',
    type: 'multiple_violations',
    severity: 'critical',
    status: 'resolved',
    detectedAt: '2024-01-15 12:15:45',
    processedAt: '2024-01-15 12:45:30',
    processedBy: '李四',
    description: '检测到多名工人违规操作',
    imageUrl: '/images/warnings/w003.jpg',
    confidence: 92.1,
    personCount: 3,
    violationCount: 3,
    notes: '已现场整改完成，加强安全培训'
  }
])

// 摄像头选项
const cameraOptions = ref([
  { value: 'CAM001', label: '主入口摄像头' },
  { value: 'CAM002', label: '施工区域A' },
  { value: 'CAM003', label: '施工区域B' },
  { value: 'CAM004', label: '材料堆放区' },
  { value: 'CAM005', label: '办公区域' }
])

// 警告类型选项
const typeOptions = [
  { value: 'no_helmet', label: '未佩戴安全帽' },
  { value: 'improper_helmet', label: '安全帽佩戴不规范' },
  { value: 'multiple_violations', label: '多项违规' }
]

// 严重程度选项
const severityOptions = [
  { value: 'low', label: '低' },
  { value: 'medium', label: '中' },
  { value: 'high', label: '高' },
  { value: 'critical', label: '严重' }
]

// 状态选项
const statusOptions = [
  { value: 'pending', label: '待处理' },
  { value: 'processing', label: '处理中' },
  { value: 'resolved', label: '已解决' },
  { value: 'ignored', label: '已忽略' }
]

// 计算属性
const filteredRecords = computed(() => {
  let result = [...warningRecords.value]
  if (filters.value.keyword) {
    const keyword = filters.value.keyword.toLowerCase()
    result = result.filter(record => record.cameraName.toLowerCase().includes(keyword) || record.location.toLowerCase().includes(keyword) || record.description.toLowerCase().includes(keyword))
  }
  if (filters.value.cameraId) {
    result = result.filter(record => record.cameraId === filters.value.cameraId)
  }
  if (filters.value.type) {
    result = result.filter(record => record.type === filters.value.type)
  }
  if (filters.value.severity) {
    result = result.filter(record => record.severity === filters.value.severity)
  }
  if (filters.value.status) {
    result = result.filter(record => record.status === filters.value.status)
  }
  if (filters.value.dateRange && filters.value.dateRange.length === 2) {
    const [startDate, endDate] = filters.value.dateRange
    result = result.filter(record => {
      const recordDate = new Date(record.detectedAt)
      return recordDate >= new Date(startDate) && recordDate <= new Date(endDate)
    })
  }
  total.value = result.length
  const start = (currentPage.value - 1) * pageSize.value
  const end = start + pageSize.value
  return result.slice(start, end)
})

const statistics = computed(() => {
  const records = warningRecords.value
  return {
    total: records.length,
    pending: records.filter(r => r.status === 'pending').length,
    processing: records.filter(r => r.status === 'processing').length,
    resolved: records.filter(r => r.status === 'resolved').length,
    critical: records.filter(r => r.severity === 'critical').length
  }
})

// 方法
const getSeverityStyle = (severity: string) => {
  return {
    low: { text: '低', class: 'text-blue-300 border-blue-400/50 bg-blue-500/20' },
    medium: { text: '中', class: 'text-yellow-300 border-yellow-400/50 bg-yellow-500/20' },
    high: { text: '高', class: 'text-orange-300 border-orange-400/50 bg-orange-500/20' },
    critical: { text: '严重', class: 'text-red-300 border-red-400/50 bg-red-500/20' },
  }[severity] || { text: '未知', class: 'text-slate-300 border-slate-400/50 bg-slate-500/20' };
};

const getStatusStyle = (status: string) => {
  return {
    pending: { text: '待处理', class: 'text-orange-300 border-orange-400/50 bg-orange-500/20' },
    processing: { text: '处理中', class: 'text-blue-300 border-blue-400/50 bg-blue-500/20' },
    resolved: { text: '已解决', class: 'text-green-300 border-green-400/50 bg-green-500/20' },
    ignored: { text: '已忽略', class: 'text-slate-300 border-slate-400/50 bg-slate-500/20' },
  }[status] || { text: '未知', class: 'text-slate-300 border-slate-400/50 bg-slate-500/20' };
};

const formatDateTime = (dateTime?: string) => {
  if (!dateTime) return 'N/A';
  return new Date(dateTime).toLocaleString('zh-CN', { year: 'numeric', month: '2-digit', day: '2-digit', hour: '2-digit', minute: '2-digit', second: '2-digit', hour12: false }).replace(/\//g, '-');
};

const handleSearch = () => {
  currentPage.value = 1
}

const handleReset = () => {
  filters.value = {
    dateRange: null,
    cameraId: '',
    type: '',
    severity: '',
    status: '',
    keyword: ''
  }
  currentPage.value = 1
}

const handleRefresh = async () => {
  tableLoading.value = true
  try {
    await new Promise(resolve => setTimeout(resolve, 1000))
    ElMessage({ message: '数据刷新成功', type: 'success', customClass: 'glass-message' });
  } catch (error) {
    ElMessage({ message: '数据刷新失败', type: 'error', customClass: 'glass-message' });
  } finally {
    tableLoading.value = false
  }
}

const handleSelectionChange = (selection: WarningRecord[]) => {
  selectedRows.value = selection
}

const handleView = (record: WarningRecord) => {
  ElMessage({ message: `查看警告记录: ${record.id}`, customClass: 'glass-message' });
}

const handleProcess = async (record: WarningRecord) => {
  try {
    await ElMessageBox.prompt('请输入处理备注', '处理警告', {
      confirmButtonText: '确定',
      cancelButtonText: '取消',
      inputPlaceholder: '请输入处理备注'
    })
    record.status = 'processing'
    record.processedAt = new Date().toISOString().slice(0, 19).replace('T', ' ')
    record.processedBy = '当前用户'
    ElMessage({ message: '警告处理成功', type: 'success', customClass: 'glass-message' });
  } catch (error) {
    // User cancelled
  }
}

const handleResolve = async (record: WarningRecord) => {
  // Logic is unchanged from your original script
  record.status = 'resolved';
  ElMessage.success('警告已解决');
}

const handleIgnore = async (record: WarningRecord) => {
  // Logic is unchanged from your original script
  record.status = 'ignored';
  ElMessage.success('警告已忽略');
}

const handleBatchProcess = async () => {
  if (selectedRows.value.length === 0) {
    ElMessage({ message: '请选择要处理的记录', type: 'warning', customClass: 'glass-message' });
    return
  }
  try {
    await ElMessageBox.confirm(`确定要批量处理选中的 ${selectedRows.value.length} 条记录吗？`, '批量处理', {
      confirmButtonText: '确定',
      cancelButtonText: '取消',
      type: 'warning'
    })
    selectedRows.value.forEach(record => {
      record.status = 'processing'
      record.processedAt = new Date().toISOString().slice(0, 19).replace('T', ' ')
      record.processedBy = '当前用户'
    })
    ElMessage({ message: `成功处理 ${selectedRows.value.length} 条记录`, type: 'success', customClass: 'glass-message' });
    selectedRows.value = []
  } catch (error) {
    // User cancelled
  }
}

const handleExport = () => {
  ElMessage({ message: '导出功能开发中...', customClass: 'glass-message' });
}

const handlePageChange = (page: number) => {
  currentPage.value = page
}

const handleSizeChange = (size: number) => {
  pageSize.value = size
  currentPage.value = 1
}
</script>

<template>
  <Motion :initial="pageVariants.initial" :animate="pageVariants.animate" :transition="pageVariants.transition as any">
    <div class="space-y-6">
      <!-- Page Header -->
      <div class="page-header">
        <div>
          <h2 class="page-title">警告管理</h2>
          <p class="page-subtitle">安全帽检测警告记录管理与处理</p>
        </div>
        <div class="flex items-center gap-2">
          <button class="action-btn" @click="handleRefresh"><el-icon>
              <Refresh />
            </el-icon>刷新</button>
          <button class="action-btn primary" @click="handleExport"><el-icon>
              <Download />
            </el-icon>导出数据</button>
        </div>
      </div>

      <!-- Statistics Cards -->
      <div class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-5 gap-6">
        <div class="stat-card from-purple-500/20 to-pink-500/20 border-purple-400/30">
          <h3 class="stat-title">总警告数</h3>
          <p class="stat-value">{{ statistics.total }}</p>
        </div>
        <div class="stat-card from-orange-500/20 to-amber-500/20 border-orange-400/30">
          <h3 class="stat-title">待处理</h3>
          <p class="stat-value">{{ statistics.pending }}</p>
        </div>
        <div class="stat-card from-blue-500/20 to-cyan-500/20 border-blue-400/30">
          <h3 class="stat-title">处理中</h3>
          <p class="stat-value">{{ statistics.processing }}</p>
        </div>
        <div class="stat-card from-green-500/20 to-emerald-500/20 border-green-400/30">
          <h3 class="stat-title">已解决</h3>
          <p class="stat-value">{{ statistics.resolved }}</p>
        </div>
        <div class="stat-card from-red-500/20 to-rose-500/20 border-red-400/30">
          <h3 class="stat-title">严重警告</h3>
          <p class="stat-value">{{ statistics.critical }}</p>
        </div>
      </div>

      <!-- Filter Panel -->
      <div class="glass-card">
        <div class="card-header">
          <h3><el-icon class="mr-2">
              <Filter />
            </el-icon>筛选条件</h3>
        </div>
        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-4 items-end">
          <el-input v-model="filters.keyword" placeholder="关键词搜索" clearable class="glass-input" />
          <el-select v-model="filters.cameraId" placeholder="选择摄像头" clearable class="glass-select"
            popper-class="glass-select-popper"><el-option v-for="cam in cameraOptions" :key="cam.value"
              :label="cam.label" :value="cam.value" /></el-select>
          <el-select v-model="filters.type" placeholder="选择类型" clearable class="glass-select"
            popper-class="glass-select-popper"><el-option v-for="t in typeOptions" :key="t.value" :label="t.label"
              :value="t.value" /></el-select>
          <el-select v-model="filters.severity" placeholder="选择严重程度" clearable class="glass-select"
            popper-class="glass-select-popper"><el-option v-for="s in severityOptions" :key="s.value" :label="s.label"
              :value="s.value" /></el-select>
          <el-select v-model="filters.status" placeholder="选择状态" clearable class="glass-select"
            popper-class="glass-select-popper"><el-option v-for="s in statusOptions" :key="s.value" :label="s.label"
              :value="s.value" /></el-select>
          <el-date-picker v-model="filters.dateRange" type="datetimerange" range-separator="至" start-placeholder="开始日期"
            end-placeholder="结束日期" class="glass-date-picker" popper-class="glass-date-popper" />
          <div class="flex items-center gap-2">
            <button class="action-btn w-full" @click="handleReset"><el-icon>
                <Refresh />
              </el-icon>重置</button>
            <button class="action-btn primary w-full" @click="handleSearch"><el-icon>
                <Search />
              </el-icon>搜索</button>
          </div>
        </div>
      </div>

      <!-- Data Table -->
      <div class="glass-card">
        <div class="card-header">
          <h3>警告记录列表</h3>
          <button class="action-btn primary" @click="handleBatchProcess" :disabled="selectedRows.length === 0">批量处理 ({{
            selectedRows.length }})</button>
        </div>
        <el-table :data="filteredRecords" v-loading="tableLoading" @selection-change="handleSelectionChange" stripe
          class="warnings-table">
          <el-table-column type="selection" width="55" />
          <el-table-column prop="id" label="警告ID" width="100" />
          <el-table-column label="摄像头信息" min-width="180">
            <template #default="{ row }">
              <div class="font-medium flex items-center gap-2"><el-icon>
                  <VideoCamera />
                </el-icon>{{ row.cameraName }}</div>
              <div class="text-xs text-white/60 mt-1 flex items-center gap-2"><el-icon>
                  <Location />
                </el-icon>{{ row.location }}</div>
            </template>
          </el-table-column>
          <el-table-column label="严重程度" width="100" align="center">
            <template #default="{ row }"><span class="status-tag" :class="getSeverityStyle(row.severity).class">{{
              getSeverityStyle(row.severity).text }}</span></template>
          </el-table-column>
          <el-table-column label="状态" width="100" align="center">
            <template #default="{ row }"><span class="status-tag" :class="getStatusStyle(row.status).class">{{
              getStatusStyle(row.status).text }}</span></template>
          </el-table-column>
          <el-table-column prop="description" label="描述" min-width="200" />
          <el-table-column label="检测时间" width="180">
            <template #default="{ row }">{{ formatDateTime(row.detectedAt) }}</template>
          </el-table-column>
          <el-table-column label="处理信息" width="180">
            <template #default="{ row }">
              <div v-if="row.processedAt" class="text-xs">
                <div>{{ formatDateTime(row.processedAt) }}</div>
                <div class="text-white/60 mt-1 flex items-center gap-1"><el-icon>
                    <User />
                  </el-icon>{{ row.processedBy }}</div>
              </div>
              <span v-else class="text-white/50 text-xs">未处理</span>
            </template>
          </el-table-column>
          <el-table-column label="操作" width="120" fixed="right" align="center">
            <template #default="{ row }">
              <div class="flex items-center justify-center gap-2">
                <button class="action-btn small" @click="handleView(row)"><el-icon>
                    <View />
                  </el-icon></button>
                <button v-if="row.status === 'pending'" class="action-btn small primary"
                  @click="handleProcess(row)"><el-icon>
                    <Check />
                  </el-icon></button>
              </div>
            </template>
          </el-table-column>
        </el-table>
        <div class="mt-6 flex justify-end">
          <el-pagination v-model:current-page="currentPage" v-model:page-size="pageSize" :page-sizes="[10, 20, 50]"
            :total="total" layout="total, sizes, prev, pager, next, jumper" />
        </div>
      </div>
    </div>
  </Motion>
</template>

<style scoped>
.page-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.page-title {
  font-size: 1.75rem;
  font-weight: 600;
  color: white;
}

.page-subtitle {
  font-size: 0.875rem;
  color: rgba(255, 255, 255, 0.7);
  margin-top: 0.25rem;
}

.stat-card {
  backdrop-filter: blur(12px);
  background-image: linear-gradient(to bottom right, var(--tw-gradient-stops));
  border-radius: 0.75rem;
  padding: 1rem;
  border: 1px solid;
  color: white;
  text-align: center;
}

.stat-title {
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.7);
}

.stat-value {
  font-size: 1.5rem;
  font-weight: 600;
}

.glass-card {
  backdrop-filter: blur(20px);
  background-color: rgba(255, 255, 255, 0.05);
  border-radius: 1rem;
  border: 1px solid rgba(255, 255, 255, 0.2);
  padding: 1.5rem;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.5rem;
  color: white;
  font-size: 1.125rem;
  font-weight: 600;
}

.action-btn {
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
  transition: all 0.2s;
}

.action-btn:hover {
  background-color: rgba(255, 255, 255, 0.2);
}

.action-btn.primary {
  background-image: linear-gradient(to right, rgba(99, 102, 241, 0.6), rgba(139, 92, 246, 0.6));
  border-color: rgba(139, 92, 246, 0.8);
}

.action-btn.small {
  padding: 0.4rem;
}

.action-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
  background-image: none;
  background-color: rgba(255, 255, 255, 0.05);
}

.warnings-table {
  background-color: transparent;
}

:deep(.el-table__header-wrapper) {
  background-color: rgba(255, 255, 255, 0.1);
}

:deep(.el-table th.el-table__cell) {
  background-color: transparent;
  color: rgba(255, 255, 255, 0.8);
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);
}

:deep(.el-table tr) {
  background-color: transparent;
  color: rgba(255, 255, 255, 0.9);
}

:deep(.el-table tr:hover > td.el-table__cell) {
  background-color: rgba(255, 255, 255, 0.05);
}

:deep(.el-table td.el-table__cell) {
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

:deep(.el-table .el-table__cell.is-leaf) {
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);
}

:deep(.warnings-table.el-table--striped .el-table__body tr.el-table__row--striped td.el-table__cell) {
  background-color: rgba(255, 255, 255, 0.03);
}

.status-tag {
  display: inline-block;
  padding: 0.2rem 0.6rem;
  border-radius: 9999px;
  font-size: 0.75rem;
  border: 1px solid;
}

:deep(.el-pagination) {
  color: rgba(255, 255, 255, 0.9) !important;
}

:deep(.el-pagination button),
:deep(.el-pager li) {
  background: transparent;
  color: rgba(255, 255, 255, 0.9) !important;
}

:deep(.el-pager li.is-active) {
  background-color: #4f46e5;
  color: white;
}

/* 分页组件选择器样式 */
:deep(.el-pagination .el-select .el-select__wrapper) {
  background-color: rgba(0, 0, 0, 0.3) !important;
  border: 1px solid rgba(255, 255, 255, 0.2) !important;
  backdrop-filter: blur(8px);
  border-radius: 6px;
}

:deep(.el-pagination .el-select .el-select__placeholder) {
  color: rgba(255, 255, 255, 0.9) !important;
}

:deep(.el-pagination .el-input .el-input__wrapper) {
  background-color: rgba(0, 0, 0, 0.3) !important;
  border: 1px solid rgba(255, 255, 255, 0.2) !important;
  backdrop-filter: blur(8px);
  border-radius: 6px;
}

:deep(.el-pagination .el-input .el-input__inner) {
  color: rgba(255, 255, 255, 0.9) !important;
}

:deep(.el-pagination__total),
:deep(.el-pagination__jump) {
  color: rgba(255, 255, 255, 0.9) !important;
}

/* 玻璃化输入框样式 */
:deep(.glass-input .el-input__wrapper) {
  background-color: rgba(0, 0, 0, 0.2) !important;
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.2) !important;
  box-shadow: none !important;
  backdrop-filter: blur(8px);
}

:deep(.glass-input .el-input__inner) {
  color: rgba(255, 255, 255, 0.9) !important;
  background-color: transparent !important;
}

/* 玻璃化选择器样式 */
:deep(.glass-select .el-select__wrapper) {
  background-color: rgba(0, 0, 0, 0.2) !important;
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.2) !important;
  box-shadow: none !important;
  backdrop-filter: blur(8px);
}

:deep(.glass-select .el-select__placeholder) {
  color: rgba(255, 255, 255, 0.7) !important;
}

:deep(.glass-select .el-select__selected-item) {
  color: rgba(255, 255, 255, 0.9) !important;
}

:deep(.glass-select .el-select__suffix) {
  color: rgba(255, 255, 255, 0.7) !important;
}

:deep(.glass-select .el-select__caret) {
  color: rgba(255, 255, 255, 0.7) !important;
}

/* 增强日期选择器样式 */
:deep(.glass-date-picker.el-date-editor) {
  background-color: rgba(0, 0, 0, 0.2) !important;
  border: 1px solid rgba(255, 255, 255, 0.2) !important;
  backdrop-filter: blur(8px);
  border-radius: 8px;
  width: 240px !important;
}

:deep(.glass-date-picker.el-date-editor .el-input__wrapper) {
  background-color: transparent !important;
  border: none !important;
  box-shadow: none !important;
}

:deep(.glass-date-picker .el-range-input) {
  background-color: transparent !important;
  color: rgba(255, 255, 255, 0.9) !important;
}

:deep(.glass-date-picker .el-range-separator) {
  color: rgba(255, 255, 255, 0.7) !important;
}

:deep(.glass-date-picker .el-input__suffix) {
  color: rgba(255, 255, 255, 0.7) !important;
}

:deep(.glass-date-picker .el-input__suffix .el-input__suffix-inner) {
  color: rgba(255, 255, 255, 0.7) !important;
}

:deep(.glass-date-picker .el-input__suffix .el-input__suffix-inner .el-icon) {
  color: rgba(255, 255, 255, 0.7) !important;
}

.glass-select-popper.el-popper,
.glass-date-popper.el-popper {
  background: rgba(30, 30, 45, 0.8) !important;
  border: 1px solid rgba(255, 255, 255, 0.2) !important;
  backdrop-filter: blur(12px);
}

.glass-select-popper .el-select-dropdown__item,
.glass-date-popper .el-date-table td {
  color: rgba(255, 255, 255, 0.8);
}

.glass-select-popper .el-select-dropdown__item.hover,
.glass-select-popper .el-select-dropdown__item:hover,
.glass-date-popper .el-date-table td .el-date-table-cell:hover {
  background-color: rgba(255, 255, 255, 0.1);
}

.glass-date-popper .el-date-picker__header,
.glass-date-popper .el-picker-panel__icon-btn,
.glass-date-popper .el-date-table th {
  color: rgba(255, 255, 255, 0.9);
}

.glass-date-popper .el-date-table td.today .el-date-table-cell__text {
  color: #67e8f9;
  font-weight: bold;
}

.glass-date-popper .el-date-table td.current .el-date-table-cell__text,
.glass-date-popper .el-date-table td.in-range .el-date-table-cell,
.glass-date-popper .el-date-table td.end-date .el-date-table-cell,
.glass-date-popper .el-date-table td.start-date .el-date-table-cell {
  background-color: #4f46e5;
  border-radius: 50%;
}

.glass-message {
  background-color: rgba(30, 30, 45, 0.8) !important;
  border: 1px solid rgba(255, 255, 255, 0.2) !important;
  backdrop-filter: blur(10px);
  color: white !important;
}
</style>
