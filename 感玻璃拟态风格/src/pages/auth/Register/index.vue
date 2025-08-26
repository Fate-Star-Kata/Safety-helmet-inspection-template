<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { ElMessage, ElMessageBox } from 'element-plus'
import type { RegisterForm, RegisterRequest, RegisterResponse, CaptchaResponse } from '@/types/apis/auth'
import { register, getCaptCha } from '@/api/user'
import { Lock, Picture, User, Refresh, Message } from '@element-plus/icons-vue'

const router = useRouter()
const formRef = ref()
const loading = ref(false)
const captchaImage = ref('')
const registerForm = ref<RegisterForm>({
  username: '',
  email: '',
  password: '',
  confirmPassword: '',
  captcha_id: '',
  captcha_text: ''
})

// Validation Rules
const rules = {
  username: [
    { required: true, message: '请输入用户名', trigger: 'blur' },
    { min: 3, max: 20, message: '长度在 3 到 20 个字符', trigger: 'blur' },
    { pattern: /^[a-zA-Z0-9_]+$/, message: '仅限字母、数字和下划线', trigger: 'blur' }
  ],
  email: [
    { required: true, message: '请输入邮箱地址', trigger: 'blur' },
    { type: 'email', message: '请输入正确的邮箱地址', trigger: 'blur' }
  ],
  password: [
    { required: true, message: '请输入密码', trigger: 'blur' },
    { min: 6, max: 20, message: '长度在 6 到 20 个字符', trigger: 'blur' },
    { pattern: /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/, message: '需包含大小写字母和数字', trigger: 'blur' }
  ],
  confirmPassword: [
    { required: true, message: '请确认密码', trigger: 'blur' },
    { validator: (rule: any, value: string, callback: Function) => {
        if (value !== registerForm.value.password) {
          callback(new Error('两次输入的密码不一致'))
        } else {
          callback()
        }
      }, trigger: 'blur'
    }
  ],
  captcha_text: [
    { required: true, message: '请输入验证码', trigger: 'blur' },
    { len: 4, message: '验证码为4位', trigger: 'blur' }
  ]
}

// Refresh Captcha
async function refreshCaptcha(): Promise<void> {
  try {
    const response: CaptchaResponse = await getCaptCha()
    if (response.code === 200 && response.data) {
      captchaImage.value = import.meta.env.VITE_SERVER_PATH + response.data.captcha_url
      registerForm.value.captcha_id = response.data.captcha_id
    } else {
      throw new Error(response.msg)
    }
  } catch (error) {
    console.error('获取验证码失败:', error)
    ElMessage({ message: '获取验证码失败', type: 'error', customClass: 'glass-message' });
  }
}

// Handle Form Submission
const handleSubmit = async () => {
  if (!formRef.value) return

  try {
    await formRef.value.validate()
    loading.value = true
    const requestData: RegisterRequest = { username: registerForm.value.username, password: registerForm.value.password, email: registerForm.value.email, captcha_id: registerForm.value.captcha_id, captcha_text: registerForm.value.captcha_text }
    const response: RegisterResponse = await register(requestData)
    if (response.code === 200) {
      ElMessage({ message: '注册成功！', type: 'success', customClass: 'glass-message' });
      await ElMessageBox.alert('注册成功，即将跳转到登录页面', '提示', { confirmButtonText: '确定', type: 'success' })
      router.push('/auth/login')
    } else {
      ElMessage({ message: response.msg || '注册失败', type: 'error', customClass: 'glass-message' });
      await refreshCaptcha()
      registerForm.value.captcha_text = ''
    }
  } catch (error: any) {
    if (error !== false) { // Is not a form validation error
      console.error('注册失败:', error)
    }
    await refreshCaptcha()
    registerForm.value.captcha_text = ''
  } finally {
    loading.value = false
  }
}

// Reset Form
const resetForm = () => {
  if (formRef.value) {
    formRef.value.resetFields()
  }
  refreshCaptcha()
}

onMounted(refreshCaptcha)
</script>

<template>
  <div class="register-container">
    <!-- Background Effects -->
    <div class="background-effects">
      <div class="circle c1"></div>
      <div class="circle c2"></div>
      <div class="circle c3"></div>
      <div class="circle c4"></div>
    </div>

    <!-- Register Form Panel -->
    <div class="register-panel">
      <!-- Header -->
      <div class="form-header">
        <h1 class="title">创建您的账户</h1>
        <p class="subtitle">欢迎加入，开启全新体验</p>
      </div>

      <!-- Form -->
      <el-form ref="formRef" :model="registerForm" :rules="rules" size="large" class="register-form" @submit.prevent="handleSubmit">
        <el-form-item prop="username">
          <el-input v-model="registerForm.username" placeholder="设置用户名" :prefix-icon="User" clearable />
        </el-form-item>
        <el-form-item prop="email">
          <el-input v-model="registerForm.email" type="email" placeholder="输入邮箱地址" :prefix-icon="Message" clearable />
        </el-form-item>
        <el-form-item prop="password">
          <el-input v-model="registerForm.password" type="password" placeholder="创建密码" :prefix-icon="Lock" show-password clearable />
        </el-form-item>
        <el-form-item prop="confirmPassword">
          <el-input v-model="registerForm.confirmPassword" type="password" placeholder="确认密码" :prefix-icon="Lock" show-password clearable />
        </el-form-item>
        <el-form-item prop="captcha_text">
          <div class="flex items-center gap-4 w-full">
            <el-input v-model="registerForm.captcha_text" placeholder="验证码" :prefix-icon="Picture" maxlength="4" clearable class="flex-grow" />
            <img v-if="captchaImage" :src="captchaImage" alt="验证码" class="captcha-image" @click="refreshCaptcha" title="点击刷新验证码" />
            <div v-else class="captcha-placeholder" @click="refreshCaptcha"><el-icon><Refresh /></el-icon></div>
          </div>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" :loading="loading" @click="handleSubmit" class="submit-btn">
            {{ loading ? '注册中...' : '立即注册' }}
          </el-button>
        </el-form-item>
      </el-form>

      <!-- Footer Links -->
      <div class="form-footer">
        <router-link to="/auth/login" class="footer-link">已有账号？ 前往登录</router-link>
      </div>
    </div>
  </div>
</template>

<style scoped>
.register-container {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background-image: linear-gradient(to bottom right, #1f2937, #111827);
  position: relative;
  overflow: hidden;
}

/* Background Effects */
.background-effects { position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; }
.circle { position: absolute; border-radius: 50%; animation: float 20s ease-in-out infinite; opacity: 0.1; }
.c1 { width: 300px; height: 300px; top: 10%; left: 5%; background: #6366f1; }
.c2 { width: 400px; height: 400px; top: 60%; right: 10%; background: #8b5cf6; animation-delay: 5s; }
.c3 { width: 250px; height: 250px; bottom: 5%; left: 30%; background: #ec4899; animation-delay: 10s; }
.c4 { width: 150px; height: 150px; top: 20%; right: 40%; background: #38bdf8; animation-delay: 15s; }
@keyframes float {
  0%, 100% { transform: translateY(0px) translateX(0px) rotate(0deg); }
  50% { transform: translateY(-40px) translateX(20px) rotate(180deg); }
}

/* Register Panel */
.register-panel {
  position: relative;
  z-index: 10;
  width: 100%;
  max-width: 450px;
  padding: 2.5rem;
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(20px);
  border-radius: 1rem;
  border: 1px solid rgba(255, 255, 255, 0.2);
  box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.37);
}

.form-header { text-align: center; margin-bottom: 2rem; }
.title {
  font-size: 1.75rem;
  font-weight: 600;
  background: linear-gradient(to right, #e0e7ff, #c7d2fe);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}
.subtitle { margin-top: 0.5rem; font-size: 0.875rem; color: rgba(255, 255, 255, 0.6); }

/* Form Styling */
:deep(.el-form-item) { margin-bottom: 1.25rem; }
:deep(.el-input__wrapper) {
  background-color: rgba(0, 0, 0, 0.2) !important;
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  box-shadow: none !important;
  padding: 0.25rem 1rem;
  height: 48px;
}
:deep(.el-input__inner) { color: rgba(255, 255, 255, 0.9) !important; }
:deep(.el-input__prefix .el-icon) { color: rgba(255, 255, 255, 0.6); }

/* Captcha */
.captcha-image, .captcha-placeholder {
  width: 120px;
  height: 48px;
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  cursor: pointer;
  transition: all 0.2s;
}
.captcha-image:hover, .captcha-placeholder:hover { border-color: rgba(167, 139, 250, 0.8); }
.captcha-placeholder { display: flex; align-items: center; justify-content: center; background-color: rgba(0, 0, 0, 0.2); color: rgba(255, 255, 255, 0.6); }

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
.form-footer { margin-top: 1.5rem; text-align: center; font-size: 0.875rem; }
.footer-link { color: rgba(255, 255, 255, 0.6); text-decoration: none; transition: color 0.2s; }
.footer-link:hover { color: #a78bfa; }
</style>
