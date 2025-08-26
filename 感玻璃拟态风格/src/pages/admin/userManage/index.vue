<script setup lang="ts">
import { ref, reactive, computed, onMounted } from 'vue'
import { Motion } from 'motion-v'
import { ElMessage, ElMessageBox } from 'element-plus'
import { cloneDeep } from 'lodash-es'
import type { User, UserQueryParams } from '@/types/components/admin'
import Search from '@/components/icon/search.vue'
import {
  getUsersAPI,
  addUserAPI,
  userDetailAPI,
  editUserAPI,
  deleteUserAPI
} from '@/api/admin/users'
import { useUserStore } from '@/stores/auth/user'

// Animation configuration
const pageVariants = {
  initial: { opacity: 0, y: 30 },
  animate: { opacity: 1, y: 0 },
  transition: { duration: 0.6, ease: 'easeOut' }
}

// Store
const userStore = useUserStore()

// Reactive data
const loading = ref(false)
const list = ref<User[]>([])
const total = ref(0)
const isAdd = ref(false)
const editableData = reactive<Record<string | number, User>>({})

// Query parameters
const params = reactive<UserQueryParams>({
  query: '',
  page: 1,
  page_size: 10
})

// Computed property
const userId = computed(() => userStore.userInfo?.user_id)

// Get user list
const getList = async () => {
  try {
    loading.value = true
    const res = await getUsersAPI(params)
    list.value = res.data.users
    total.value = res.data.total
  } catch (error) {
    console.error('Failed to get data:', error)
    ElMessage({ message: '获取用户列表失败', type: 'error', customClass: 'glass-message' })
  } finally {
    loading.value = false
  }
}

// Search
const search = () => {
  params.page = 1
  getList()
}

// Reset parameters
const resetParams = () => {
  params.page = 1
  params.query = ''
  getList()
}

// View user details
const detail = async (id: number) => {
  try {
    const res = await userDetailAPI(id)
    console.log('User details:', res)
    ElMessage({ message: '查看用户详情成功', type: 'success', customClass: 'glass-message' })
  } catch (error) {
    console.error('Failed to get user details:', error)
    ElMessage({ message: '获取用户详情失败', type: 'error', customClass: 'glass-message' })
  }
}

// Edit user
const edit = (id: number) => {
  const user = list.value.find(item => item.id === id)
  if (user) {
    editableData[id] = cloneDeep(user)
  }
}

// Add user
const handleAdd = () => {
  if (isAdd.value) return
  const newUser: User = { id: null, username: '', email: '', is_active: true, is_staff: false, is_superuser: false }
  editableData['new'] = cloneDeep(newUser) // Use a temporary key
  isAdd.value = true
  list.value.unshift(newUser)
}

// Cancel edit
const clean = (id: number | null) => {
  if (isAdd.value && id === null) {
    list.value.shift()
    isAdd.value = false
    delete editableData['new']
  } else {
    delete editableData[id as any]
  }
}

// Form validation
const validateData = (data: User) => {
  if (!data.username?.trim()) return '用户名不能为空'
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  if (!emailRegex.test(data.email)) return '邮箱格式不正确'
  return null
}

// Save user
const save = async (key: number | null) => {
  try {
    const dataToSave = editableData[key === null ? 'new' : key]
    const errorMsg = validateData(dataToSave)
    if (errorMsg) {
      ElMessage({ message: errorMsg, type: 'error', customClass: 'glass-message' })
      return
    }

    const res = isAdd.value ? await addUserAPI(dataToSave) : await editUserAPI(dataToSave)
    ElMessage({ message: res.message || '操作成功', type: 'success', customClass: 'glass-message' })

    delete editableData[key === null ? 'new' : key]
    isAdd.value = false
    await getList()
  } catch (error: any) {
    console.error('Save failed:', error)
    ElMessage({ message: error.message || '操作失败', type: 'error', customClass: 'glass-message' })
  }
}

// Delete user
const deleteRecord = async (id: number) => {
  try {
    await deleteUserAPI(id)
    ElMessage({ message: '删除成功', type: 'success', customClass: 'glass-message' })
    await getList()
  } catch (error) {
    console.error('Delete failed:', error)
    ElMessage({ message: '删除失败', type: 'error', customClass: 'glass-message' })
  }
}

// Pagination handlers
const handleSizeChange = (size: number) => {
  params.page_size = size
  params.page = 1
  getList()
}
const handleCurrentChange = (page: number) => {
  params.page = page
  getList()
}

// Component mounted
onMounted(getList)
</script>

<template>
  <Motion :initial="pageVariants.initial" :animate="pageVariants.animate" :transition="pageVariants.transition as any">
    <div class="user-manage-page">
      <!-- Search and Action Area -->
      <Motion :initial="{ opacity: 0, y: -20 }" :animate="{ opacity: 1, y: 0 }"
        :transition="{ duration: 0.5, delay: 0.1 }">
        <div class="flex flex-wrap items-center gap-4 mb-6">
          <el-input v-model="params.query" placeholder="请输入邮箱或用户名" clearable @keyup.enter="search"
            class="glass-input w-64" />
          <div class="flex items-center gap-2">
            <button class="action-btn" @click="resetParams"><el-icon>
                <Refresh />
              </el-icon>重置</button>
            <button class="action-btn primary" @click="search"><el-icon>
                <Search />
              </el-icon>查询</button>
            <button class="action-btn success" @click="handleAdd"><el-icon>
                <Plus />
              </el-icon>添加</button>
          </div>
        </div>
      </Motion>

      <!-- Table Area -->
      <Motion :initial="{ opacity: 0, y: 20 }" :animate="{ opacity: 1, y: 0 }"
        :transition="{ duration: 0.6, delay: 0.2 }">
        <el-table :data="list" v-loading="loading" stripe class="user-table">
          <el-table-column label="序号" width="80" align="center">
            <template #default="{ $index }">
              {{ (params.page - 1) * params.page_size + $index + 1 }}
            </template>
          </el-table-column>

          <el-table-column label="用户名" prop="username" min-width="120">
            <template #default="{ row }">
              <el-input v-if="editableData[row.id ?? 'new']" v-model="editableData[row.id ?? 'new'].username"
                size="small" class="glass-input" />
              <el-link v-else type="primary" @click="detail(row.id)">{{ row.username }}</el-link>
            </template>
          </el-table-column>

          <el-table-column label="邮箱" prop="email" min-width="180">
            <template #default="{ row }">
              <el-input v-if="editableData[row.id ?? 'new']" v-model="editableData[row.id ?? 'new'].email" size="small"
                class="glass-input" />
              <span v-else>{{ row.email }}</span>
            </template>
          </el-table-column>

          <el-table-column label="状态" prop="is_active" width="120" align="center">
            <template #default="{ row }">
              <el-switch v-if="editableData[row.id ?? 'new']" v-model="editableData[row.id ?? 'new'].is_active"
                size="small" />
              <el-tag v-else :type="row.is_active ? 'success' : 'danger'" effect="dark" round>{{ row.is_active ? '启用' :
                '停用' }}</el-tag>
            </template>
          </el-table-column>

          <el-table-column label="工作人员" prop="is_staff" width="120" align="center">
            <template #default="{ row }">
              <el-switch v-if="editableData[row.id ?? 'new']" v-model="editableData[row.id ?? 'new'].is_staff"
                size="small" />
              <el-tag v-else :type="row.is_staff ? 'success' : 'info'" effect="dark" round>{{ row.is_staff ? '是' : '否'
                }}</el-tag>
            </template>
          </el-table-column>

          <el-table-column label="超级管理员" prop="is_superuser" width="120" align="center">
            <template #default="{ row }">
              <el-switch v-if="editableData[row.id ?? 'new']" v-model="editableData[row.id ?? 'new'].is_superuser"
                size="small" />
              <el-tag v-else :type="row.is_superuser ? 'warning' : 'info'" effect="dark" round>{{ row.is_superuser ? '是'
                : '否' }}</el-tag>
            </template>
          </el-table-column>

          <el-table-column label="操作" width="200" align="center" fixed="right">
            <template #default="{ row }">
              <div class="flex items-center justify-center gap-2">
                <template v-if="editableData[row.id ?? 'new']">
                  <button class="action-btn primary small" @click="save(row.id)"><el-icon>
                      <Check />
                    </el-icon>保存</button>
                  <button class="action-btn small" @click="clean(row.id)"><el-icon>
                      <Close />
                    </el-icon>取消</button>
                </template>
                <template v-else>
                  <button class="action-btn primary small" :disabled="userId === row.id" @click="edit(row.id)"><el-icon>
                      <Edit />
                    </el-icon>编辑</button>
                  <el-popconfirm v-if="userId !== row.id" title="确认删除吗?" confirm-button-text="确认"
                    cancel-button-text="取消" @confirm="deleteRecord(row.id)">
                    <template #reference>
                      <button class="action-btn danger small"><el-icon>
                          <Delete />
                        </el-icon>删除</button>
                    </template>
                  </el-popconfirm>
                </template>
              </div>
            </template>
          </el-table-column>
        </el-table>
      </Motion>

      <!-- Pagination Area -->
      <Motion :initial="{ opacity: 0, y: 20 }" :animate="{ opacity: 1, y: 0 }"
        :transition="{ duration: 0.5, delay: 0.3 }" class="mt-6 flex justify-end">
        <el-pagination v-model:current-page="params.page" v-model:page-size="params.page_size" :total="total"
          :page-sizes="[10, 20, 50, 100]" layout="total, sizes, prev, pager, next, jumper"
          @size-change="handleSizeChange" @current-change="handleCurrentChange" />
      </Motion>
    </div>
  </Motion>
</template>

<style scoped>
.user-manage-page {
  backdrop-filter: blur(20px);
  background-color: rgba(255, 255, 255, 0.05);
  border-radius: 1rem;
  /* 16px */
  border: 1px solid rgba(255, 255, 255, 0.2);
  padding: 1.5rem;
  /* 24px */
}

/* Custom Action Buttons */
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
  transform: translateY(-2px);
}

.action-btn.primary {
  background-image: linear-gradient(to right, rgba(99, 102, 241, 0.6), rgba(139, 92, 246, 0.6));
  border-color: rgba(139, 92, 246, 0.8);
}

.action-btn.primary:hover {
  background-image: linear-gradient(to right, #6366f1, #8b5cf6);
}

.action-btn.success {
  background-image: linear-gradient(to right, rgba(34, 197, 94, 0.6), rgba(22, 163, 74, 0.6));
  border-color: rgba(34, 197, 94, 0.8);
}

.action-btn.success:hover {
  background-image: linear-gradient(to right, #22c55e, #16a34a);
}

.action-btn.danger {
  background-image: linear-gradient(to right, rgba(239, 68, 68, 0.6), rgba(220, 38, 38, 0.6));
  border-color: rgba(239, 68, 68, 0.8);
}

.action-btn.small {
  padding: 0.25rem 0.75rem;
  font-size: 0.75rem;
}

/* Table styling */
.user-table {
  background-color: transparent;
  border-radius: 0.75rem;
  overflow: hidden;
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

:deep(.el-table--striped .el-table__body tr.el-table__row--striped td.el-table__cell) {
  background-color: rgba(255, 255, 255, 0.03);
}

:deep(.el-table .el-link) {
  color: #67e8f9;
  /* cyan-300 */
}

:deep(.el-table .el-link:hover) {
  color: #a5f3fc;
  /* cyan-200 */
}

:deep(.el-table .el-tag) {
  background-color: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

:deep(.el-table .el-tag--success) {
  color: #6ee7b7;
  border-color: #34d399;
}

:deep(.el-table .el-tag--danger) {
  color: #f87171;
  border-color: #ef4444;
}

:deep(.el-table .el-tag--warning) {
  color: #fcd34d;
  border-color: #f59e0b;
}

:deep(.el-table .el-tag--info) {
  color: #9ca3af;
  border-color: #6b7280;
}

:deep(.el-table .el-switch.is-checked .el-switch__core) {
  background-color: #4f46e5;
  border-color: #4f46e5;
}

/* Glass Input styling */
:deep(.glass-input .el-input__wrapper) {
  background-color: rgba(0, 0, 0, 0.2) !important;
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  box-shadow: none;
}

:deep(.glass-input .el-input__inner) {
  color: rgba(255, 255, 255, 0.9) !important;
}

/* Pagination styling */
:deep(.el-pagination) {
  color: rgba(255, 255, 255, 0.8);
}

:deep(.el-pagination button),
:deep(.el-pager li) {
  background: transparent;
  color: rgba(255, 255, 255, 0.8);
}

:deep(.el-pager li.is-active) {
  background-color: #4f46e5;
  color: white;
}

:deep(.el-pagination .el-input__inner) {
  color: white;
}

:deep(.el-pagination .el-select .el-input__wrapper) {
  background-color: rgba(0, 0, 0, 0.2);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.el-popover.el-popper {
  background: rgba(30, 30, 45, 0.8) !important;
  border: 1px solid rgba(255, 255, 255, 0.2) !important;
  border-radius: 12px;
  backdrop-filter: blur(12px);
  color: white !important;
}

.el-popover.el-popper .el-popper__arrow::before {
  background: rgba(30, 30, 45, 0.8) !important;
  border-color: rgba(255, 255, 255, 0.2) !important;
}

.el-popconfirm__main-text {
  color: white !important;
}
</style>
