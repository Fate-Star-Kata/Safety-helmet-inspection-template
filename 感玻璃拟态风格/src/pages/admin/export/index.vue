<script setup lang="ts">
import { ref, reactive, onMounted, computed } from 'vue'
import { Motion } from 'motion-v'
import { ElMessage, ElMessageBox } from 'element-plus'
import {
  Download,
  Document,
  Calendar,
  Filter,
  Refresh,
  DocumentCopy,
  Files,
  Histogram,
  PieChart,
  TrendCharts,
  DataAnalysis,
  Clock,
  User,
  Camera,
  Warning
} from '@element-plus/icons-vue'

// 数据接口定义
interface ExportTask {
  id: string
  name: string
  type: 'detection_records' | 'warning_logs' | 'system_reports' | 'user_activity'
  format: 'excel' | 'pdf' | 'csv' | 'json'
  status: 'pending' | 'processing' | 'completed' | 'failed'
  progress: number
  created_at: string
  completed_at?: string
  file_size?: string
  download_url?: string
  parameters: {
    date_range: [string, string]
    filters?: Record<string, any>
  }
}

interface ReportTemplate {
  id: string
  name: string
  description: string
  type: 'daily' | 'weekly' | 'monthly' | 'custom'
  format: 'pdf' | 'excel'
  sections: string[]
  enabled: boolean
  schedule?: string
}

// 动画配置
const pageVariants = {
  initial: { opacity: 0, y: 30 },
  animate: { opacity: 1, y: 0 },
  transition: { duration: 0.6, ease: 'easeOut' }
}

// 响应式数据
const loading = ref(false)
const activeTab = ref('export')
const exportTasks = ref<ExportTask[]>([
  {
    id: 'task_001',
    name: '2024年1月检测记录',
    type: 'detection_records',
    format: 'excel',
    status: 'completed',
    progress: 100,
    created_at: '2024-01-15 10:30:00',
    completed_at: '2024-01-15 10:32:15',
    file_size: '2.5MB',
    download_url: '/downloads/detection_records_202401.xlsx',
    parameters: {
      date_range: ['2024-01-01', '2024-01-31']
    }
  },
  {
    id: 'task_002',
    name: '警告日志导出',
    type: 'warning_logs',
    format: 'pdf',
    status: 'processing',
    progress: 65,
    created_at: '2024-01-16 14:20:00',
    parameters: {
      date_range: ['2024-01-10', '2024-01-16']
    }
  }
])

// 报告模板
const reportTemplates = ref<ReportTemplate[]>([
  {
    id: 'template_001',
    name: '日报模板',
    description: '每日安全检测汇总报告',
    type: 'daily',
    format: 'pdf',
    sections: ['检测统计', '警告汇总', '设备状态', '趋势分析'],
    enabled: true,
    schedule: '0 18 * * *'
  },
  {
    id: 'template_002',
    name: '周报模板',
    description: '每周安全检测详细报告',
    type: 'weekly',
    format: 'excel',
    sections: ['周度统计', '违规分析', '改进建议', '对比分析'],
    enabled: true,
    schedule: '0 9 * * 1'
  }
])

// 导出表单
const quickExportForm = reactive({
  type: 'detection_records',
  format: 'excel',
  date_range: ['', ''],
  name: ''
})

// 选项数据
const exportTypeOptions = [
  { label: '检测记录', value: 'detection_records', icon: Camera },
  { label: '警告日志', value: 'warning_logs', icon: Warning },
  { label: '系统报告', value: 'system_reports', icon: Document },
  { label: '用户活动', value: 'user_activity', icon: User }
]
const formatOptions = [
  { label: 'Excel表格', value: 'excel' },
  { label: 'PDF文档', value: 'pdf' },
  { label: 'CSV文件', value: 'csv' },
  { label: 'JSON数据', value: 'json' }
]

const showQuickExportDialog = ref(false)
const showTemplateDialog = ref(false)
const editingTemplate = ref<ReportTemplate | null>(null)

// 计算属性
const completedTasks = computed(() => exportTasks.value.filter(task => task.status === 'completed'))
const processingTasks = computed(() => exportTasks.value.filter(task => task.status === 'processing'))

// 方法
const getStatusStyle = (status: string) => ({
  pending: { text: '等待中', class: 'text-blue-300' },
  processing: { text: '处理中', class: 'text-yellow-300' },
  completed: { text: '已完成', class: 'text-green-300' },
  failed: { text: '失败', class: 'text-red-300' },
}[status] || { text: '未知', class: 'text-slate-300' });

const getTypeText = (type: string) => ({
  detection_records: '检测记录',
  warning_logs: '警告日志',
  system_reports: '系统报告',
  user_activity: '用户活动'
}[type] || '未知类型');

const getFormatIcon = (format: string) => ({ excel: '📊', pdf: '📄', csv: '📋', json: '🔧' }[format] || '📄');

const handleQuickExport = () => {
  const endDate = new Date()
  const startDate = new Date()
  startDate.setDate(endDate.getDate() - 7)
  quickExportForm.date_range = [startDate.toISOString().split('T')[0], endDate.toISOString().split('T')[0]]
  showQuickExportDialog.value = true
}

const handleStartExport = async () => {
  if (!quickExportForm.name || !quickExportForm.date_range[0] || !quickExportForm.date_range[1]) {
    ElMessage({ message: '请填写所有必填项', type: 'warning', customClass: 'glass-message' });
    return
  }
  loading.value = true
  try {
    await new Promise(resolve => setTimeout(resolve, 1000))
    const newTask: ExportTask = { id: `task_${Date.now()}`, name: quickExportForm.name, type: quickExportForm.type as any, format: quickExportForm.format as any, status: 'processing', progress: 0, created_at: new Date().toLocaleString(), parameters: { date_range: [...quickExportForm.date_range] as [string, string] } }
    exportTasks.value.unshift(newTask)
    showQuickExportDialog.value = false
    simulateProgress(newTask)
    ElMessage({ message: '导出任务已创建', type: 'success', customClass: 'glass-message' });
  } catch (error) {
    ElMessage({ message: '创建导出任务失败', type: 'error', customClass: 'glass-message' });
  } finally {
    loading.value = false
  }
}

const simulateProgress = (task: ExportTask) => {
  const interval = setInterval(() => {
    task.progress += Math.random() * 20
    if (task.progress >= 100) {
      task.progress = 100
      task.status = 'completed'
      task.completed_at = new Date().toLocaleString()
      task.file_size = `${(Math.random() * 5 + 1).toFixed(1)}MB`
      task.download_url = `/downloads/${task.name.replace(/\s+/g, '_')}.${task.format}`
      clearInterval(interval)
      ElMessage({ message: `导出任务"${task.name}"已完成`, type: 'success', customClass: 'glass-message' });
    }
  }, 1000)
}

const handleDownload = (task: ExportTask) => {
  if (task.status === 'completed' && task.download_url) {
    ElMessage({ message: `开始下载: ${task.name}`, type: 'success', customClass: 'glass-message' });
  }
}

const handleDeleteTask = async (task: ExportTask) => {
  try {
    await ElMessageBox.confirm(`确定要删除导出任务"${task.name}"吗？`, '删除任务', { confirmButtonText: '确定', cancelButtonText: '取消', type: 'warning' })
    const index = exportTasks.value.findIndex(t => t.id === task.id)
    if (index > -1) {
      exportTasks.value.splice(index, 1)
      ElMessage({ message: '任务删除成功', type: 'success', customClass: 'glass-message' });
    }
  } catch (error) { /* User cancelled */ }
}

const handleRetryTask = async (task: ExportTask) => {
  task.status = 'processing'
  task.progress = 0
  simulateProgress(task)
  ElMessage({ message: '任务已重新开始', type: 'success', customClass: 'glass-message' });
}

const handleGenerateReport = async (template: ReportTemplate) => {
  loading.value = true
  try {
    await new Promise(resolve => setTimeout(resolve, 2000))
    const reportTask: ExportTask = { id: `report_${Date.now()}`, name: `${template.name}_${new Date().toLocaleDateString()}`, type: 'system_reports', format: template.format as any, status: 'processing', progress: 0, created_at: new Date().toLocaleString(), parameters: { date_range: ['', ''] } }
    exportTasks.value.unshift(reportTask)
    simulateProgress(reportTask)
    ElMessage({ message: `报告"${template.name}"生成中`, type: 'success', customClass: 'glass-message' });
  } catch (error) {
    ElMessage({ message: '报告生成失败', type: 'error', customClass: 'glass-message' });
  } finally {
    loading.value = false
  }
}

const handleToggleTemplate = (template: ReportTemplate) => {
  template.enabled = !template.enabled
  ElMessage({ message: `模板"${template.name}"已${template.enabled ? '启用' : '禁用'}`, type: 'success', customClass: 'glass-message' });
}

const handleEditTemplate = (template: ReportTemplate) => {
  editingTemplate.value = { ...template }
  showTemplateDialog.value = true
}

const handleSaveTemplate = () => {
  if (!editingTemplate.value) return
  const index = reportTemplates.value.findIndex(t => t.id === editingTemplate.value!.id)
  if (index > -1) {
    reportTemplates.value[index] = { ...editingTemplate.value }
    ElMessage({ message: '模板保存成功', type: 'success', customClass: 'glass-message' });
  }
  showTemplateDialog.value = false
  editingTemplate.value = null
}

const handleRefreshTasks = async () => {
  loading.value = true
  try {
    await new Promise(resolve => setTimeout(resolve, 1000))
    ElMessage({ message: '任务列表已刷新', type: 'success', customClass: 'glass-message' });
  } catch (error) {
    ElMessage({ message: '刷新失败', type: 'error', customClass: 'glass-message' });
  } finally {
    loading.value = false
  }
}

const resetQuickExportForm = () => {
  Object.assign(quickExportForm, { type: 'detection_records', format: 'excel', date_range: ['', ''], name: '' })
}
</script>

<template>
  <Motion :initial="pageVariants.initial" :animate="pageVariants.animate" :transition="pageVariants.transition as any">
    <div class="space-y-6">
      <!-- Page Header -->
      <div class="page-header">
        <div>
          <h2 class="page-title">数据导出中心</h2>
          <p class="page-subtitle">导出检测数据、生成分析报告</p>
        </div>
        <div class="flex items-center gap-2">
          <button class="action-btn" @click="handleRefreshTasks" :loading="loading"><el-icon>
              <Refresh />
            </el-icon>刷新</button>
          <button class="action-btn primary" @click="handleQuickExport"><el-icon>
              <Download />
            </el-icon>快速导出</button>
        </div>
      </div>

      <!-- Statistics Cards -->
      <div class="grid grid-cols-2 md:grid-cols-4 gap-6">
        <div class="stat-card from-purple-500/20 to-pink-500/20 border-purple-400/30">
          <div class="stat-icon">
            <Files />
          </div>
          <div>
            <p class="stat-value">{{ exportTasks.length }}</p>
            <h3 class="stat-title">总任务数</h3>
          </div>
        </div>
        <div class="stat-card from-yellow-500/20 to-amber-500/20 border-yellow-400/30">
          <div class="stat-icon">
            <Clock />
          </div>
          <div>
            <p class="stat-value">{{ processingTasks.length }}</p>
            <h3 class="stat-title">处理中</h3>
          </div>
        </div>
        <div class="stat-card from-blue-500/20 to-cyan-500/20 border-blue-400/30">
          <div class="stat-icon">
            <DocumentCopy />
          </div>
          <div>
            <p class="stat-value">{{ completedTasks.length }}</p>
            <h3 class="stat-title">已完成</h3>
          </div>
        </div>
        <div class="stat-card from-green-500/20 to-emerald-500/20 border-green-400/30">
          <div class="stat-icon">
            <Document />
          </div>
          <div>
            <p class="stat-value">{{reportTemplates.filter(t => t.enabled).length}}</p>
            <h3 class="stat-title">活跃模板</h3>
          </div>
        </div>
      </div>

      <!-- Main Content -->
      <div class="glass-card p-0 sm:p-0">
        <el-tabs v-model="activeTab" class="export-tabs">
          <el-tab-pane label="导出任务" name="export">
            <template #label><span class="tab-label"><el-icon>
                  <Download />
                </el-icon>导出任务</span></template>
            <div class="p-6 space-y-4">
              <div v-for="task in exportTasks" :key="task.id" class="task-card">
                <div class="flex flex-wrap gap-4 justify-between items-start">
                  <div>
                    <h4 class="task-name">{{ task.name }}</h4>
                    <div class="task-meta">
                      <span class="status-tag" :class="getStatusStyle(task.status).class">{{
                        getStatusStyle(task.status).text
                        }}</span>
                      <span class="meta-item">{{ getTypeText(task.type) }}</span>
                      <span class="meta-item">{{ getFormatIcon(task.format) }} {{ task.format.toUpperCase() }}</span>
                    </div>
                  </div>
                  <div class="flex items-center gap-2 flex-shrink-0">
                    <button v-if="task.status === 'completed'" class="action-btn small primary"
                      @click="handleDownload(task)"><el-icon>
                        <Download />
                      </el-icon>下载</button>
                    <button v-if="task.status === 'failed'" class="action-btn small"
                      @click="handleRetryTask(task)"><el-icon>
                        <Refresh />
                      </el-icon>重试</button>
                    <button class="action-btn small" @click="handleDeleteTask(task)">删除</button>
                  </div>
                </div>
                <div v-if="task.status === 'processing'" class="mt-4"><el-progress
                    :percentage="Math.round(task.progress)" :stroke-width="6" /></div>
                <div class="task-details">
                  <p><span>创建:</span><span>{{ task.created_at }}</span></p>
                  <p v-if="task.completed_at"><span>完成:</span><span>{{ task.completed_at }}</span></p>
                  <p v-if="task.file_size"><span>大小:</span><span>{{ task.file_size }}</span></p>
                  <p><span>范围:</span><span>{{ task.parameters.date_range.join(' ~ ') }}</span></p>
                </div>
              </div>
              <el-empty v-if="!exportTasks.length" description="暂无导出任务" />
            </div>
          </el-tab-pane>

          <el-tab-pane label="报告模板" name="templates">
            <template #label><span class="tab-label"><el-icon>
                  <Document />
                </el-icon>报告模板</span></template>
            <div class="p-6 grid grid-cols-1 lg:grid-cols-2 xl:grid-cols-3 gap-6">
              <div v-for="template in reportTemplates" :key="template.id" class="template-card"
                :class="{ 'opacity-60': !template.enabled }">
                <div class="flex justify-between items-start mb-4">
                  <div>
                    <h4 class="template-name">{{ template.name }}</h4>
                    <p class="template-description">{{ template.description }}</p>
                  </div>
                  <el-switch v-model="template.enabled" @change="handleToggleTemplate(template)" size="small" />
                </div>
                <div class="space-y-3 mb-4">
                  <div class="flex items-center gap-2"><el-tag size="small">{{ template.type }}</el-tag><el-tag
                      size="small">{{
                      template.format }}</el-tag></div>
                  <div>
                    <p class="text-xs text-white/60 mb-1">报告内容:</p>
                    <div class="flex flex-wrap gap-1"><el-tag v-for="s in template.sections" :key="s" size="small"
                        type="info">{{
                        s }}</el-tag></div>
                  </div>
                  <p v-if="template.schedule" class="text-xs"><span class="text-white/60">定时:</span> {{
                    template.schedule }}</p>
                </div>
                <div class="flex justify-end gap-2 pt-4 border-t border-white/10">
                  <button class="action-btn small" @click="handleEditTemplate(template)">编辑</button>
                  <button class="action-btn small primary" @click="handleGenerateReport(template)"
                    :disabled="!template.enabled">生成报告</button>
                </div>
              </div>
            </div>
          </el-tab-pane>
        </el-tabs>
      </div>
    </div>

    <!-- Dialogs -->
    <el-dialog v-model="showQuickExportDialog" title="快速导出" width="600px" @close="resetQuickExportForm"
      class="glass-dialog">
      <el-form :model="quickExportForm" label-width="100px" label-position="top">
        <el-form-item label="任务名称"><el-input v-model="quickExportForm.name" placeholder="请输入导出任务名称" /></el-form-item>
        <div class="grid grid-cols-2 gap-4">
          <el-form-item label="数据类型"><el-select v-model="quickExportForm.type"><el-option v-for="t in exportTypeOptions"
                :key="t.value" :label="t.label" :value="t.value" /></el-select></el-form-item>
          <el-form-item label="文件格式"><el-select v-model="quickExportForm.format"><el-option v-for="f in formatOptions"
                :key="f.value" :label="f.label" :value="f.value" /></el-select></el-form-item>
        </div>
        <el-form-item label="日期范围"><el-date-picker v-model="quickExportForm.date_range" type="daterange"
            range-separator="至" start-placeholder="开始日期" end-placeholder="结束日期" format="YYYY-MM-DD"
            value-format="YYYY-MM-DD" /></el-form-item>
      </el-form>
      <template #footer>
        <div class="flex justify-end gap-2"><button class="action-btn"
            @click="showQuickExportDialog = false">取消</button><button class="action-btn primary"
            @click="handleStartExport" :loading="loading">开始导出</button></div>
      </template>
    </el-dialog>

    <el-dialog v-model="showTemplateDialog" title="编辑报告模板" width="600px" class="glass-dialog">
      <el-form :model="editingTemplate" label-width="100px" label-position="top" v-if="editingTemplate">
        <el-form-item label="模板名称"><el-input v-model="editingTemplate.name" /></el-form-item>
        <el-form-item label="描述"><el-input v-model="editingTemplate.description" type="textarea"
            :rows="2" /></el-form-item>
        <!-- Other form items -->
      </el-form>
      <template #footer>
        <div class="flex justify-end gap-2"><button class="action-btn"
            @click="showTemplateDialog = false">取消</button><button class="action-btn primary"
            @click="handleSaveTemplate">保存</button></div>
      </template>
    </el-dialog>
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
  display: flex;
  align-items: center;
  gap: 1rem;
  backdrop-filter: blur(12px);
  background-image: linear-gradient(to bottom right, var(--tw-gradient-stops));
  border-radius: 0.75rem;
  padding: 1rem;
  border: 1px solid;
}

.stat-icon {
  width: 48px;
  height: 48px;
  border-radius: 9999px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.5rem;
  color: white;
  background-color: rgba(255, 255, 255, 0.1);
}

.stat-value {
  font-size: 1.5rem;
  font-weight: 600;
  color: white;
}

.stat-title {
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.7);
}

.glass-card {
  backdrop-filter: blur(20px);
  background-color: rgba(255, 255, 255, 0.05);
  border-radius: 1rem;
  border: 1px solid rgba(255, 255, 255, 0.2);
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
  padding: 0.25rem 0.5rem;
  font-size: 0.75rem;
}

.export-tabs {
  padding: 0;
}

.tab-label {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.task-card {
  background-color: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 0.75rem;
  padding: 1rem;
  color: white;
}

.task-name {
  font-size: 1.125rem;
  font-weight: 600;
}

.task-meta {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 0.5rem;
  margin-top: 0.5rem;
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.7);
}

.status-tag {
  padding: 0.125rem 0.5rem;
  border-radius: 9999px;
  font-weight: 500;
}

.task-details {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 0.5rem 1rem;
  font-size: 0.75rem;
  margin-top: 1rem;
  padding-top: 1rem;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
}

.task-details p {
  display: flex;
  justify-content: space-between;
}

.task-details span:first-child {
  color: rgba(255, 255, 255, 0.6);
}

.template-card {
  background-color: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 0.75rem;
  padding: 1rem;
  color: white;
}

.template-name {
  font-weight: 600;
}

.template-description {
  font-size: 0.875rem;
  color: rgba(255, 255, 255, 0.7);
}
</style>

<style>
/* Tabs */
.export-tabs .el-tabs__header {
  margin: 0;
}

.export-tabs .el-tabs__nav {
  padding: 0.5rem 1.5rem;
}

.export-tabs .el-tabs__item {
  color: rgba(255, 255, 255, 0.7);
}

.export-tabs .el-tabs__item.is-active {
  color: white;
}

.export-tabs .el-tabs__active-bar {
  background-color: #a78bfa;
}

.export-tabs .el-tabs__nav-wrap::after {
  background-color: rgba(255, 255, 255, 0.2);
}

/* Dialog and Form */
.glass-dialog {
  background: rgba(30, 30, 45, 0.8) !important;
  border: 1px solid rgba(255, 255, 255, 0.2) !important;
  backdrop-filter: blur(12px);
  border-radius: 1rem;
}

.glass-dialog .el-dialog__title {
  color: white;
}

.glass-dialog .el-dialog__headerbtn .el-icon {
  color: white;
}

.glass-dialog .el-form-item__label {
  color: rgba(255, 255, 255, 0.8);
}

.glass-dialog .el-input__wrapper,
.glass-dialog .el-select .el-select__wrapper,
.glass-dialog .el-date-editor {
  background-color: rgba(0, 0, 0, 0.2) !important;
  border: 1px solid rgba(255, 255, 255, 0.2);
  box-shadow: none !important;
}

.glass-dialog .el-input__inner,
.glass-dialog .el-select__placeholder,
.glass-dialog .el-range-input {
  color: rgba(255, 255, 255, 0.9) !important;
}
</style>
