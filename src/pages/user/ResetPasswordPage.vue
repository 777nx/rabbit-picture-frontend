<template>
  <div id="resetPasswordPage">
    <h2 class="title">兔子云图库 - 忘记密码</h2>
    <div class="desc">企业级智能协同云图库</div>
    <a-form :model="formState" name="basic" autocomplete="off" @finish="handleSubmit">
      <a-form-item name="userEmail" :rules="[
        { required: true, message: '请输入邮箱' },
        { pattern: '^[a-zA-Z0-9_-]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$', message: '请输入正确的邮箱格式' },
      ]">
        <a-input v-model:value="formState.userEmail" placeholder="请输入邮箱" />
      </a-form-item>
      <a-form-item
        name="code"
        :rules="[
          { required: true, message: '请输入验证码' },
          { min: 6, message: '验证码长度不能小于6位' },
        ]"
      >
        <a-input v-model:value="formState.code" placeholder="请输入验证码">
          <template #suffix>
            <a-button type="primary" size="small" :disabled="countdown > 0" :loading="loading" @click="doSendEmail">
              {{ buttonText }}
            </a-button>
          </template>
        </a-input>
      </a-form-item>
      <a-form-item
        name="password"
        :rules="[
          { required: true, message: '请输入密码' },
          { min: 8, message: '密码长度不能小于8位' },
          { max: 16, message: '密码长度不能大于16位' }
        ]"
      >
        <a-input-password
          v-model:value="formState.password"
          placeholder="请输入密码"
          autocomplete="off"
        />
      </a-form-item>
      <a-form-item
        name="checkPassword"
        :rules="[
          { required: true, message: '请输入确认密码' },
          { min: 8, message: '确认密码长度不能小于8位' },
          { max: 16, message: '确认密码长度不能大于16位' }
        ]"
      >
        <a-input-password
          v-model:value="formState.checkPassword"
          placeholder="请输入确认密码"
          autocomplete="off"
        />
      </a-form-item>
      <div class="tips">
        <a-row justify="space-between">
          <a-col>
            已有账号？
            <router-link to="/user/login">去登录</router-link>
          </a-col>
          <a-col>
            没有账号？
            <router-link to="/user/register">去注册</router-link>
          </a-col>
        </a-row>
      </div>
      <a-form-item>
        <a-button type="primary" html-type="submit" style="width: 100%">确认提交</a-button>
      </a-form-item>
    </a-form>
  </div>
</template>
<script lang="ts" setup>
import { computed, onBeforeUnmount, reactive, ref } from 'vue'
import {
  resetPasswordUsingPost,
  sendEmailUsingGet,
} from '@/api/userController.ts'
import { message } from 'ant-design-vue'
import router from '@/router'

// 用于接受表单输入的值
const formState = reactive<API.UserRegisterRequest>({
  userEmail: '',
  code: '',
  password: '',
  checkPassword: '',
})

const loading = ref<boolean>(false)
const countdown = ref(0)
let timer = null

// 按钮文本
const buttonText = computed(() => {
  return countdown.value > 0 ? `${countdown.value}秒后重试` : '发送验证码'
})

/**
 * 发送验证码
 */
const doSendEmail = async () => {
  loading.value = true
  // 校验
  if (!formState.userEmail) {
    loading.value = false
    return
  }
  const res = await sendEmailUsingGet({
    email: formState.userEmail
  })
  if (res.data.code === 0) {
    message.success('验证码已发送，请注意查收')
    countdown.value = 60
    timer = setInterval(() => {
      countdown.value--
      if (countdown.value <= 0) {
        clearInterval(timer)
      }
    }, 1000)
  } else {
    message.error('发送验证码失败：' + res.data.message)
  }
  loading.value = false
}

// 组件卸载前清除定时器
onBeforeUnmount(() => {
  if (timer) clearInterval(timer)
})

/**
 * 提交表单
 * @param values
 */
const handleSubmit = async (values: any) => {
  // 校验两次输入的密码是否一致
  if (values.password !== values.checkPassword) {
    message.error('两次输入的密码不一致')
    return
  }
  const res = await resetPasswordUsingPost(values)
  // 重置密码成功，跳转到登录页面
  if (res.data.code === 0 && res.data.data) {
    message.success('重置密码成功')
    router.push({
      path: '/user/login',
      replace: true,
    })
  } else {
    message.error('重置密码失败：' + res.data.message)
  }
}
</script>

<style scoped>
#resetPasswordPage {
  max-width: 360px;
  margin: 0 auto;
}

.title {
  text-align: center;
  margin-bottom: 16px;
}

.desc {
  text-align: center;
  color: #bbb;
  margin-bottom: 16px;
}

.tips {
  color: #bbb;
  text-align: right;
  font-size: 13px;
  margin-bottom: 16px;
}
</style>
