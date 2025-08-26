<script setup lang="ts">
import serverConfig, { adminMenuItems } from '@/configs';
import { useUserStore } from '@/stores/auth/user';
import type { AdminHeader } from '@/types/factory';
import { storeToRefs } from 'pinia';
import { ref, computed, watch, onMounted } from 'vue'
import { useRouter, useRoute } from 'vue-router'
const store = useUserStore()

const { userInfo } = storeToRefs(store)

interface Props {
  isCollapse: boolean
}

defineProps<Props>()

const router = useRouter()
const route = useRoute()

// 菜单数据
const menuItems = ref<AdminHeader[]>(adminMenuItems)

// 当前激活的菜单项
const activeMenu = computed(() => route.path)

// 默认展开的父菜单
const defaultOpeneds = computed(() => {
  const matched = menuItems.value.find(item =>
    item.children?.some(child => child.path === route.path)
  )
  return matched ? [matched.id] : []
})

// 处理菜单点击
function handleMenuClick(path: string) {
  if (path && path !== route.path) {
    router.push(path)
  }
}

// 监听路由变化更新
watch(() => route.path, () => {
  // computed property activeMenu will update automatically
})

onMounted(() => {
  // Login check
  if (!userInfo.value) {
    router.push('/auth/login')
  }
})
</script>

<template>
  <div class="admin-navbar h-full">
    <!-- Logo -->
    <div class="logo-container flex items-center justify-center h-16 border-b border-white/20">
      <div v-if="!isCollapse" class="flex items-center">
        <!-- Gradient text for the title -->
        <span
          class="bg-gradient-to-r from-white via-blue-100 to-purple-200 bg-clip-text text-transparent text-lg font-semibold">{{
            serverConfig.VITE_APP_TITLE }}</span>
      </div>

      <div v-else class="flex items-center">
        <div class="w-8 h-8 flex items-center justify-center">
          <el-icon size="23" class="text-white">
            <Monitor />
          </el-icon>
        </div>
      </div>
    </div>

    <!-- Menu -->
    <div class="menu-container flex-1 overflow-y-auto">
      <!-- Updated el-menu properties for the new theme -->
      <el-menu :default-active="activeMenu" :default-openeds="defaultOpeneds" :collapse="isCollapse"
        :unique-opened="true" background-color="transparent" text-color="rgba(255, 255, 255, 0.7)"
        active-text-color="#ffffff" class="border-none">
        <template v-for="item in menuItems" :key="item.id">
          <template v-if="!item.hide">
            <!-- Without children -->
            <el-menu-item v-if="!item.children || item.children.length === 0" :index="item.path"
              @click="handleMenuClick(item.path)">
              <el-icon>
                <component :is="item.icon" />
              </el-icon>
              <template #title>{{ item.title }}</template>
            </el-menu-item>

            <!-- With children -->
            <el-sub-menu v-else :index="item.id">
              <template #title>
                <el-icon>
                  <component :is="item.icon" />
                </el-icon>
                <span>{{ item.title }}</span>
              </template>

              <el-menu-item v-for="child in item.children" :key="child.id" :index="child.path" v-show="!child.hide"
                @click="handleMenuClick(child.path)">
                <el-icon v-if="child.icon" style="margin-right: 0;">
                  <component :is="child.icon" />
                </el-icon>
                <span>{{ child.title }}</span>
              </el-menu-item>
            </el-sub-menu>
          </template>
        </template>
      </el-menu>
    </div>

    <!-- Footer -->
    <div v-if="!isCollapse" class="footer-info p-4 border-t border-white/20">
      <div class="text-center text-white/60 text-xs">
        <div class="font-medium text-white/80">{{ serverConfig.VITE_APP_TITLE }} {{ serverConfig.VITE_APP_VERSION }}
        </div>
        <div class="mt-1 text-white/50">© 2024 HZ Tech</div>
      </div>
    </div>
  </div>
</template>


<style scoped>
.admin-navbar {
  /* The navbar itself is transparent, its container provides the glass effect */
  background-color: transparent;
  display: flex;
  flex-direction: column;
}

.menu-container {
  flex: 1;
  overflow-y: auto;
  padding: 8px 0;
}

/* Themed scrollbar for the menu */
.menu-container::-webkit-scrollbar {
  width: 4px;
}

.menu-container::-webkit-scrollbar-track {
  background: transparent;
}

.menu-container::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.2);
  border-radius: 2px;
}

.menu-container::-webkit-scrollbar-thumb:hover {
  background: rgba(255, 255, 255, 0.4);
}

:deep(.el-menu) {
  border-right: none;
  padding: 0 8px;
  /* Slightly adjusted padding */
}

/* Styling for both top-level items and submenu titles */
:deep(.el-menu-item),
:deep(.el-sub-menu .el-sub-menu__title) {
  height: 44px;
  line-height: 44px;
  margin: 4px 0;
  /* A bit more spacing */
  border-radius: 8px;
  transition: all 0.2s ease-in-out;
  color: rgba(255, 255, 255, 0.7);
}

/* Hover effect for all menu items */
:deep(.el-menu-item:hover),
:deep(.el-sub-menu__title:hover) {
  background-color: rgba(255, 255, 255, 0.1) !important;
  color: #ffffff !important;
}

/* Active menu item styling */
:deep(.el-menu-item.is-active) {
  background: linear-gradient(135deg, rgba(99, 102, 241, 0.5) 0%, rgba(139, 92, 246, 0.5) 100%) !important;
  color: #ffffff !important;
  box-shadow: 0 4px 12px rgba(99, 102, 241, 0.2);
  font-weight: 500;
}

/* Nested submenu items */
:deep(.el-sub-menu .el-menu--inline) {
  background-color: rgba(0, 0, 0, 0.2) !important;
  /* Darker nested menu */
  border-radius: 8px;
  padding: 4px;
}

:deep(.el-sub-menu .el-menu-item) {
  margin: 2px 0;
  height: 38px !important;
  line-height: 38px !important;
  color: rgba(255, 255, 255, 0.6);
}

:deep(.el-sub-menu .el-menu-item:hover) {
  color: #ffffff !important;
  background-color: rgba(255, 255, 255, 0.08) !important;
}

:deep(.el-sub-menu .el-menu-item.is-active) {
  color: #ffffff !important;
  background: linear-gradient(135deg, rgba(99, 102, 241, 0.4) 0%, rgba(139, 92, 246, 0.4) 100%) !important;
  font-weight: normal;
  box-shadow: none;
}

:deep(.el-menu-item .el-icon),
:deep(.el-sub-menu__title .el-icon) {
  margin-right: 8px;
  font-size: 16px;
  color: inherit;
  /* Icon should inherit color from parent */
}

/* Active item icon should be white */
:deep(.el-menu-item.is-active .el-icon) {
  color: #ffffff !important;
}

.footer-info {
  background: transparent;
  /* Footer is part of the glass pane */
}
</style>
