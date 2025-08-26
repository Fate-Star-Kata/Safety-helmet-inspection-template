<script setup lang="ts">
import { computed } from 'vue'
import { useRoute } from 'vue-router'
import { useI18n } from '@/utils/useI18n'
import Footer from './Footer.vue'
import Header from './Header.vue'
// eslint-disable-next-line unused-imports/no-unused-vars
const { t } = useI18n()
const route = useRoute()

// 判断是否在auth路径下
const isAuthPage = computed(() => {
  return route.path.startsWith('/auth')
})
</script>

<template>
  <div class="app-layout">
    <!-- Background Effects -->
    <div class="background-effects">
      <div class="circle c1"></div>
      <div class="circle c2"></div>
      <div class="circle c3"></div>
      <div class="circle c4"></div>
    </div>

    <!-- Conditional rendering for main app layout vs. auth pages -->
    <template v-if="!isAuthPage">
      <Header />
      <main class="main-content">
        <slot>
          <!-- RouterView will be rendered here -->
        </slot>
      </main>
      <Footer />
    </template>
    
    <template v-else>
      <main class="auth-content">
        <slot>
          <!-- Login/Register pages will be rendered here -->
        </slot>
      </main>
    </template>
  </div>
</template>

<style scoped>
.app-layout {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  position: relative;
  overflow: hidden;
  background-image: linear-gradient(to bottom right, #111827, #1f2937);
}

/* Main Content Area (for pages with Header/Footer) */
.main-content {
  flex: 1; /* Takes up all available space between header and footer */
  overflow-y: auto; /* Enables scrolling for the content area only */
  position: relative;
  z-index: 10;
}

/* Auth Content Area (for Login/Register pages) */
.auth-content {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
}

/* Custom Scrollbar for Main Content */
.main-content::-webkit-scrollbar {
  width: 6px;
}
.main-content::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.1);
}
.main-content::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.3);
  border-radius: 3px;
}
.main-content::-webkit-scrollbar-thumb:hover {
  background: rgba(255, 255, 255, 0.5);
}

/* Background Effects */
.background-effects {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 1;
}
.circle {
  position: absolute;
  border-radius: 50%;
  animation: float 20s ease-in-out infinite;
  opacity: 0.1;
  filter: blur(100px);
}
.c1 { width: 400px; height: 400px; top: -50px; left: -100px; background: #6366f1; }
.c2 { width: 500px; height: 500px; top: 50%; right: -150px; background: #8b5cf6; animation-delay: 5s; }
.c3 { width: 350px; height: 350px; bottom: -100px; left: 20%; background: #ec4899; animation-delay: 10s; }
.c4 { width: 250px; height: 250px; top: 10%; right: 30%; background: #38bdf8; animation-delay: 15s; }

@keyframes float {
  0%, 100% { transform: translateY(0px) translateX(0px) rotate(0deg); }
  50% { transform: translateY(-40px) translateX(20px) rotate(180deg); }
}
</style>
