<template>
  <div class="auth-page">
    <!-- 动态背景层 -->
    <div class="bg-layer">
      <div class="floating-shapes">
        <div class="shape shape-1"></div>
        <div class="shape shape-2"></div>
        <div class="shape shape-3"></div>
        <div class="shape shape-4"></div>
        <div class="shape shape-5"></div>
      </div>
      <div class="grid-overlay"></div>
    </div>

    <!-- 主容器：左右布局 -->
    <div class="auth-container" :class="{ 'register-mode': mode === 'register' }">
      <!-- 左侧：特性展示 -->
      <div class="features-section">
        <div class="features-header">
          <h3 class="features-title">为什么选择我们</h3>
          <p class="features-subtitle">专业的 AI 模型中转服务</p>
        </div>
        <div class="feature-list">
          <div class="feature-item" v-for="(feat, idx) in features" :key="idx">
            <div class="feature-icon">
              <a-icon :type="feat.icon" />
            </div>
            <div class="feature-content">
              <h4>{{ feat.title }}</h4>
              <p>{{ feat.desc }}</p>
            </div>
          </div>
        </div>
        <div class="features-footer">
          <div class="stat-badge">
            <span class="stat-number">17+</span>
            <span class="stat-label">AI 模型</span>
          </div>
          <div class="stat-badge">
            <span class="stat-number">99.9%</span>
            <span class="stat-label">可用性</span>
          </div>
        </div>
      </div>

      <!-- 右侧：登录表单 -->
      <div class="form-section">
        <!-- 顶部品牌区 -->
        <div class="brand-header">
          <div class="logo-section">
            <div class="logo-ring">
              <a-icon type="swap" />
            </div>
            <div class="brand-info">
              <h1 class="brand-title">{{ siteConfig.site_name || 'RelayX AI' }}</h1>
              <p class="brand-subtitle">{{ siteConfig.site_subtitle || '一站式 AI 模型调用服务' }}</p>
            </div>
          </div>
        </div>

        <!-- 表单区 -->
        <div class="form-panel">
        <div class="form-wrapper">
          <!-- 切换标签 -->
          <div class="auth-tabs">
            <button
              class="tab-btn"
              :class="{ active: mode === 'login' }"
              @click="switchMode('login')"
            >
              {{ isAgentLogin ? '代理登录' : '登录' }}
            </button>
            <button
              v-if="!isAgentLogin"
              class="tab-btn"
              :class="{ active: mode === 'register' }"
              @click="switchMode('register')"
            >
              注册
            </button>
            <div class="tab-indicator" :class="{ 'at-register': mode === 'register' }"></div>
          </div>

          <!-- 欢迎文字 -->
          <transition name="text-fade" mode="out-in">
            <div class="welcome-text" :key="mode">
              <h2>{{ welcomeTitle }}</h2>
              <p>{{ welcomeSubtitle }}</p>
            </div>
          </transition>

          <!-- 登录表单 -->
          <transition name="form-slide" mode="out-in">
            <a-form
              v-if="mode === 'login'"
              :form="loginForm"
              @submit="handleLogin"
              key="login-form"
              class="auth-form"
            >
              <div class="input-group" :class="{ focused: focusedField === 'l-username', filled: loginValues.username }">
                <label>用户名</label>
                <a-form-item>
                  <a-input
                    v-decorator="['username', { rules: [{ required: true, message: '请输入用户名' }] }]"
                    @focus="focusedField = 'l-username'"
                    @blur="handleBlur('l-username', 'login', 'username')"
                    @change="e => loginValues.username = e.target.value"
                    size="large"
                  >
                    <a-icon slot="prefix" type="user" class="input-icon" />
                  </a-input>
                </a-form-item>
              </div>

              <div class="input-group" :class="{ focused: focusedField === 'l-password', filled: loginValues.password }">
                <label>密码</label>
                <a-form-item>
                  <a-input-password
                    v-decorator="['password', { rules: [{ required: true, message: '请输入密码' }] }]"
                    @focus="focusedField = 'l-password'"
                    @blur="handleBlur('l-password', 'login', 'password')"
                    @change="e => loginValues.password = e.target.value"
                    size="large"
                  >
                    <a-icon slot="prefix" type="lock" class="input-icon" />
                  </a-input-password>
                </a-form-item>
              </div>

              <a-form-item>
                <a-button
                  type="primary"
                  html-type="submit"
                  :loading="loading"
                  block
                  size="large"
                  class="submit-btn"
                  :class="{ 'btn-loading': loading }"
                >
                  <span v-if="!loading" class="btn-content">
                    <span>登 录</span>
                    <a-icon type="arrow-right" class="btn-arrow" />
                  </span>
                </a-button>
              </a-form-item>

              <div v-if="!isAgentLogin" class="login-actions">
                <a @click.prevent="openForgotPassword">忘记密码？</a>
              </div>

              <div v-if="!isAgentLogin" class="form-footer">
                <span>还没有账户？</span>
                <a @click.prevent="switchMode('register')">立即注册</a>
              </div>
            </a-form>

            <!-- 注册表单 -->
            <a-form
              v-else
              :form="registerForm"
              @submit="handleRegister"
              key="register-form"
              class="auth-form"
            >
              <div class="input-group" :class="{ focused: focusedField === 'r-username', filled: registerValues.username }">
                <label>用户名</label>
                <a-form-item>
                  <a-input
                    v-decorator="['username', { rules: [{ required: true, message: '请输入用户名' }, { min: 3, max: 20, message: '用户名长度为 3-20 个字符' }] }]"
                    @focus="focusedField = 'r-username'"
                    @blur="handleBlur('r-username', 'register', 'username')"
                    @change="e => registerValues.username = e.target.value"
                    size="large"
                  >
                    <a-icon slot="prefix" type="user" class="input-icon" />
                  </a-input>
                </a-form-item>
              </div>

              <div class="input-group" :class="{ focused: focusedField === 'r-email', filled: registerValues.email }">
                <label>邮箱</label>
                <a-form-item>
                  <a-input
                    v-decorator="['email', { rules: [{ required: true, message: '请输入邮箱' }, { type: 'email', message: '请输入有效的邮箱地址' }] }]"
                    @focus="focusedField = 'r-email'"
                    @blur="handleBlur('r-email', 'register', 'email')"
                    @change="e => registerValues.email = e.target.value"
                    size="large"
                  >
                    <a-icon slot="prefix" type="mail" class="input-icon" />
                  </a-input>
                </a-form-item>
              </div>

              <div class="input-group" :class="{ focused: focusedField === 'r-password', filled: registerValues.password }">
                <label>密码</label>
                <a-form-item>
                  <a-input-password
                    v-decorator="['password', { rules: [{ required: true, message: '请输入密码' }, { min: 6, message: '密码至少 6 个字符' }, { validator: validateToNextPassword }] }]"
                    @focus="focusedField = 'r-password'"
                    @blur="handleBlur('r-password', 'register', 'password')"
                    @change="handlePasswordChange"
                    size="large"
                  >
                    <a-icon slot="prefix" type="lock" class="input-icon" />
                  </a-input-password>
                </a-form-item>
                <!-- 密码强度指示器 -->
                <div class="password-strength" v-if="registerValues.password">
                  <div class="strength-bars">
                    <span class="bar" :class="{ active: passwordStrength >= 1, weak: passwordStrength === 1, medium: passwordStrength === 2, strong: passwordStrength >= 3 }"></span>
                    <span class="bar" :class="{ active: passwordStrength >= 2, medium: passwordStrength === 2, strong: passwordStrength >= 3 }"></span>
                    <span class="bar" :class="{ active: passwordStrength >= 3, strong: passwordStrength >= 3 }"></span>
                  </div>
                  <span class="strength-text" :class="{ weak: passwordStrength === 1, medium: passwordStrength === 2, strong: passwordStrength >= 3 }">
                    {{ strengthLabel }}
                  </span>
                </div>
              </div>

              <div class="input-group" :class="{ focused: focusedField === 'r-confirm', filled: registerValues.confirmPassword }">
                <label>确认密码</label>
                <a-form-item>
                  <a-input-password
                    v-decorator="['confirmPassword', { rules: [{ required: true, message: '请确认密码' }, { validator: compareToFirstPassword }] }]"
                    @focus="focusedField = 'r-confirm'"
                    @blur="handleConfirmBlur"
                    @change="e => registerValues.confirmPassword = e.target.value"
                    size="large"
                  >
                    <a-icon slot="prefix" type="safety-certificate" class="input-icon" />
                  </a-input-password>
                </a-form-item>
              </div>

              <a-form-item>
                <a-button
                  type="primary"
                  html-type="submit"
                  :loading="loading"
                  block
                  size="large"
                  class="submit-btn"
                  :class="{ 'btn-loading': loading }"
                >
                  <span v-if="!loading" class="btn-content">
                    <span>注 册</span>
                    <a-icon type="arrow-right" class="btn-arrow" />
                  </span>
                </a-button>
              </a-form-item>

              <div class="form-footer">
                <span>已有账户？</span>
                <a @click.prevent="switchMode('login')">立即登录</a>
              </div>
            </a-form>
          </transition>
        </div>

        <!-- 版权信息 -->
        <div class="copyright-text">@2026 RelayX AI</div>
      </div>
      </div>
    </div>

    <!-- AI 模型 Logo 底部展示 -->
    <div class="ai-logos-bottom">
      <div class="logos-container">
        <div class="logo-item" v-for="(logo, index) in aiLogos" :key="index" :style="{ animationDelay: `${index * 0.2}s` }">
          <div class="logo-card">
            <div class="logo-icon" :style="{ background: logo.logo ? 'transparent' : logo.color }">
              <img v-if="logo.logo" :src="logo.logo" :alt="logo.name" class="logo-image" />
              <span v-else>{{ logo.name.substring(0, 2) }}</span>
            </div>
            <span class="logo-name">{{ logo.name }}</span>
          </div>
        </div>
      </div>
    </div>

    <a-modal
      title="找回密码"
      :visible="forgotVisible"
      :confirm-loading="forgotLoading"
      :ok-text="forgotStep === 'identity' ? '验证身份' : '重置密码'"
      cancel-text="取消"
      :mask-closable="false"
      :closable="!forgotLoading"
      centered
      wrapClassName="forgot-password-modal"
      @ok="handleForgotPassword"
      @cancel="closeForgotPassword"
    >
      <a-form :form="forgotForm" layout="vertical" class="forgot-form">
        <div class="forgot-step-tip">
          <span v-if="forgotStep === 'identity'">先校验账号和邮箱，校验通过后再设置新密码</span>
          <span v-else>身份已通过校验，现在请输入新的登录密码</span>
          <a v-if="forgotStep === 'reset'" @click.prevent="resetForgotIdentityStep">重新验证</a>
        </div>
        <a-form-item label="账号">
          <a-input
            v-decorator="['username', { rules: [{ required: true, message: '请输入账号' }] }]"
            placeholder="请输入注册账号"
            size="large"
            :disabled="forgotStep === 'reset'"
          >
            <a-icon slot="prefix" type="user" />
          </a-input>
        </a-form-item>
        <a-form-item label="邮箱">
          <a-input
            v-decorator="['email', { rules: [{ required: true, message: '请输入邮箱' }, { type: 'email', message: '请输入有效的邮箱地址' }] }]"
            placeholder="请输入注册邮箱"
            size="large"
            :disabled="forgotStep === 'reset'"
          >
            <a-icon slot="prefix" type="mail" />
          </a-input>
        </a-form-item>
        <a-form-item v-if="forgotStep === 'reset'" label="新密码">
          <a-input-password
            v-decorator="['new_password', { rules: [{ required: true, message: '请输入新密码' }, { min: 6, message: '密码至少 6 个字符' }, { validator: validateForgotPassword }] }]"
            placeholder="请输入新的登录密码"
            size="large"
            @change="handleForgotPasswordChange"
          >
            <a-icon slot="prefix" type="lock" />
          </a-input-password>
        </a-form-item>
        <a-form-item v-if="forgotStep === 'reset'" label="确认新密码">
          <a-input-password
            v-decorator="['confirm_password', { rules: [{ required: true, message: '请再次输入新密码' }, { validator: compareForgotPassword }] }]"
            placeholder="请再次输入新的登录密码"
            size="large"
            @blur="handleForgotConfirmBlur"
          >
            <a-icon slot="prefix" type="safety-certificate" />
          </a-input-password>
        </a-form-item>
      </a-form>
    </a-modal>
  </div>
</template>

<script>
import { getPublicSiteConfig } from '@/api/public'
import { forgotPassword, verifyForgotPasswordIdentity } from '@/api/auth'

export default {
  name: 'Login',
  data() {
    return {
      mode: 'login', // 'login' 或 'register'
      loading: false,
      forgotLoading: false,
      forgotVisible: false,
      forgotStep: 'identity',
      focusedField: null,
      confirmDirty: false,
      forgotConfirmDirty: false,
      loginValues: { username: '', password: '' },
      registerValues: { username: '', email: '', password: '', confirmPassword: '' },
      forgotValues: { username: '', email: '', newPassword: '', confirmPassword: '' },
      // 品牌区特性列表
      features: [
        { icon: 'api', title: '多模型支持', desc: '接入 GPT、Claude、Gemini 等主流模型' },
        { icon: 'dashboard', title: '智能路由', desc: '自动负载均衡，智能故障转移' },
        { icon: 'safety-certificate', title: '安全可靠', desc: '企业级安全防护，数据加密传输' }
      ],
      // AI 模型 Logo 列表
      aiLogos: [
        { name: 'GPT', color: 'linear-gradient(135deg, #10a37f, #1a7f64)', logo: 'https://cdn.oaistatic.com/_next/static/media/apple-touch-icon.59f2e898.png' },
        { name: 'Claude', color: 'linear-gradient(135deg, #d97757, #c45a3a)', logo: 'https://registry.npmmirror.com/@lobehub/icons-static-png/latest/files/light/claude-color.png' },
        { name: 'Gemini', color: 'linear-gradient(135deg, #4285f4, #34a853)', logo: 'https://www.gstatic.com/lamda/images/gemini_sparkle_v002_d4735304ff6292a690345.svg' },
        { name: 'LLaMA', color: 'linear-gradient(135deg, #0668e1, #0552b5)', logo: 'https://www.zuoshipin.com/wp-content/uploads/2023/11/ea5f9-ai.meta.com.png' },
        { name: 'Mistral', color: 'linear-gradient(135deg, #f2a900, #d89000)', logo: 'https://avatars.githubusercontent.com/u/132372032?s=200&v=4' },
        { name: 'Qwen', color: 'linear-gradient(135deg, #ff6a00, #ee5a00)', logo: 'https://assets.alicdn.com/g/qwenweb/qwen-webui-fe/0.0.54/static/favicon.png' },
        { name: 'DeepSeek', color: 'linear-gradient(135deg, #6366f1, #4f46e5)', logo: 'https://s1.aigei.com/src/img/png/03/0305d15156154b85a80848ae4edd22ab.png?imageMogr2/auto-orient/thumbnail/!282x282r/gravity/Center/crop/282x282/quality/85/%7CimageView2/2/w/282&e=2051020800&token=P7S2Xpzfz11vAkASLTkfHN7Fw-oOZBecqeJaxypL:eFZ0GvEP17SkCu1zdapd0tTtlTw=' },
        { name: 'GLM', color: 'linear-gradient(135deg, #06b6d4, #0891b2)', logo: 'https://ts1.tc.mm.bing.net/th/id/OIP-C.J-rEPlF829AMwLOWrIaD7AHaHa?rs=1&pid=ImgDetMain&o=7&rm=3' },
        { name: 'Grok', color: 'linear-gradient(135deg, #000000, #333333)', logo: 'https://www.freelogovectors.net/wp-content/uploads/2025/06/grok_logo-freelogovectors.net_.png' },
        { name: 'ERNIE Bot', color: 'linear-gradient(135deg, #2932e1, #1a1f8f)', logo: 'https://pic1.zhimg.com/v2-a0b74ce4adfeb43f3d9b13fee094083a.jpg?source=57bbeac9' },
        { name: 'Hunyuan', color: 'linear-gradient(135deg, #006eff, #0052cc)', logo: 'https://cdn-3d-prod.hunyuan.tencent.com/public/static/favicon/android-chrome-512x512.png' },
        { name: 'Doubao', color: 'linear-gradient(135deg, #3370ff, #245bdb)', logo: 'https://ts1.tc.mm.bing.net/th/id/R-C.52fb7ce149641a9d12431a3b00dca540?rik=ejDbB%2bDoDbD5Qw&riu=http%3a%2f%2fai.kukuwg.com%2fwp-content%2fuploads%2f2024%2f09%2fdoubao-llm-logo1.png&ehk=Uaz4H4re4naHCr4WrAUzM7AOvS2rylE0YeBv0l%2bOTmk%3d&risl=&pid=ImgRaw&r=0' },
        { name: 'Kimi', color: 'linear-gradient(135deg, #7c3aed, #6d28d9)', logo: 'https://statics.moonshot.cn/kimi-chat/favicon.ico' }
      ],
      siteConfig: {},
      // 粒子动画相关
      particles: [],
      animationId: null
    }
  },
  computed: {
    /** 计算密码强度 (1=弱, 2=中, 3=强) */
    passwordStrength() {
      const pwd = this.registerValues.password
      if (!pwd) return 0
      let score = 0
      if (pwd.length >= 6) score++
      if (/[A-Z]/.test(pwd) && /[a-z]/.test(pwd)) score++
      if (/[0-9]/.test(pwd) && /[^A-Za-z0-9]/.test(pwd)) score++
      return score
    },
    strengthLabel() {
      const labels = { 0: '', 1: '弱', 2: '中', 3: '强' }
      return labels[this.passwordStrength] || ''
    },
    isAgentLogin() {
      return this.$route.path === '/agents/login'
    },
    welcomeTitle() {
      if (this.isAgentLogin) return '代理后台登录'
      return this.mode === 'login' ? '欢迎回来' : '创建账户'
    },
    welcomeSubtitle() {
      if (this.isAgentLogin) return '登录代理账号以管理用户、套餐和站点配置'
      return this.mode === 'login' ? '登录您的账户以继续使用服务' : '注册新账户，开始您的 AI 之旅'
    }
  },
  beforeCreate() {
    this.loginForm = this.$form.createForm(this, { name: 'login' })
    this.registerForm = this.$form.createForm(this, { name: 'register' })
    this.forgotForm = this.$form.createForm(this, { name: 'forgot_password' })
  },
  mounted() {
    // 根据路由判断初始模式
    if (this.$route.path === '/register' && !this.isAgentLogin) {
      this.mode = 'register'
    } else {
      this.mode = 'login'
    }
    this.fetchSiteConfig()
    this.initParticles()
  },
  beforeDestroy() {
    if (this.animationId) {
      cancelAnimationFrame(this.animationId)
    }
  },
  methods: {
    /** 切换登录/注册模式 */
    switchMode(newMode) {
      if (this.isAgentLogin && newMode !== 'login') return
      if (this.mode === newMode) return
      this.mode = newMode
      this.focusedField = null
      // 更新路由路径（不刷新页面）
      const targetPath = newMode === 'login' ? '/login' : '/register'
      if (this.$route.path !== targetPath) {
        this.$router.replace(targetPath)
      }
    },
    handleBlur(fieldKey, formType, fieldName) {
      const form = formType === 'login' ? this.loginForm : this.registerForm
      const values = formType === 'login' ? this.loginValues : this.registerValues
      const val = form.getFieldValue(fieldName)
      values[fieldName] = val || ''
      if (this.focusedField === fieldKey) {
        this.focusedField = null
      }
    },
    handleConfirmBlur(e) {
      const value = e.target.value
      this.confirmDirty = this.confirmDirty || !!value
      this.registerValues.confirmPassword = value || ''
      if (this.focusedField === 'r-confirm') {
        this.focusedField = null
      }
    },
    handlePasswordChange(e) {
      this.registerValues.password = e.target.value
    },
    validateToNextPassword(rule, value, callback) {
      if (value && this.confirmDirty) {
        this.registerForm.validateFields(['confirmPassword'], { force: true })
      }
      callback()
    },
    compareToFirstPassword(rule, value, callback) {
      if (value && value !== this.registerForm.getFieldValue('password')) {
        callback('两次输入的密码不一致')
      } else {
        callback()
      }
    },
    handleForgotPasswordChange(e) {
      this.forgotValues.newPassword = e.target.value
    },
    handleForgotConfirmBlur(e) {
      const value = e.target.value
      this.forgotConfirmDirty = this.forgotConfirmDirty || !!value
      this.forgotValues.confirmPassword = value || ''
    },
    validateForgotPassword(rule, value, callback) {
      if (value && this.forgotConfirmDirty) {
        this.forgotForm.validateFields(['confirm_password'], { force: true })
      }
      callback()
    },
    compareForgotPassword(rule, value, callback) {
      if (value && value !== this.forgotForm.getFieldValue('new_password')) {
        callback('两次输入的密码不一致')
      } else {
        callback()
      }
    },
    openForgotPassword() {
      this.forgotVisible = true
      this.forgotStep = 'identity'
      this.$nextTick(() => {
        const username = this.loginForm.getFieldValue('username') || this.loginValues.username || ''
        this.forgotForm.setFieldsValue({ username })
      })
    },
    resetForgotIdentityStep() {
      this.forgotStep = 'identity'
      this.forgotConfirmDirty = false
      this.forgotValues.newPassword = ''
      this.forgotValues.confirmPassword = ''
      this.$nextTick(() => {
        this.forgotForm.setFieldsValue({
          new_password: undefined,
          confirm_password: undefined
        })
      })
    },
    closeForgotPassword() {
      if (this.forgotLoading) return
      this.forgotVisible = false
      this.forgotStep = 'identity'
      this.forgotConfirmDirty = false
      this.forgotValues = { username: '', email: '', newPassword: '', confirmPassword: '' }
      this.forgotForm.resetFields()
    },
    handleForgotPassword() {
      this.forgotForm.validateFields((err, values) => {
        if (err) return
        this.forgotLoading = true
        const action = this.forgotStep === 'identity'
          ? verifyForgotPasswordIdentity({
            username: values.username,
            email: values.email
          })
          : forgotPassword({
            username: values.username,
            email: values.email,
            new_password: values.new_password
          })
        action
          .then(() => {
            if (this.forgotStep === 'identity') {
              this.forgotValues.username = values.username || ''
              this.forgotValues.email = values.email || ''
              this.forgotStep = 'reset'
              this.$message.success('身份校验通过，请输入新密码')
              return
            }
            this.$message.success('密码已重置，请使用新密码登录')
            this.loginForm.setFieldsValue({
              username: values.username,
              password: ''
            })
            this.loginValues.username = values.username || ''
            this.loginValues.password = ''
            this.forgotVisible = false
            this.forgotStep = 'identity'
            this.forgotConfirmDirty = false
            this.forgotValues = { username: '', email: '', newPassword: '', confirmPassword: '' }
            this.forgotForm.resetFields()
            this.switchMode('login')
          })
          .catch((error) => {
            const msg =
              (error.response && error.response.data && error.response.data.message) ||
              error.message ||
              (this.forgotStep === 'identity' ? '身份校验失败，请重试' : '密码找回失败，请重试')
            this.$message.error(msg)
          })
          .finally(() => {
            this.forgotLoading = false
          })
      })
    },

    /** 登录提交 */
    handleLogin(e) {
      e.preventDefault()
      this.loginForm.validateFields((err, values) => {
        if (err) return
        this.loading = true
        this.$store
          .dispatch('login', values)
          .then(() => {
            const user = this.$store.state.user
            if (this.isAgentLogin && (!user || user.role !== 'agent')) {
              this.$store.dispatch('logout')
              this.$message.error('当前账号不是代理账号')
              return
            }
            this.$message.success('登录成功')
            const redirect = this.$route.query.redirect
            if (user && user.role === 'admin') {
              this.$router.push(redirect || '/admin/dashboard')
            } else if (user && user.role === 'agent') {
              this.$router.push(redirect || '/agent/workbench')
            } else {
              this.$router.push(redirect || '/user/dashboard')
            }
          })
          .catch((error) => {
            const msg =
              (error.response && error.response.data && error.response.data.message) ||
              error.message ||
              '登录失败，请重试'
            this.$message.error(msg)
          })
          .finally(() => {
            this.loading = false
          })
      })
    },

    /** 注册提交 */
    handleRegister(e) {
      e.preventDefault()
      this.registerForm.validateFields((err, values) => {
        if (err) return
        this.loading = true
        const payload = {
          username: values.username,
          email: values.email,
          password: values.password
        }
        this.$store
          .dispatch('register', payload)
          .then(() => {
            this.$message.success('注册成功！请登录。')
            this.switchMode('login')
          })
          .catch((error) => {
            const msg =
              (error.response && error.response.data && error.response.data.message) ||
              error.message ||
              '注册失败，请重试'
            this.$message.error(msg)
          })
          .finally(() => {
            this.loading = false
          })
      })
    },
    async fetchSiteConfig() {
      try {
        const res = await getPublicSiteConfig()
        this.siteConfig = res.data || {}
      } catch (e) {
        this.siteConfig = {}
      }
    },

    /** 计算 Logo 位置样式 */
    getLogoStyle(index) {
      const total = this.aiLogos.length
      const angle = (index / total) * 360
      return {
        '--angle': `${angle}deg`,
        '--delay': `${index * 0.5}s`
      }
    },

    /** 粒子动画初始化 */
    initParticles() {
      const canvas = this.$refs.particleCanvas
      if (!canvas) return
      const ctx = canvas.getContext('2d')
      const resize = () => {
        canvas.width = window.innerWidth
        canvas.height = window.innerHeight
      }
      resize()
      window.addEventListener('resize', resize)

      // 创建粒子
      const count = 60
      this.particles = Array.from({ length: count }, () => ({
        x: Math.random() * canvas.width,
        y: Math.random() * canvas.height,
        vx: (Math.random() - 0.5) * 0.4,
        vy: (Math.random() - 0.5) * 0.4,
        r: Math.random() * 2 + 0.5,
        alpha: Math.random() * 0.5 + 0.1
      }))

      const animate = () => {
        ctx.clearRect(0, 0, canvas.width, canvas.height)
        this.particles.forEach(p => {
          p.x += p.vx
          p.y += p.vy
          // 边界反弹
          if (p.x < 0 || p.x > canvas.width) p.vx *= -1
          if (p.y < 0 || p.y > canvas.height) p.vy *= -1
          ctx.beginPath()
          ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2)
          ctx.fillStyle = `rgba(168, 184, 255, ${p.alpha})`
          ctx.fill()
        })

        // 画连线
        for (let i = 0; i < this.particles.length; i++) {
          for (let j = i + 1; j < this.particles.length; j++) {
            const a = this.particles[i]
            const b = this.particles[j]
            const dist = Math.hypot(a.x - b.x, a.y - b.y)
            if (dist < 120) {
              ctx.beginPath()
              ctx.moveTo(a.x, a.y)
              ctx.lineTo(b.x, b.y)
              ctx.strokeStyle = `rgba(102, 126, 234, ${0.12 * (1 - dist / 120)})`
              ctx.lineWidth = 0.5
              ctx.stroke()
            }
          }
        }
        this.animationId = requestAnimationFrame(animate)
      }
      animate()
    }
  }
}
</script>

<style lang="less" scoped>

// Color Variables - Light Tech Theme
@primary-color: #2563eb;
@primary-light: #3b82f6;
@primary-dark: #1e40af;
@bg-primary: #f8fafc;
@bg-secondary: #ffffff;
@border-color: #e2e8f0;
@text-primary: #0f172a;
@text-secondary: #475569;
@text-tertiary: #94a3b8;

/* =============================================
   全局页面
   ============================================= */
.auth-page {
  width: 100%;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #f8fafc 0%, #e0e7ff 100%);
  overflow: hidden;
  position: relative;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'PingFang SC', 'Hiragino Sans GB', 'Microsoft YaHei', sans-serif;
}

/* =============================================
   动态背景层
   ============================================= */
.bg-layer {
  position: absolute;
  inset: 0;
  z-index: 0;
  overflow: hidden;
}

.floating-shapes {
  position: absolute;
  inset: 0;
}

.shape {
  position: absolute;
  border-radius: 50%;
  background: linear-gradient(135deg, rgba(37, 99, 235, 0.1), rgba(139, 92, 246, 0.1));
  animation-timing-function: ease-in-out;
  animation-iteration-count: infinite;
  animation-direction: alternate;
}

/* =============================================
   AI 模型 Logo 底部滚动条
   ============================================= */
.ai-logos-bottom {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 100px;
  z-index: 1;
  overflow: hidden;
  background: linear-gradient(to top, rgba(255, 255, 255, 0.8), transparent);
  backdrop-filter: blur(10px);
}

.logos-track {
  display: flex;
  gap: 24px;
  animation: scroll-logos 30s linear infinite;
  padding: 20px 0;
}

.logo-item {
  flex-shrink: 0;
  animation: logo-fade-in 0.6s ease-out both;
  animation-delay: calc(var(--index, 0) * 0.1s);
}

.logo-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  padding: 12px 16px;
  background: white;
  border: 2px solid @border-color;
  border-radius: 16px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  transition: all 0.3s ease;

  &:hover {
    transform: translateY(-8px);
    box-shadow: 0 12px 24px rgba(37, 99, 235, 0.15);
    border-color: @primary-light;
  }
}

.logo-icon {
  width: 48px;
  height: 48px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 16px;
  font-weight: 700;
  color: white;
  text-transform: uppercase;
  letter-spacing: -0.5px;
}

.logo-name {
  font-size: 12px;
  font-weight: 600;
  color: @text-secondary;
  white-space: nowrap;
}

.shape-1 {
  width: 300px;
  height: 300px;
  top: -10%;
  left: -5%;
  animation: float-1 20s;
}

.shape-2 {
  width: 200px;
  height: 200px;
  top: 60%;
  right: -5%;
  animation: float-2 15s;
}

.shape-3 {
  width: 150px;
  height: 150px;
  bottom: 10%;
  left: 20%;
  animation: float-3 18s;
}

.shape-4 {
  width: 250px;
  height: 250px;
  top: 20%;
  right: 15%;
  animation: float-4 22s;
}

.shape-5 {
  width: 180px;
  height: 180px;
  bottom: 20%;
  right: 30%;
  animation: float-5 16s;
}

.grid-overlay {
  position: absolute;
  inset: 0;
  background-image:
    linear-gradient(rgba(37, 99, 235, 0.03) 1px, transparent 1px),
    linear-gradient(90deg, rgba(37, 99, 235, 0.03) 1px, transparent 1px);
  background-size: 50px 50px;
  animation: grid-move 30s linear infinite;
}

/* =============================================
   主容器 — 左右布局
   ============================================= */
.auth-container {
  position: relative;
  z-index: 2;
  display: flex;
  width: 800px;
  max-width: 90%;
  margin-top: -125px;
  margin-bottom: 0;
  background: @bg-secondary;
  border: 1px solid @border-color;
  border-radius: 24px;
  box-shadow:
    0 20px 60px rgba(0, 0, 0, 0.08),
    0 0 0 1px rgba(255, 255, 255, 0.5) inset;
  overflow: hidden;
  animation: container-enter 0.8s cubic-bezier(0.16, 1, 0.3, 1) both;
  backdrop-filter: blur(10px);
  transition: margin-top 0.3s ease;

  &.register-mode {
    margin-top: -50px;
  }
}

/* =============================================
   右侧：表单区域
   ============================================= */
.form-section {
  flex: 0 0 400px;
  display: flex;
  flex-direction: column;
  order: 2;
}

/* =============================================
   顶部品牌区
   ============================================= */
.brand-header {
  padding: 16px 20px 12px;
  background: linear-gradient(135deg, #f1f5f9 0%, #e0e7ff 100%);
  border-bottom: 1px solid @border-color;
  position: relative;
  overflow: hidden;

  &::before {
    content: '';
    position: absolute;
    top: -50%;
    right: -20%;
    width: 300px;
    height: 300px;
    background: radial-gradient(circle, rgba(37, 99, 235, 0.08), transparent 70%);
    animation: pulse 8s ease-in-out infinite;
  }
}

.logo-section {
  display: flex;
  align-items: center;
  gap: 14px;
  position: relative;
  z-index: 1;
}

.logo-ring {
  width: 48px;
  height: 48px;
  border-radius: 14px;
  background: linear-gradient(135deg, @primary-color, @primary-light);
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 6px 20px rgba(37, 99, 235, 0.25);

  /deep/ .anticon {
    font-size: 24px;
    color: white;
  }
}

.brand-info {
  flex: 1;
}

.brand-title {
  font-size: 18px;
  font-weight: 700;
  color: @text-primary;
  margin: 0 0 2px;
  letter-spacing: -0.5px;
}

.brand-subtitle {
  font-size: 12px;
  color: @text-tertiary;
  margin: 0;
}

/* =============================================
   表单面板
   ============================================= */
.form-panel {
  padding: 8px 17px;
  flex: 1;
}

.form-wrapper {
  width: 100%;
}

/* =============================================
   左侧：特性展示区域
   ============================================= */
.features-section {
  flex: 1;
  padding: 16px 22px;
  background: linear-gradient(135deg, #fafbff 0%, #f0f4ff 100%);
  display: flex;
  flex-direction: column;
  justify-content: center;
  order: 1;
}

.features-header {
  margin-bottom: 12px;
}

.feature-list {
  display: flex;
  flex-direction: column;
  gap: 20px;
  flex: 1;
  margin-top: 20px;
}

.features-title {
  font-size: 20px;
  font-weight: 700;
  color: @text-primary;
  margin: 0 0 8px;
  letter-spacing: -0.5px;
}

.features-subtitle {
  font-size: 13px;
  color: @text-tertiary;
  margin: 0;
}

.feature-list {
  display: flex;
  flex-direction: column;
  gap: 20px;
  flex: 1;
}

.feature-item {
  display: flex;
  gap: 16px;
  padding: 18px;
  background: white;
  border: 1px solid @border-color;
  border-radius: 14px;
  transition: all 0.3s ease;

  &:hover {
    transform: translateX(4px);
    box-shadow: 0 8px 20px rgba(37, 99, 235, 0.08);
    border-color: @primary-light;
  }
}

.feature-icon {
  flex-shrink: 0;
  width: 44px;
  height: 44px;
  border-radius: 12px;
  background: linear-gradient(135deg, @primary-color, @primary-light);
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-size: 18px;
}

.feature-content {
  flex: 1;

  h4 {
    font-size: 14px;
    font-weight: 700;
    color: @text-primary;
    margin: 0 0 4px;
  }

  p {
    font-size: 12px;
    color: @text-secondary;
    margin: 0;
    line-height: 1.6;
  }
}

.features-footer {
  display: flex;
  gap: 16px;
  margin-top: 24px;
  padding-top: 24px;
  border-top: 1px solid @border-color;
}

.stat-badge {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 16px;
  background: white;
  border: 1px solid @border-color;
  border-radius: 12px;

  .stat-number {
    font-size: 24px;
    font-weight: 700;
    color: @primary-color;
    margin-bottom: 4px;
    font-family: 'SF Mono', monospace;
  }

  .stat-label {
    font-size: 11px;
    color: @text-tertiary;
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 0.5px;
  }
}

/* =============================================
   AI 模型 Logo 底部展示
   ============================================= */
.ai-logos-bottom {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 140px;
  z-index: 1;
  overflow: hidden;
  background: transparent;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 12px;
}

.logos-container {
  display: flex;
  gap: 16px;
  padding: 0 20px;
  justify-content: center;
  flex-wrap: wrap;
  max-width: 1200px;
  align-content: center;
}

.copyright-text {
  text-align: center;
  margin-top: 16px;
  font-size: 11px;
  color: @text-tertiary;
  opacity: 0.6;
  letter-spacing: 0.5px;
  font-weight: 500;
}

.logo-item {
  animation: logo-float-up 3s ease-in-out infinite;
}

@keyframes logo-float-up {
  0%, 100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-12px);
  }
}

.logo-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  padding: 10px 14px;
  background: white;
  border: 2px solid @border-color;
  border-radius: 14px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.06);
  transition: all 0.3s ease;
  cursor: pointer;

  &:hover {
    transform: translateY(-8px);
    box-shadow: 0 12px 24px rgba(37, 99, 235, 0.15);
    border-color: @primary-light;
  }
}

.logo-icon {
  width: 40px;
  height: 40px;
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 14px;
  font-weight: 700;
  color: white;
  text-transform: uppercase;
  letter-spacing: -0.5px;
  overflow: hidden;

  .logo-image {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
}

.logo-name {
  font-size: 11px;
  font-weight: 600;
  color: @text-secondary;
  white-space: nowrap;
}

/* =============================================
   切换标签
   ============================================= */
.auth-tabs {
  position: relative;
  display: flex;
  margin-bottom: 32px;
  background: @bg-primary;
  border-radius: 12px;
  padding: 4px;
  border: 1px solid @border-color;
}

.tab-btn {
  flex: 1;
  padding: 10px 0;
  background: none;
  border: none;
  color: @text-tertiary;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  position: relative;
  z-index: 1;
  transition: color 0.3s ease;
  outline: none;

  &.active {
    color: @text-primary;
  }

  &:hover:not(.active) {
    color: @text-secondary;
  }
}

.tab-indicator {
  position: absolute;
  top: 4px;
  left: 4px;
  width: calc(50% - 4px);
  height: calc(100% - 8px);
  background: white;
  border-radius: 9px;
  transition: transform 0.35s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);

  &.at-register {
    transform: translateX(100%);
  }
}

/* =============================================
   欢迎文字
   ============================================= */
.welcome-text {
  margin-bottom: 28px;

  h2 {
    font-size: 22px;
    font-weight: 700;
    color: @text-primary;
    margin: 0 0 8px;
  }

  p {
    font-size: 13px;
    color: @text-tertiary;
    margin: 0;
  }
}

/* =============================================
   表单字段
   ============================================= */
.auth-form {
  /deep/ .ant-form-item {
    margin-bottom: 0;
  }
}

.input-group {
  position: relative;
  margin-bottom: 22px;

  > label {
    position: absolute;
    left: 40px;
    top: 50%;
    transform: translateY(-50%);
    font-size: 14px;
    color: @text-tertiary;
    pointer-events: none;
    transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
    z-index: 2;
  }

  &.focused > label,
  &.filled > label {
    top: -18px;
    left: 0;
    transform: none;
    font-size: 12px;
    color: @primary-color;
    letter-spacing: 0.5px;
  }

  /deep/ .ant-input,
  /deep/ .ant-input-password .ant-input {
    background: @bg-primary;
    border: 1px solid @border-color;
    border-radius: 12px;
    color: @text-primary;
    height: 48px;
    padding: 0 14px 0 40px;
    font-size: 14px;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);

    &::placeholder {
      color: transparent;
    }

    &:hover {
      border-color: @primary-light;
      background: white;
    }

    &:focus,
    &.ant-input-focused {
      background: white;
      border-color: @primary-color;
      box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.1);
    }
  }

  .input-icon {
    color: @text-tertiary;
    font-size: 16px;
    transition: color 0.3s;
  }

  &.focused .input-icon {
    color: @primary-color;
  }

  /deep/ .ant-input-prefix {
    left: 14px;
  }

  /deep/ .ant-input-suffix {
    color: @text-tertiary;
    .anticon {
      color: @text-tertiary;
    }
  }

  /deep/ .ant-form-explain {
    color: #ef4444;
    font-size: 12px;
    margin-top: 6px;
    padding-left: 2px;
  }
}

/* =============================================
   密码强度指示器
   ============================================= */
.password-strength {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-top: 8px;
  padding-left: 2px;
}

.strength-bars {
  display: flex;
  gap: 4px;
}

.bar {
  width: 40px;
  height: 3px;
  border-radius: 2px;
  background: @border-color;
  transition: all 0.3s ease;

  &.active.weak {
    background: #ef4444;
  }
  &.active.medium {
    background: #f59e0b;
  }
  &.active.strong {
    background: #10b981;
  }
}

.strength-text {
  font-size: 11px;
  font-weight: 500;

  &.weak { color: #ef4444; }
  &.medium { color: #f59e0b; }
  &.strong { color: #10b981; }
}

/* =============================================
   提交按钮
   ============================================= */
.submit-btn {
  margin-top: 4px;
  height: 48px !important;
  border-radius: 12px !important;
  font-size: 15px !important;
  font-weight: 600 !important;
  border: none !important;
  background: linear-gradient(135deg, @primary-color 0%, @primary-dark 100%) !important;
  position: relative;
  overflow: hidden;
  transition: all 0.35s cubic-bezier(0.4, 0, 0.2, 1) !important;

  .btn-content {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
    letter-spacing: 2px;
  }

  .btn-arrow {
    font-size: 14px;
    transition: transform 0.3s ease;
  }

  &::before {
    content: '';
    position: absolute;
    inset: 0;
    background: linear-gradient(135deg, rgba(255,255,255,0.2), transparent 60%);
    opacity: 0;
    transition: opacity 0.35s;
  }

  &:hover::before {
    opacity: 1;
  }

  &:hover {
    transform: translateY(-2px) !important;
    box-shadow: 0 12px 32px rgba(37, 99, 235, 0.3) !important;

    .btn-arrow {
      transform: translateX(4px);
    }
  }

  &:active {
    transform: translateY(0) !important;
    box-shadow: 0 4px 12px rgba(37, 99, 235, 0.2) !important;
  }
}

/* =============================================
   登录辅助操作
   ============================================= */
.login-actions {
  display: flex;
  justify-content: flex-end;
  margin: -4px 0 16px;

  a {
    color: @primary-color;
    font-size: 13px;
    text-decoration: none;
    transition: all 0.2s ease;

    &:hover {
      color: @primary-dark;
    }
  }
}

/* =============================================
   表单底部
   ============================================= */
.form-footer {
  text-align: center;
  margin-top: 24px;
  font-size: 13px;
  color: @text-tertiary;

  a {
    color: @primary-color;
    font-weight: 500;
    margin-left: 4px;
    cursor: pointer;
    transition: all 0.2s;
    text-decoration: none;

    &:hover {
      color: @primary-dark;
    }
  }
}

/* =============================================
   模态框样式
   ============================================= */
/deep/ .forgot-password-modal {
  .ant-modal-content {
    background: white;
    border: 1px solid @border-color;
    border-radius: 16px;
    box-shadow: 0 20px 60px rgba(0, 0, 0, 0.15);
  }

  .ant-modal-header {
    background: transparent;
    border-bottom: 1px solid @border-color;
    border-radius: 16px 16px 0 0;
  }

  .ant-modal-title,
  .ant-modal-close,
  .ant-modal-close-x {
    color: @text-primary;
  }

  .ant-modal-body {
    padding-top: 20px;
  }

  .ant-modal-footer {
    border-top: 1px solid @border-color;
  }

  .ant-form-item-label > label {
    color: @text-secondary;
  }

  .ant-input,
  .ant-input-password .ant-input {
    background: @bg-primary;
    border: 1px solid @border-color;
    border-radius: 12px;
    color: @text-primary;
  }

  .ant-input:hover,
  .ant-input-password .ant-input:hover,
  .ant-input:focus,
  .ant-input-password .ant-input:focus,
  .ant-input-focused {
    border-color: @primary-color;
    box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.1);
  }

  .ant-input-affix-wrapper .ant-input-prefix,
  .ant-input-prefix,
  .ant-input-password-icon,
  .anticon {
    color: @text-tertiary;
  }

  .ant-form-explain {
    color: #ef4444;
  }
}

.forgot-step-tip {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  margin-bottom: 18px;
  color: @text-secondary;
  font-size: 13px;
  line-height: 1.6;

  a {
    flex-shrink: 0;
    color: @primary-color;
    text-decoration: none;
  }
}

/* =============================================
   过渡动画
   ============================================= */
.text-fade-enter-active,
.text-fade-leave-active {
  transition: all 0.25s ease;
}
.text-fade-enter {
  opacity: 0;
  transform: translateY(-8px);
}
.text-fade-leave-to {
  opacity: 0;
  transform: translateY(8px);
}

.form-slide-enter-active {
  transition: all 0.35s cubic-bezier(0.16, 1, 0.3, 1);
}
.form-slide-leave-active {
  transition: all 0.2s ease-in;
}
.form-slide-enter {
  opacity: 0;
  transform: translateX(24px);
}
.form-slide-leave-to {
  opacity: 0;
  transform: translateX(-24px);
}

/* =============================================
   关键帧动画
   ============================================= */
@keyframes container-enter {
  from {
    opacity: 0;
    transform: translateY(30px) scale(0.95);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

@keyframes scroll-logos {
  from {
    transform: translateX(0);
  }
  to {
    transform: translateX(calc(-100% - 24px));
  }
}

@keyframes logo-fade-in {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes float-1 {
  0%, 100% { transform: translate(0, 0) rotate(0deg); }
  50% { transform: translate(30px, -30px) rotate(180deg); }
}

@keyframes float-2 {
  0%, 100% { transform: translate(0, 0) rotate(0deg); }
  50% { transform: translate(-40px, 20px) rotate(-180deg); }
}

@keyframes float-3 {
  0%, 100% { transform: translate(0, 0) scale(1); }
  50% { transform: translate(20px, -40px) scale(1.1); }
}

@keyframes float-4 {
  0%, 100% { transform: translate(0, 0) rotate(0deg); }
  50% { transform: translate(-30px, 30px) rotate(90deg); }
}

@keyframes float-5 {
  0%, 100% { transform: translate(0, 0) scale(1); }
  50% { transform: translate(40px, -20px) scale(0.9); }
}

@keyframes grid-move {
  0% { background-position: 0 0; }
  100% { background-position: 50px 50px; }
}

@keyframes pulse {
  0%, 100% { opacity: 0.3; transform: scale(1); }
  50% { opacity: 0.5; transform: scale(1.1); }
}

@keyframes logo-float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-8px); }
}

/* =============================================
   响应式适配
   ============================================= */
@media (max-width: 1100px) {
  .auth-container {
    flex-direction: column;
    width: 95%;
    max-width: 600px;
    margin-bottom: 120px;
  }

  .form-section {
    flex: none;
  }

  .features-section {
    padding: 32px;
  }

  .feature-list {
    gap: 16px;
  }
}

@media (max-width: 600px) {
  .auth-container {
    width: 95%;
    border-radius: 20px;
  }

  .brand-header {
    padding: 24px 20px 20px;
  }

  .logo-ring {
    width: 40px;
    height: 40px;

    /deep/ .anticon {
      font-size: 20px;
    }
  }

  .brand-title {
    font-size: 16px;
  }

  .brand-subtitle {
    font-size: 11px;
  }

  .form-panel {
    padding: 24px 20px;
  }

  .features-section {
    padding: 24px 20px;
  }

  .features-title {
    font-size: 16px;
    margin-bottom: 16px;
  }

  .feature-item {
    padding: 16px;
    gap: 12px;
  }

  .feature-icon {
    width: 40px;
    height: 40px;
    font-size: 18px;
  }

  .ai-logos-bottom {
    height: 80px;
  }

  .logos-track {
    padding: 15px 0;
  }

  .logo-card {
    padding: 10px 12px;
  }

  .logo-icon {
    width: 40px;
    height: 40px;
    font-size: 14px;
  }

  .logo-name {
    font-size: 11px;
  }
}
</style>
