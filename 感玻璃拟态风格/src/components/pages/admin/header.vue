<script setup lang="ts">
import { ref, computed } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import { ElMessageBox, ElMessage } from 'element-plus'
import { useUserStore } from '@/stores/auth/user'
import { storeToRefs } from 'pinia'
import { adminMenuItems } from '@/configs'
import type { AdminHeader as AdminHeaderType } from '@/types/factory' // Renamed to avoid conflict
import { useCommonStore } from '@/stores/common'

interface Props {
  isCollapse: boolean
}

defineProps<Props>()

const emit = defineEmits<{
  toggleSidebar: []
}>()

const router = useRouter()
const route = useRoute()

const store = useUserStore()
const { GlobalKeyWord } = storeToRefs(useCommonStore())
const { userInfo, getUserAvatar } = storeToRefs(store)

const avatar = getUserAvatar

// Dynamic breadcrumb logic (remains unchanged)
const breadcrumbs = computed(() => {
  const currentPath = route.path
  const breadcrumbItems = [{ label: '首页', path: '/admin/dashboard' }]

  const buildBreadcrumb = (items: AdminHeaderType[], targetPath: string, parentPath: AdminHeaderType[] = []): AdminHeaderType[] => {
    for (const item of items) {
      const currentBreadcrumb = [...parentPath, item]
      if (item.path === targetPath) {
        return currentBreadcrumb
      }
      if (item.children) {
        const result = buildBreadcrumb(item.children, targetPath, currentBreadcrumb)
        if (result.length > 0) {
          return result
        }
      }
    }
    return []
  }

  if (currentPath !== '/admin/dashboard') {
    const menuPath = buildBreadcrumb(adminMenuItems, currentPath)
    // The first item is the parent category, which is already represented by '首页' in our context.
    // So we only take the last item, which is the current page.
    if (menuPath.length > 0) {
      const currentPage = menuPath[menuPath.length - 1]
      breadcrumbItems.push({
        label: currentPage.title,
        path: currentPage.path
      })
    }
  }

  return breadcrumbItems
})


// User dropdown menu items (remains unchanged)
const userMenuItems = ref([
  { command: 'profile', label: '个人中心', icon: 'User', event: () => { ElMessage.info('个人中心功能开发中') } },
  { command: 'settings', label: '设置', icon: 'Setting', event: () => { ElMessage.info('设置功能开发中') } },
  { command: 'logout', label: '退出登录', icon: 'SwitchButton', divided: true, event: () => { handleLogout() } }
])

// Event handlers (remains unchanged)
function handleToggleSidebar() {
  emit('toggleSidebar')
}

function handleCommand(command: string) {
  const menuItem = userMenuItems.value.find(item => item.command === command)
  if (menuItem && menuItem.event) {
    menuItem.event()
  }
}

function handleLogout() {
  ElMessageBox.confirm('确定要退出登录吗？', '提示', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning',
  }).then(() => {
    store.setUserInfo(null) // 清除用户信息实现登出功能
    ElMessage.success('退出成功')
    router.push('/auth/login')
  }).catch(() => { /* User cancelled */ })
}

function toggleFullscreen() {
  if (!document.fullscreenElement) {
    document.documentElement.requestFullscreen()
  } else {
    document.exitFullscreen()
  }
}
</script>

<template>
  <div class="flex items-center justify-between px-6 h-full">
    <!-- Left Area -->
    <div class="flex items-center">
      <!-- Menu toggle button -->
      <button @click="handleToggleSidebar" class="header-btn mr-4">
        <el-icon size="18">
          <Fold v-if="!isCollapse" />
          <Expand v-else />
        </el-icon>
      </button>

      <!-- Breadcrumb navigation -->
      <el-breadcrumb separator-icon="ArrowRight" class="text-sm">
        <el-breadcrumb-item v-for="(item, index) in breadcrumbs" :key="index"
          :to="index < breadcrumbs.length - 1 ? { path: item.path } : undefined">
          {{ item.label }}
        </el-breadcrumb-item>
      </el-breadcrumb>
    </div>

    <!-- Right Area -->
    <div class="flex items-center space-x-2">
      <!-- Search Input -->
      <el-input placeholder="搜索..." size="small" v-model="GlobalKeyWord" style="width: 200px" prefix-icon="Search"
        clearable class="glass-input" />

      <!-- Fullscreen Button -->
      <el-tooltip content="全屏" placement="bottom" effect="dark">
        <button @click="toggleFullscreen" class="header-btn">
          <el-icon size="18">
            <FullScreen />
          </el-icon>
        </button>
      </el-tooltip>

      <!-- Notification Button -->
      <el-tooltip content="通知" placement="bottom" effect="dark">
        <el-badge :value="3" class="item" :offset="[-4, 8]">
          <button class="header-btn">
            <el-icon size="18">
              <Bell />
            </el-icon>
          </button>
        </el-badge>
      </el-tooltip>

      <!-- User Info Dropdown -->
      <el-dropdown @command="handleCommand" trigger="click" popper-class="glass-dropdown">
        <div
          class="flex items-center cursor-pointer hover:bg-white/10 px-2 py-1 rounded-lg transition-colors duration-200">
          <el-avatar :size="32" :src="avatar" class="mr-3" />
          <div class="text-sm text-left">
            <div class="font-medium text-white/90">{{ userInfo?.username || '用户' }}</div>
            <div class="text-white/60 text-xs">{{ userInfo?.role || '管理员' }}</div>
          </div>
        </div>

        <template #dropdown>
          <el-dropdown-menu>
            <el-dropdown-item v-for="item in userMenuItems" :key="item.command" :command="item.command"
              :divided="item.divided">
              <el-icon class="mr-2">
                <component :is="item.icon" />
              </el-icon>
              {{ item.label }}
            </el-dropdown-item>
          </el-dropdown-menu>
        </template>
      </el-dropdown>
    </div>
  </div>
</template>

<style scoped>
/* Custom class for header action buttons */
.header-btn {
  width: 36px;
  height: 36px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  color: rgba(255, 255, 255, 0.8);
  background-color: transparent;
  transition: background-color 0.2s ease-in-out;
  border: none;
  cursor: pointer;
}

.header-btn:hover {
  background-color: rgba(255, 255, 255, 0.1);
  color: #ffffff;
}

.item {
  display: flex;
  align-items: center;
}

/* Breadcrumb styling */
:deep(.el-breadcrumb__item .el-breadcrumb__inner),
:deep(.el-breadcrumb__item .el-breadcrumb__separator) {
  color: rgba(255, 255, 255, 0.6) !important;
  font-weight: normal;
}

:deep(.el-breadcrumb__item:last-child .el-breadcrumb__inner) {
  color: #ffffff !important;
  font-weight: 500;
}

:deep(.el-breadcrumb__item .el-breadcrumb__inner:hover) {
  color: #ffffff !important;
}

/* Glass Input styling */
:deep(.glass-input .el-input__wrapper) {
  background-color: rgba(0, 0, 0, 0.2) !important;
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  box-shadow: none;
  color: white;
}

:deep(.glass-input .el-input__inner) {
  color: rgba(255, 255, 255, 0.9) !important;
}

:deep(.glass-input .el-input__wrapper.is-focus) {
  border-color: rgba(99, 102, 241, 0.7);
}

:deep(.glass-input .el-input__inner::placeholder) {
  color: rgba(255, 255, 255, 0.4);
}

:deep(.glass-input .el-input__prefix .el-icon),
:deep(.glass-input .el-input__suffix .el-icon) {
  color: rgba(255, 255, 255, 0.6);
}

/* Notification Badge styling */
:deep(.el-badge__content) {
  background-color: #ef4444;
  /* red-500 */
  border: none;
}
</style>

<!-- Global style for the dropdown popper -->
<style>
.glass-dropdown .el-popper__arrow::before {
  background: rgba(30, 30, 45, 0.8) !important;
  border-color: rgba(255, 255, 255, 0.2) !important;
}

.glass-dropdown.el-popper {
  background: rgba(30, 30, 45, 0.8) !important;
  border: 1px solid rgba(255, 255, 255, 0.2) !important;
  border-radius: 12px;
  backdrop-filter: blur(12px);
  box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.37);
}

.glass-dropdown .el-dropdown-menu {
  background: transparent !important;
  padding: 6px;
}

.glass-dropdown .el-dropdown-menu__item {
  color: rgba(255, 255, 255, 0.8);
  border-radius: 8px;
}

.glass-dropdown .el-dropdown-menu__item:hover {
  background-color: rgba(255, 255, 255, 0.1) !important;
  color: #ffffff;
}

.glass-dropdown .el-dropdown-menu__item--divided {
  border-top: 1px solid rgba(255, 255, 255, 0.2);
  margin-top: 4px;
  padding-top: 4px;
}
</style>
