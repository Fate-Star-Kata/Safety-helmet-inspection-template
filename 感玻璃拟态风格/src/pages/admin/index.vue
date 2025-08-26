<!-- 关闭这个页面 Motion 组件的所有ts检测 -->
<script setup lang="ts">
import { ref } from 'vue'
import { Motion } from 'motion-v'
import AdminHeader from '@/components/pages/admin/header.vue'
import AdminNavbar from '@/components/pages/admin/navbar.vue'

// 侧边栏折叠状态
const isCollapse = ref(false)

// 切换侧边栏折叠状态
function toggleSidebar() {
  isCollapse.value = !isCollapse.value
}

// 动画配置 (These are well-defined and compatible with the new theme)
const pageVariants = {
  initial: { opacity: 0, y: 20 },
  animate: { opacity: 1, y: 0 },
  transition: { duration: 0.6, ease: ['easeOut'] }
}

const sidebarVariants = {
  initial: { x: -200, opacity: 0 },
  animate: { x: 0, opacity: 1 },
  transition: { duration: 0.5, ease: ['easeOut'] }
}

const sidebarWidthVariants = {
  expanded: { width: '200px' },
  collapsed: { width: '64px' },
  transition: { duration: 0.3, ease: 'easeOut' }
}

const headerVariants = {
  initial: { y: -60, opacity: 0 },
  animate: { y: 0, opacity: 1 },
  transition: { duration: 0.5, delay: 0.2, ease: ['easeOut'] }
}

const cardVariants = {
  initial: { opacity: 0, y: 30, scale: 0.95 },
  animate: { opacity: 1, y: 0, scale: 1 },
  whileHover: {
    scale: 1.02,
    y: -5,
    transition: { duration: 0.2, ease: ['easeOut'] }
  },
  transition: { duration: 0.4, ease: ['easeOut'] }
}
</script>


<template>
  <Motion :initial="pageVariants.initial" :animate="pageVariants.animate" :transition="pageVariants.transition as any">
    <!-- Main container with the cosmic gradient background -->
    <el-container
      class="h-screen bg-gradient-to-br from-indigo-900 via-purple-900 to-pink-900 relative overflow-hidden">

      <!-- Background Effects -->
      <div class="absolute inset-0 opacity-20 -z-10">
        <div class="absolute top-20 left-20 w-72 h-72 bg-blue-400 rounded-full blur-3xl animate-pulse"></div>
        <div class="absolute top-40 right-32 w-96 h-96 bg-purple-400 rounded-full blur-3xl animate-pulse delay-1000">
        </div>
        <div class="absolute bottom-20 left-1/3 w-80 h-80 bg-pink-400 rounded-full blur-3xl animate-pulse delay-2000">
        </div>
      </div>

      <!-- Sidebar with Glassmorphism style -->
      <Motion :animate="isCollapse ? sidebarWidthVariants.collapsed : sidebarWidthVariants.expanded"
        :transition="sidebarWidthVariants.transition as any"
        class="el-aside-wrapper backdrop-blur-xl bg-white/10 border-r border-white/20">
        <el-aside class="el-aside h-full">
          <Motion :initial="sidebarVariants.initial" :animate="sidebarVariants.animate"
            :transition="sidebarVariants.transition as any">
            <AdminNavbar :is-collapse="isCollapse" />
          </Motion>
        </el-aside>
      </Motion>

      <!-- Main content area -->
      <el-container class="main-content-transition">
        <!-- Header with Glassmorphism style -->
        <el-header class="el-header backdrop-blur-xl bg-black/10 border-b border-white/20"
          style="overflow: visible; position: relative; z-index: 50; height: 60px;">
          <Motion :initial="headerVariants.initial" :animate="headerVariants.animate"
            :transition="headerVariants.transition as any" class="h-full">
            <AdminHeader @toggle-sidebar="toggleSidebar" :is-collapse="isCollapse" />
          </Motion>
        </el-header>

        <!-- Main content with transparent background to show the gradient -->
        <el-main class="el-main">
          <div class="p-6">
            <!-- Child route content will appear here -->
            <Motion :initial="cardVariants.initial" :animate="cardVariants.animate"
              :transition="{ ...cardVariants.transition, delay: 0.2 } as any">
              <router-view />
            </Motion>
          </div>
        </el-main>
      </el-container>
    </el-container>
  </Motion>
</template>

<style scoped>
/* Remove styles that are now handled by utility classes or are no longer needed */
.el-aside-wrapper {
  /* background-color and box-shadow are replaced by glassmorphism classes */
  position: relative;
  overflow: hidden;
  /* Prevent flexbox from shrinking the sidebar width on content overflow */
  flex: 0 0 auto;
}

.el-aside {
  /* Make aside transparent to let the wrapper's style show through */
  background-color: transparent;
  width: 100% !important;
  position: relative;
  overflow: hidden;
}

.el-header {
  /* background-color is replaced, padding is handled by AdminHeader */
  padding: 0;
  line-height: 60px;
}

.el-main {
  /* background-color is removed to show the root gradient, padding is moved to an inner div */
  padding: 0;
  overflow-y: auto;
}

/* Main content area transition remains useful */
.main-content-transition {
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

/* Responsive animation optimization */
@media (prefers-reduced-motion: reduce) {
  .main-content-transition {
    transition: none;
  }
}

/* Base container style */
.el-container {
  position: relative;
  /* Setting the background is now done via utility classes on the component itself */
}

/* Custom scrollbar for glassmorphism theme, applied to el-main */
.el-main::-webkit-scrollbar {
  width: 6px;
}

.el-main::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 3px;
}

.el-main::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.3);
  border-radius: 3px;
}

.el-main::-webkit-scrollbar-thumb:hover {
  background: rgba(255, 255, 255, 0.5);
}
</style>
