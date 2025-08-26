<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue'
import { Motion } from 'motion-v'
import { ElMessage, ElMessageBox } from 'element-plus'
import {
  Camera,
  Warning,
  Refresh,
  Check,
  Plus,
  Delete
} from '@element-plus/icons-vue'

// 数据接口定义
interface DetectionSettings {
  confidence_threshold: number
  detection_interval: number
  image_quality: 'low' | 'medium' | 'high'
  save_detection_images: boolean
  max_detection_distance: number
  helmet_colors: string[]
  working_hours: {
    start: string
    end: string
  }
  weekend_detection: boolean
}

interface AlertRule {
  id: string
  name: string
  type: 'no_helmet' | 'improper_helmet' | 'multiple_violations'
  severity: 'low' | 'medium' | 'high' | 'critical'
  conditions: {
    min_confidence: number
    max_response_time: number
    repeat_count: number
  }
  actions: {
    email_notification: boolean
    sms_notification: boolean
    sound_alarm: boolean
    auto_record: boolean
  }
  enabled: boolean
}

// 动画配置
const pageVariants = {
  initial: { opacity: 0, y: 30 },
  animate: { opacity: 1, y: 0 },
  transition: { duration: 0.6, ease: 'easeOut' }
}

// 响应式数据
const saving = ref(false)
const activeTab = ref('detection')

// 检测设置
const detectionSettings = reactive<DetectionSettings>({
  confidence_threshold: 85,
  detection_interval: 2,
  image_quality: 'high',
  save_detection_images: true,
  max_detection_distance: 50,
  helmet_colors: ['#FF0000', '#FFFF00', '#0000FF', '#FFFFFF'],
  working_hours: { start: '08:00', end: '18:00' },
  weekend_detection: false
})

// 警告规则
const alertRules = ref<AlertRule[]>([
  { id: 'rule_001', name: '未佩戴安全帽-高危', type: 'no_helmet', severity: 'critical', conditions: { min_confidence: 90, max_response_time: 30, repeat_count: 1 }, actions: { email_notification: true, sms_notification: true, sound_alarm: true, auto_record: true }, enabled: true },
  { id: 'rule_002', name: '佩戴不规范-中危', type: 'improper_helmet', severity: 'medium', conditions: { min_confidence: 80, max_response_time: 60, repeat_count: 2 }, actions: { email_notification: true, sms_notification: false, sound_alarm: false, auto_record: true }, enabled: true }
])

// 选项数据
const imageQualityOptions = [{ label: '低质量', value: 'low' }, { label: '中等质量', value: 'medium' }, { label: '高质量', value: 'high' }]
const alertTypeOptions = [{ label: '未佩戴安全帽', value: 'no_helmet' }, { label: '佩戴不规范', value: 'improper_helmet' }, { label: '多项违规', value: 'multiple_violations' }]
const severityOptions = [{ label: '低', value: 'low' }, { label: '中', value: 'medium' }, { label: '高', value: 'high' }, { label: '严重', value: 'critical' }]

// 新增规则表单
const newRuleForm = reactive<Partial<AlertRule>>({
  name: '', type: 'no_helmet', severity: 'medium', conditions: { min_confidence: 80, max_response_time: 60, repeat_count: 1 }, actions: { email_notification: true, sms_notification: false, sound_alarm: false, auto_record: true }, enabled: true
})
const showAddRuleDialog = ref(false)

// 方法
const getSeverityStyle = (severity: string) => ({
  low: { text: '低', class: 'text-blue-300 border-blue-400/50 bg-blue-500/20' },
  medium: { text: '中', class: 'text-yellow-300 border-yellow-400/50 bg-yellow-500/20' },
  high: { text: '高', class: 'text-orange-300 border-orange-400/50 bg-orange-500/20' },
  critical: { text: '严重', class: 'text-red-300 border-red-400/50 bg-red-500/20' },
}[severity] || { text: '未知', class: 'text-slate-300 border-slate-400/50 bg-slate-500/20' });

const getTypeText = (type: string) => ({ no_helmet: '未佩戴安全帽', improper_helmet: '佩戴不规范', multiple_violations: '多项违规' }[type] || '未知');

const handleSaveSettings = async () => {
  saving.value = true
  try {
    await new Promise(resolve => setTimeout(resolve, 1500))
    ElMessage({ message: '设置保存成功', type: 'success', customClass: 'glass-message' });
  } catch (error) {
    ElMessage({ message: '设置保存失败', type: 'error', customClass: 'glass-message' });
  } finally {
    saving.value = false
  }
}

const handleResetSettings = async () => {
  try {
    await ElMessageBox.confirm('确定要重置当前设置吗？此操作不可恢复。', '重置设置', { confirmButtonText: '确定', cancelButtonText: '取消', type: 'warning' })
    ElMessage({ message: '设置已重置', type: 'success', customClass: 'glass-message' });
  } catch (error) { /* User cancelled */ }
}

const handleAddRule = () => { showAddRuleDialog.value = true }

const handleSaveRule = () => {
  if (!newRuleForm.name) {
    ElMessage({ message: '请输入规则名称', type: 'warning', customClass: 'glass-message' });
    return
  }
  const rule: AlertRule = { id: `rule_${Date.now()}`, name: newRuleForm.name!, type: newRuleForm.type!, severity: newRuleForm.severity!, conditions: { ...newRuleForm.conditions! }, actions: { ...newRuleForm.actions! }, enabled: newRuleForm.enabled! }
  alertRules.value.push(rule)
  showAddRuleDialog.value = false
  Object.assign(newRuleForm, { name: '', type: 'no_helmet', severity: 'medium', conditions: { min_confidence: 80, max_response_time: 60, repeat_count: 1 }, actions: { email_notification: true, sms_notification: false, sound_alarm: false, auto_record: true }, enabled: true })
  ElMessage({ message: '规则添加成功', type: 'success', customClass: 'glass-message' });
}

const handleDeleteRule = async (rule: AlertRule) => {
  try {
    await ElMessageBox.confirm(`确定要删除规则"${rule.name}"吗？`, '删除规则', { confirmButtonText: '确定', cancelButtonText: '取消', type: 'warning' })
    const index = alertRules.value.findIndex(r => r.id === rule.id)
    if (index > -1) {
      alertRules.value.splice(index, 1)
      ElMessage({ message: '规则删除成功', type: 'success', customClass: 'glass-message' });
    }
  } catch (error) { /* User cancelled */ }
}

const handleToggleRule = (rule: AlertRule) => {
  rule.enabled = !rule.enabled
  ElMessage({ message: `规则已${rule.enabled ? '启用' : '禁用'}`, type: 'success', customClass: 'glass-message' });
}
</script>

<template>
  <Motion :initial="pageVariants.initial" :animate="pageVariants.animate" :transition="pageVariants.transition as any">
    <div class="space-y-6">
      <!-- Page Header -->
      <div class="page-header">
        <div>
          <h2 class="page-title">系统设置</h2>
          <p class="page-subtitle">安全帽检测系统参数配置与管理</p>
        </div>
        <div class="flex items-center gap-2">
          <button class="action-btn" @click="handleResetSettings"><el-icon><Refresh /></el-icon>重置设置</button>
          <button class="action-btn primary" @click="handleSaveSettings" :loading="saving"><el-icon><Check /></el-icon>保存设置</button>
        </div>
      </div>

      <!-- Settings Card -->
      <div class="glass-card p-0 sm:p-0">
        <el-tabs v-model="activeTab" class="settings-tabs">
          <!-- Detection Settings -->
          <el-tab-pane label="检测设置" name="detection">
            <template #label><span class="tab-label"><el-icon><Camera /></el-icon>检测设置</span></template>
            <div class="p-6">
              <h3 class="section-title">基础检测参数</h3>
              <el-form :model="detectionSettings" label-width="140px" label-position="top" class="settings-form">
                <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-x-6 gap-y-2">
                  <el-form-item label="置信度阈值"><el-slider v-model="detectionSettings.confidence_threshold" :min="50" :max="100" :step="5" show-input /></el-form-item>
                  <el-form-item label="检测间隔(秒)"><el-input-number v-model="detectionSettings.detection_interval" :min="1" :max="60" :step="1" controls-position="right" /></el-form-item>
                  <el-form-item label="图像质量"><el-select v-model="detectionSettings.image_quality" popper-class="glass-select-popper"><el-option v-for="o in imageQualityOptions" :key="o.value" :label="o.label" :value="o.value"/></el-select></el-form-item>
                  <el-form-item label="最大检测距离(米)"><el-input-number v-model="detectionSettings.max_detection_distance" :min="10" :max="200" :step="5" controls-position="right" /></el-form-item>
                  <el-form-item label="工作时间"><div class="flex items-center w-full"><el-time-picker v-model="detectionSettings.working_hours.start" placeholder="开始" format="HH:mm" popper-class="glass-date-popper"/><span class="mx-2 text-white/50">-</span><el-time-picker v-model="detectionSettings.working_hours.end" placeholder="结束" format="HH:mm" popper-class="glass-date-popper"/></div></el-form-item>
                  <el-form-item label="安全帽颜色"><div class="flex flex-wrap gap-2 pt-2"><el-color-picker v-for="(c, i) in detectionSettings.helmet_colors" :key="i" v-model="detectionSettings.helmet_colors[i]" show-alpha popper-class="glass-color-popper" /></div></el-form-item>
                  <el-form-item label="保存检测图像"><el-switch v-model="detectionSettings.save_detection_images" /></el-form-item>
                  <el-form-item label="周末检测"><el-switch v-model="detectionSettings.weekend_detection" /></el-form-item>
                </div>
              </el-form>
            </div>
          </el-tab-pane>

          <!-- Alert Rules -->
          <el-tab-pane label="警告规则" name="alerts">
            <template #label><span class="tab-label"><el-icon><Warning /></el-icon>警告规则</span></template>
            <div class="p-6">
              <div class="flex justify-between items-center mb-6">
                <h3 class="section-title !m-0 !p-0 !border-0">警告规则配置</h3>
                <button class="action-btn primary" @click="handleAddRule"><el-icon><Plus /></el-icon>添加规则</button>
              </div>
              <div class="space-y-4">
                <div v-for="rule in alertRules" :key="rule.id" class="rule-card">
                  <div class="flex justify-between items-start gap-4">
                    <div>
                      <h4 class="rule-name">{{ rule.name }}</h4>
                      <div class="flex items-center gap-2 mt-2"><span class="status-tag" :class="getSeverityStyle(rule.severity).class">{{ getSeverityStyle(rule.severity).text }}</span><span class="text-xs text-white/60">{{ getTypeText(rule.type) }}</span></div>
                    </div>
                    <div class="flex items-center gap-2 flex-shrink-0">
                      <el-switch v-model="rule.enabled" @change="handleToggleRule(rule)" size="small" />
                      <button class="action-btn small" @click="handleDeleteRule(rule)"><el-icon><Delete /></el-icon></button>
                    </div>
                  </div>
                  <div class="rule-details">
                    <p><span>最低置信度:</span><span>{{ rule.conditions.min_confidence }}%</span></p>
                    <p><span>响应时间:</span><span>{{ rule.conditions.max_response_time }}秒</span></p>
                    <p><span>重复次数:</span><span>{{ rule.conditions.repeat_count }}次</span></p>
                  </div>
                  <div class="rule-actions">
                    <span class="text-xs text-white/60">触发动作:</span>
                    <div class="flex flex-wrap gap-2"><el-tag v-if="rule.actions.email_notification" size="small" effect="dark" type="success">邮件</el-tag><el-tag v-if="rule.actions.sms_notification" size="small" effect="dark" type="warning">短信</el-tag><el-tag v-if="rule.actions.sound_alarm" size="small" effect="dark" type="danger">报警</el-tag><el-tag v-if="rule.actions.auto_record" size="small" effect="dark" type="info">录制</el-tag></div>
                  </div>
                </div>
              </div>
            </div>
          </el-tab-pane>
        </el-tabs>
      </div>
    </div>

    <!-- Add Rule Dialog -->
    <el-dialog v-model="showAddRuleDialog" title="添加警告规则" width="600px" class="glass-dialog">
      <el-form :model="newRuleForm" label-width="120px" label-position="top">
        <el-form-item label="规则名称"><el-input v-model="newRuleForm.name" placeholder="请输入规则名称" /></el-form-item>
        <div class="grid grid-cols-2 gap-4">
          <el-form-item label="警告类型"><el-select v-model="newRuleForm.type" popper-class="glass-select-popper"><el-option v-for="t in alertTypeOptions" :key="t.value" :label="t.label" :value="t.value" /></el-select></el-form-item>
          <el-form-item label="严重程度"><el-select v-model="newRuleForm.severity" popper-class="glass-select-popper"><el-option v-for="s in severityOptions" :key="s.value" :label="s.label" :value="s.value" /></el-select></el-form-item>
        </div>
        <el-form-item label="触发条件"><div class="grid grid-cols-3 gap-2 w-full"><el-input-number placeholder="置信度" v-model="newRuleForm.conditions!.min_confidence" :min="50" :max="100" controls-position="right" /><el-input-number placeholder="响应时间" v-model="newRuleForm.conditions!.max_response_time" :min="10" :max="300" controls-position="right" /><el-input-number placeholder="重复次数" v-model="newRuleForm.conditions!.repeat_count" :min="1" :max="10" controls-position="right" /></div></el-form-item>
        <el-form-item label="触发动作"><el-checkbox-group v-model="newRuleForm.actions"><el-checkbox label="email_notification">邮件通知</el-checkbox><el-checkbox label="sms_notification">短信通知</el-checkbox><el-checkbox label="sound_alarm">声音报警</el-checkbox><el-checkbox label="auto_record">自动录制</el-checkbox></el-checkbox-group></el-form-item>
        <el-form-item label="启用规则"><el-switch v-model="newRuleForm.enabled" /></el-form-item>
      </el-form>
      <template #footer><div class="flex justify-end gap-2"><button class="action-btn" @click="showAddRuleDialog = false">取消</button><button class="action-btn primary" @click="handleSaveRule">保存</button></div></template>
    </el-dialog>
  </Motion>
</template>

<style scoped>
.page-header { display: flex; justify-content: space-between; align-items: center; }
.page-title { font-size: 1.75rem; font-weight: 600; color: white; }
.page-subtitle { font-size: 0.875rem; color: rgba(255, 255, 255, 0.7); margin-top: 0.25rem; }

.glass-card { backdrop-filter: blur(20px); background-color: rgba(255, 255, 255, 0.05); border-radius: 1rem; border: 1px solid rgba(255, 255, 255, 0.2); }
.section-title { font-size: 1.125rem; font-weight: 600; color: white; margin-bottom: 1.5rem; padding-bottom: 0.75rem; border-bottom: 1px solid rgba(255, 255, 255, 0.2); }
.tab-label { display: flex; align-items: center; gap: 0.5rem; }

.action-btn { display: inline-flex; align-items: center; gap: 0.5rem; background-color: rgba(255, 255, 255, 0.1); color: rgba(255, 255, 255, 0.9); border: 1px solid rgba(255, 255, 255, 0.2); padding: 0.5rem 1rem; border-radius: 0.5rem; font-size: 0.875rem; cursor: pointer; transition: all 0.2s; }
.action-btn:hover { background-color: rgba(255, 255, 255, 0.2); }
.action-btn.primary { background-image: linear-gradient(to right, rgba(99, 102, 241, 0.6), rgba(139, 92, 246, 0.6)); border-color: rgba(139, 92, 246, 0.8); }
.action-btn.small { padding: 0.25rem 0.5rem; font-size: 0.75rem; }

.rule-card { background-color: rgba(255, 255, 255, 0.05); border: 1px solid rgba(255, 255, 255, 0.1); border-radius: 0.75rem; padding: 1rem; color: white; }
.rule-name { font-weight: 600; }
.status-tag { display: inline-block; padding: 0.2rem 0.6rem; border-radius: 9999px; font-size: 0.75rem; border: 1px solid; }
.rule-details { display: grid; grid-template-columns: repeat(auto-fit, minmax(180px, 1fr)); gap: 0.5rem 1rem; font-size: 0.75rem; margin-top: 1rem; padding-top: 1rem; border-top: 1px solid rgba(255, 255, 255, 0.1); }
.rule-details p { display: flex; justify-content: space-between; }
.rule-details span:first-child { color: rgba(255, 255, 255, 0.6); }
.rule-actions { margin-top: 1rem; display: flex; align-items: center; gap: 0.5rem; }

/* Scoped styles for child components using :deep() */
:deep(.settings-tabs .el-tabs__header) { margin: 0; }
:deep(.settings-tabs .el-tabs__nav) { padding: 0.5rem 1.5rem; }
:deep(.settings-tabs .el-tabs__item) { color: rgba(255, 255, 255, 0.7); }
:deep(.settings-tabs .el-tabs__item.is-active) { color: white; }
:deep(.settings-tabs .el-tabs__active-bar) { background-color: #a78bfa; }
:deep(.settings-tabs .el-tabs__nav-wrap::after) { background-color: rgba(255, 255, 255, 0.2); }

:deep(.settings-form .el-form-item__label) { color: rgba(255, 255, 255, 0.8); }
:deep(.settings-form .el-input__wrapper), 
:deep(.settings-form .el-select__wrapper), 
:deep(.settings-form .el-input-number), 
:deep(.settings-form .el-date-editor) { background-color: rgba(0, 0, 0, 0.2); border: 1px solid rgba(255, 255, 255, 0.2); box-shadow: none; }
:deep(.settings-form .el-input__inner), 
:deep(.settings-form .el-range-input) { color: white; }
:deep(.settings-form .el-slider__bar) { background-color: #8b5cf6; }
:deep(.settings-form .el-slider__button) { border-color: #8b5cf6; }
:deep(.settings-form .el-switch.is-checked .el-switch__core) { background-color: #4f46e5; border-color: #4f46e5;}
:deep(.settings-form .el-tag) { background-color: rgba(255, 255, 255, 0.1); border: 1px solid rgba(255, 255, 255, 0.2); color: white; }
/* FIX: Target input-number buttons */
:deep(.settings-form .el-input-number__decrease),
:deep(.settings-form .el-input-number__increase) { background-color: rgba(0, 0, 0, 0.2); border: 1px solid rgba(255, 255, 255, 0.2); }
:deep(.settings-form .el-input-number__decrease .el-icon),
:deep(.settings-form .el-input-number__increase .el-icon) { color: rgba(255, 255, 255, 0.7); }
:deep(.settings-form .el-input-number__decrease:hover),
:deep(.settings-form .el-input-number__increase:hover) { background-color: rgba(255, 255, 255, 0.1); }

/* Scoped styles for TELEPORTED elements using :deep() */
:deep(.glass-dialog) { background: rgba(30, 30, 45, 0.85) !important; border: 1px solid rgba(255, 255, 255, 0.2) !important; backdrop-filter: blur(12px); border-radius: 1rem; }
:deep(.glass-dialog .el-dialog__title) { color: white; }
:deep(.glass-dialog .el-dialog__headerbtn .el-icon) { color: white; }
:deep(.glass-dialog .el-dialog__body) { color: rgba(255, 255, 255, 0.9); }
:deep(.glass-dialog .el-form-item__label) { color: rgba(255, 255, 255, 0.8) !important; }

:deep(.glass-dialog .el-input__wrapper), 
:deep(.glass-dialog .el-select__wrapper), 
:deep(.glass-dialog .el-input-number), 
:deep(.glass-dialog .el-date-editor), 
:deep(.glass-dialog .el-checkbox__inner) { background-color: rgba(0, 0, 0, 0.2) !important; border: 1px solid rgba(255, 255, 255, 0.2); box-shadow: none !important; color: white; }
:deep(.glass-dialog .el-input__inner), 
:deep(.glass-dialog .el-select__placeholder), 
:deep(.glass-dialog .el-range-input) { color: rgba(255, 255, 255, 0.9) !important; }
:deep(.glass-dialog .el-checkbox__label) { color: rgba(255, 255, 255, 0.9) !important; }
:deep(.glass-dialog .el-checkbox__inner::after) { border-color: white !important; }
:deep(.glass-dialog .el-switch.is-checked .el-switch__core) { background-color: #4f46e5; border-color: #4f46e5;}

:deep(.glass-select-popper) { background: rgba(30, 30, 45, 0.85) !important; border: 1px solid rgba(255, 255, 255, 0.2) !important; backdrop-filter: blur(12px); }
/* FIX: Target inner list for select dropdown */
:deep(.glass-select-popper .el-select-dropdown__wrap) { background-color: transparent !important; }
:deep(.glass-select-popper .el-select-dropdown__list) { background-color: transparent !important; }
:deep(.glass-select-popper .el-select-dropdown__item.hover), 
:deep(.glass-select-popper .el-select-dropdown__item:hover) { background-color: rgba(255, 255, 255, 0.1); }
:deep(.glass-select-popper .el-select-dropdown__item) { color: rgba(255, 255, 255, 0.8); }

:deep(.glass-date-popper) { background: rgba(30, 30, 45, 0.85) !important; border: 1px solid rgba(255, 255, 255, 0.2) !important; backdrop-filter: blur(12px); }
:deep(.glass-date-popper .el-picker-panel__icon-btn), 
:deep(.glass-date-popper .el-date-table th),
:deep(.glass-date-popper .el-time-panel__title) { color: rgba(255, 255, 255, 0.9); }
:deep(.glass-date-popper .el-date-table td .el-date-table-cell:hover),
:deep(.glass-date-popper .el-time-spinner__item:hover) { background-color: rgba(255, 255, 255, 0.1) !important; }
:deep(.glass-date-popper .el-date-table td.today .el-date-table-cell__text) { color: #67e8f9; font-weight: bold; }
:deep(.glass-date-popper .el-date-table td.current .el-date-table-cell__text) { background-color: #4f46e5; border-radius: 50%; }
:deep(.glass-date-popper .el-time-spinner__item.is-active) { color: #a78bfa; font-weight: bold; }

:deep(.glass-color-popper) { background: rgba(30, 30, 45, 0.85) !important; border: 1px solid rgba(255, 255, 255, 0.2) !important; backdrop-filter: blur(12px); }
:deep(.glass-color-popper .el-color-svpanel) { background-color: transparent !important;}
:deep(.glass-color-popper .el-color-dropdown__btns .el-button--text) { color: #a78bfa; }

:deep(.glass-message) { background-color: rgba(30, 30, 45, 0.8) !important; border: 1px solid rgba(255, 255, 255, 0.2) !important; backdrop-filter: blur(10px); color: white !important; }
</style>
