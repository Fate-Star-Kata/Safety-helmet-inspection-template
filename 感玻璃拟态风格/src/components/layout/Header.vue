<script setup lang="ts">
import { provide, ref, computed } from 'vue'
import { useI18n } from 'vue-i18n'
import router from '@/router'
import LanguageSwitch from './btns/LanguageSwitch.vue'
// import Search from './btns/search.vue' // Assuming Search component exists
import ThemeSwitch from './btns/ThemeSwitch.vue'
import MenuItems from './MenuItems.vue'
import serverConfig, { defaultMenuItems } from '@/configs'
import { useUserStore } from '@/stores/auth/user'
import { storeToRefs } from "pinia"

const { t } = useI18n()
const expandedMenus = ref<Set<string>>(new Set())
const icon = serverConfig.VITE_APP_LOGO

const store = useUserStore()
const { userInfo, getUserAvatar } = storeToRefs(store)
const avatar = getUserAvatar

// 用户头像下拉菜单项
const userMenuItems = ref([
  {
    title: '个人资料',
    class: 'justify-between',
    badge: '新',
    event: () => {
      console.log('跳转到个人资料页面')
      router.push('/userInfo')
    }
  },
  {
    title: '设置',
    class: '',
    hide: true,
    event: () => {
      console.log('跳转到设置页面')
      // router.push('/settings')
    }
  },
  {
    title: '退出登录',
    class: '',
    event: () => {
      console.log('执行退出登录')
      // FIX: Call the correct store method to log out
      store.setUserInfo(null) 
      router.push("/auth/login")
    }
  }
])

// 从configs获取菜单项并应用国际化
const menuItems = computed(() => {
  const applyI18n = (items: any[]): any[] => {
    return items.map(item => ({
      ...item,
      label: t(item.label),
      children: item.children ? applyI18n(item.children) : undefined
    }))
  }
  return applyI18n(defaultMenuItems)
})

function toggleSubmenu(menuId: string) {
  if (expandedMenus.value.has(menuId)) {
    expandedMenus.value.delete(menuId)
  }
  else {
    expandedMenus.value.add(menuId)
  }
}

function isMenuExpanded(menuId: string) {
  return expandedMenus.value.has(menuId)
}

// 提供菜单状态管理函数给子组件
provide('isMenuExpanded', isMenuExpanded)
provide('toggleSubmenu', toggleSubmenu)
</script>

<template>
  <div class="custom-navbar">
    <div class="navbar-start">
      <div class="dropdown">
        <div tabindex="0" role="button" class="action-icon lg:hidden">
          <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h8m-8 6h16" />
          </svg>
        </div>
        <ul tabindex="0" class="menu-dropdown">
          <MenuItems :menu-items="menuItems" />
        </ul>
      </div>
      <a href="/" class="brand-link">
        <!-- Assuming the icon is a component -->
        <component :is="icon" class="w-8 h-8" v-if="icon" />
        <span class="text-xl font-semibold">{{ serverConfig.VITE_APP_TITLE }}</span>
      </a>
    </div>
    <div class="navbar-center hidden lg:flex">
      <ul class="main-menu">
        <MenuItems :menu-items="menuItems" />
      </ul>
    </div>
    <div class="navbar-end">
      <div class="flex items-center gap-2">
        <LanguageSwitch />
        <ThemeSwitch />

        <!-- User Avatar Dropdown -->
        <div class="dropdown dropdown-end" v-if="userInfo?.username">
          <div tabindex="0" role="button" class="action-icon avatar">
            <div class="w-10 rounded-full ring-2 ring-white/20">
              <img alt="用户头像" :src="avatar" />
            </div>
          </div>
          <ul tabindex="0" class="menu-dropdown w-52">
            <li v-for="(item, index) in userMenuItems" :key="index" v-show="!item.hide">
              <a @click="item.event">
                {{ item.title }}
                <span v-if="item.badge" class="badge">{{ item.badge }}</span>
              </a>
            </li>
          </ul>
        </div>

        <!-- Login/Register Buttons -->
        <div v-else class="flex items-center gap-2">
          <button class="action-btn" @click="router.push('/auth/login')">
            {{ t('header.login') }}
          </button>
          <button class="action-btn primary" @click="router.push('/auth/register')">
            {{ t('header.register') }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.custom-navbar {
  display: flex;
  align-items: center;
  padding: 0 1.5rem;
  height: 64px;
  position: sticky;
  top: 0;
  z-index: 50;
  
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(20px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);
}

.navbar-start, .navbar-center, .navbar-end {
  display: flex;
  align-items: center;
  flex: 1;
}
.navbar-center { justify-content: center; }
.navbar-end { justify-content: flex-end; }

.brand-link {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  text-decoration: none;
  color: white;
  transition: opacity 0.2s;
}
.brand-link:hover {
  opacity: 0.8;
}

/* Action buttons for icons and avatar */
.action-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 40px;
  height: 40px;
  border-radius: 9999px;
  background-color: transparent;
  color: white;
  border: none;
  cursor: pointer;
  transition: background-color 0.2s;
}
.action-icon:hover {
  background-color: rgba(255, 255, 255, 0.1);
}

/* Main Menu (Desktop) container */
.main-menu {
  display: flex;
  align-items: center;
  gap: 0.5rem; /* spacing between menu items */
}

:deep(.main-menu > li > a) {
  color: rgba(255, 255, 255, 0.7);
  padding: 0.5rem 1rem;
  border-radius: 0.5rem;
  transition: all 0.2s;
}
:deep(.main-menu > li > a:hover) {
  color: white;
  background-color: rgba(255, 255, 255, 0.1);
}
:deep(.main-menu > li > a.active) {
  color: white;
  background-image: linear-gradient(to right, rgba(99, 102, 241, 0.5), rgba(139, 92, 246, 0.5));
}

/* Submenu styles (within Desktop menu) */
:deep(.main-menu .dropdown-content) {
  background: rgba(30, 30, 45, 0.85) !important;
  border: 1px solid rgba(255, 255, 255, 0.2) !important;
  backdrop-filter: blur(12px);
  border-radius: 0.75rem;
  padding: 0.5rem;
  margin-top: 0.75rem;
}
:deep(.main-menu .dropdown-content a) {
  color: rgba(255, 255, 255, 0.8);
  border-radius: 0.5rem;
}
:deep(.main-menu .dropdown-content a:hover) {
  background-color: rgba(255, 255, 255, 0.1);
  color: white;
}

/* Dropdown Menus (Mobile & User) */
.menu-dropdown {
  display: none; /* Hidden by default, shown by dropdown logic */
  position: absolute;
  margin-top: 0.75rem;
  padding: 0.5rem;
  width: 13rem;
  z-index: 50;
  border-radius: 0.75rem;
  background: rgba(30, 30, 45, 0.85); /* SEMI-TRANSPARENT BACKGROUND */
  border: 1px solid rgba(255, 255, 255, 0.2);
  backdrop-filter: blur(12px);
  box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.37);
}
.dropdown:hover .menu-dropdown, .dropdown:focus-within .menu-dropdown {
  display: block; /* Show on hover/focus */
}

:deep(.menu-dropdown li > a) {
  color: rgba(255, 255, 255, 0.8);
  border-radius: 0.5rem;
  display: flex; /* Ensure proper layout inside */
  justify-content: space-between;
  padding: 0.5rem 0.75rem;
}
:deep(.menu-dropdown li > a:hover) {
  background-color: rgba(255, 255, 255, 0.1);
  color: white;
}
.badge {
  background-color: #ec4899;
  color: white;
  border: none;
  font-size: 0.7rem;
  padding: 0.1rem 0.5rem;
  border-radius: 9999px;
}

/* Login/Register Buttons */
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
</style>
