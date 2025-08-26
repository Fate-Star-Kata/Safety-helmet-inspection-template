<script setup lang="ts">
import { getCaptCha, login } from '@/api/user'
import type { LoginReq, LoginFormRules, CaptchaResponse, LoginResNoToken } from '@/types/apis/auth'
import { Lock, Picture, User } from '@element-plus/icons-vue'
import { ElMessage } from 'element-plus'
import { onMounted, ref } from 'vue'
import router from '@/router'
import { useUserStore } from '@/stores/auth/user'

const store = useUserStore();
const captchaImage = ref('')
const loading = ref(false)
const loginFormRef = ref()

const loginForm = ref<LoginReq>({
  username: '',
  password: '',
  captcha_id: '',
  captcha_text: '',
})

// 表单验证规则
const rules: LoginFormRules = {
  username: [
    { required: true, message: '请输入用户名', trigger: 'blur' },
    { min: 3, max: 20, message: '用户名长度在 3 到 20 个字符', trigger: 'blur' },
  ],
  password: [
    { required: true, message: '请输入密码', trigger: 'blur' },
    { min: 6, max: 20, message: '密码长度在 6 到 20 个字符', trigger: 'blur' },
  ],
  captcha_text: [
    { required: true, message: '请输入验证码', trigger: 'blur' },
    { len: 4, message: '验证码长度为 4 个字符', trigger: 'blur' },
  ],
}

// 刷新验证码
async function refreshCaptcha(): Promise<void> {
  try {
    const response: CaptchaResponse = await getCaptCha()
    if (response.code === 200 && response.data) {
      captchaImage.value = import.meta.env.VITE_SERVER_PATH + response.data.captcha_url
      loginForm.value.captcha_id = response.data.captcha_id
    } else {
      throw new Error(response.msg)
    }
  }
  catch (error) {
    console.error('获取验证码失败:', error)
    ElMessage({ message: '获取验证码失败', type: 'error', customClass: 'glass-message' });
  }
}

// 登录处理
async function handleLogin(): Promise<void> {
  if (!loginFormRef.value) return

  try {
    await loginFormRef.value.validate()
    loading.value = true
    const response: LoginResNoToken = await login(loginForm.value)
    if (response.code === 200 && response.data) {
      store.setUserInfo(response.data)
      ElMessage({ message: response.msg || '登录成功！', type: 'success', customClass: 'glass-message' });
      router.push('/')
    } else {
      ElMessage({ message: response.msg || '登录失败！', type: 'error', customClass: 'glass-message' });
      await refreshCaptcha()
      throw new Error(response.msg || '登录失败')
    }
  }
  catch (error) {
    if (error !== false) { // Is not a form validation error
      console.error('登录失败:', error)
      await refreshCaptcha()
    }
  }
  finally {
    loading.value = false
  }
}

// 组件挂载时获取验证码
onMounted(() => {
  refreshCaptcha()
  if (store.getUserInfo() != null) {
    store.setUserInfo(null)
  }
})
</script>

<template>
  <div class="login-container">
    <!-- Background Effects -->
    <div class="background-effects">
      <div class="circle c1"></div>
      <div class="circle c2"></div>
      <div class="circle c3"></div>
      <div class="circle c4"></div>
    </div>

    <!-- Login Form Panel -->
    <div class="login-panel">
      <!-- Header -->
      <div class="form-header">
        <h1 class="title">工地安全帽检测系统</h1>
        <p class="subtitle">欢迎回来，请登录您的账户</p>
      </div>

      <!-- Form -->
      <el-form ref="loginFormRef" :model="loginForm" :rules="rules" size="large" class="login-form" @submit.prevent="handleLogin">
        <!-- Username -->
        <el-form-item prop="username">
          <el-input v-model="loginForm.username" placeholder="用户名" :prefix-icon="User" clearable />
        </el-form-item>

        <!-- Password -->
        <el-form-item prop="password">
          <el-input v-model="loginForm.password" type="password" placeholder="密码" :prefix-icon="Lock" show-password clearable />
        </el-form-item>

        <!-- Captcha -->
        <el-form-item prop="captcha_text">
          <div class="flex items-center gap-4 w-full">
            <el-input v-model="loginForm.captcha_text" placeholder="验证码" :prefix-icon="Picture" maxlength="4" clearable class="flex-grow" />
            <img v-if="captchaImage" :src="captchaImage" alt="验证码" class="captcha-image" @click="refreshCaptcha" title="点击刷新验证码" />
            <div v-else class="captcha-placeholder" @click="refreshCaptcha">
              <el-icon><Refresh /></el-icon>
            </div>
          </div>
        </el-form-item>

        <!-- Login Button -->
        <el-form-item>
          <el-button type="primary" :loading="loading" @click="handleLogin" class="submit-btn">
            {{ loading ? '登录中...' : '立即登录' }}
          </el-button>
        </el-form-item>
      </el-form>

      <!-- Footer Links -->
      <div class="form-footer">
        <router-link to="#" class="footer-link">忘记密码？</router-link>
        <router-link to="/auth/Register" class="footer-link">还没有账号？ 立即注册</router-link>
      </div>
    </div>
  </div>
</template>

<style scoped>
.login-container {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background-image: linear-gradient(to bottom right, #1f2937, #111827);
  position: relative;
  overflow: hidden;
}

/* Background Effects */
.background-effects {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
}
.circle {
  position: absolute;
  border-radius: 50%;
  animation: float 20s ease-in-out infinite;
  opacity: 0.1;
}
.c1 { width: 300px; height: 300px; top: 10%; left: 5%; background: #6366f1; }
.c2 { width: 400px; height: 400px; top: 60%; right: 10%; background: #8b5cf6; animation-delay: 5s; }
.c3 { width: 250px; height: 250px; bottom: 5%; left: 30%; background: #ec4899; animation-delay: 10s; }
.c4 { width: 150px; height: 150px; top: 20%; right: 40%; background: #38bdf8; animation-delay: 15s; }
@keyframes float {
  0%, 100% { transform: translateY(0px) translateX(0px) rotate(0deg); }
  50% { transform: translateY(-40px) translateX(20px) rotate(180deg); }
}

/* Login Panel */
.login-panel {
  position: relative;
  z-index: 10;
  width: 100%;
  max-width: 420px;
  padding: 2.5rem;
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(20px);
  border-radius: 1rem;
  border: 1px solid rgba(255, 255, 255, 0.2);
  box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.37);
}

.form-header {
  text-align: center;
  margin-bottom: 2rem;
}
.title {
  font-size: 1.75rem;
  font-weight: 600;
  background: linear-gradient(to right, #e0e7ff, #c7d2fe);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}
.subtitle {
  margin-top: 0.5rem;
  font-size: 0.875rem;
  color: rgba(255, 255, 255, 0.6);
}

/* Form Styling */
:deep(.el-form-item) {
  margin-bottom: 1.5rem;
}
:deep(.el-input__wrapper) {
  background-color: rgba(0, 0, 0, 0.2) !important;
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  box-shadow: none !important;
  padding: 0.25rem 1rem;
  height: 48px;
}
:deep(.el-input__inner) {
  color: rgba(255, 255, 255, 0.9) !important;
}
:deep(.el-input__prefix .el-icon) {
  color: rgba(255, 255, 255, 0.6);
}

/* Captcha */
.captcha-image, .captcha-placeholder {
  width: 120px;
  height: 48px;
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  cursor: pointer;
  transition: all 0.2s;
}
.captcha-image:hover, .captcha-placeholder:hover {
  border-color: rgba(167, 139, 250, 0.8);
}
.captcha-placeholder {
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: rgba(0, 0, 0, 0.2);
  color: rgba(255, 255, 255, 0.6);
}

/* Submit Button */
.submit-btn {
  width: 100%;
  height: 48px;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 500;
  border: none;
  background-image: linear-gradient(to right, #6366f1, #8b5cf6);
  transition: all 0.3s ease;
}
.submit-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(139, 92, 246, 0.4);
}

/* Footer Links */
.form-footer {
  margin-top: 1.5rem;
  display: flex;
  justify-content: space-between;
  font-size: 0.875rem;
}
.footer-link {
  color: rgba(255, 255, 255, 0.6);
  text-decoration: none;
  transition: color 0.2s;
}
.footer-link:hover {
  color: #a78bfa;
}
</style>
