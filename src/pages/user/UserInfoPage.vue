<template>
  <div id="userInfoPage">
    <a-card class="info" title="个人信息">
      <a-row>
        <a-col :span="12">
          <a-flex vertical justify="center" align="center" style="height: 100%">
            <div>
              <img class="avatar" v-if="avatar" :src="avatar" alt="avatar" />
              <a-avatar v-else :size="128">
                {{ loginUserStore.loginUser.userName?.substring(0, 1) }}
              </a-avatar>
            </div>
            <div style="margin-bottom: 16px" />
            <a-button type="primary" html-type="submit" style="width: 30%" @click="triggerFileInput">修改头像</a-button>
            <input
              type="file"
              ref="fileInput"
              style="display: none"
              accept="image/jpeg,image/png,image/jpg,image/webp"
              @change="handleFileChange"
            >
          </a-flex>
        </a-col>
        <a-col :span="12">
          <a-form :model="loginUser" name="basic" autocomplete="off" @finish="handleSubmit">
            <a-form-item name="id">
              用户 id：
              <a-input v-model:value="loginUser.id" placeholder="请输入用户 id" disabled />
            </a-form-item>
            <a-form-item name="userAccount">
              账号：
              <a-input v-model:value="loginUser.userAccount" placeholder="请输入账号" disabled />
            </a-form-item>
            <a-form-item name="userEmail">
              邮箱：
              <a-input v-model:value="loginUser.userEmail" placeholder="请输入邮箱" disabled />
            </a-form-item>
            <a-form-item name="userName">
              用户名：
              <a-input v-model:value="loginUser.userName" placeholder="请输入用户名" />
            </a-form-item>
            <a-form-item name="userProfile">
              简介：
              <a-textarea
                v-model:value="loginUser.userProfile"
                placeholder="请输入简介"
                :auto-size="{ minRows: 2, maxRows: 5 }"
              />
            </a-form-item>
            <a-form-item>
              <a-button type="primary" html-type="submit" style="width: 100%">提交</a-button>
            </a-form-item>
          </a-form>
        </a-col>
      </a-row>
    </a-card>
  </div>
</template>

<script setup lang="ts">
import { useLoginUserStore } from '@/stores/useLoginUserStore.ts'
import { updateUserAvatarUsingPost, updateUserUsingPost } from '@/api/userController.ts'
import { message } from 'ant-design-vue'
import { useRouter } from 'vue-router'
import { ref } from 'vue'

const loginUserStore = useLoginUserStore()
const loginUser = loginUserStore.loginUser
const avatar = ref(loginUser.userAvatar)

const router = useRouter()

// 添加文件输入框的引用
const fileInput = ref<HTMLInputElement | null>(null)

// 点击按钮触发文件选择
const triggerFileInput = () => {
  fileInput.value?.click()
}

const loading = ref<boolean>(false)

/**
 * 上传头像
 * @param e
 */
const handleFileChange = async (e: Event) => {
  const file = (e.target as HTMLInputElement).files?.[0]
  if (!file) return

  // 上传前的校验
  if (!beforeUpload(file)) {
    message.error('文件不符合要求')
    return
  }

  // 调用上传逻辑
  try {
    loading.value = true
    const res = await updateUserAvatarUsingPost({
      id: loginUser.id
    }, {}, file)

    if (res.data.code === 0 && res.data.data) {
      message.success('头像上传成功')
      avatar.value = res.data.data
    } else {
      message.error('头像上传失败: ' + res.data.message)
    }
  } catch (error) {
    message.error('上传失败: ' + (error as Error).message)
  } finally {
    loading.value = false
    // 重置input以允许再次选择相同文件
    if (fileInput.value) fileInput.value.value = ''
  }
}

/**
 * 上传前的校验
 * @param file
 */
const beforeUpload = (file: File) => {
  const isImage = ['image/jpeg', 'image/png', 'image/jpg', 'image/webp'].includes(file.type)
  const isLt2M = file.size / 1024 / 1024 < 2
  return isImage && isLt2M
}

/**
 * 提交表单
 * @param values
 */
const handleSubmit = async (values: any) => {
  const res = await updateUserUsingPost({
    ...values,
    id: loginUser.id,
  })
  if (res.data.code === 0) {
    // 修改成功重新获取下登录信息，返回到上一页面
    await loginUserStore.fetchLoginUser()
    message.success("修改成功")
    router.back()
  } else {
    message.error("修改失败：" + res.data.message)
  }
}
</script>

<style scoped>
#userInfoPage {
  max-width: 1080px;
  margin: 0 auto;
}

#userInfoPage :deep(.ant-avatar-string) {
  font-size: 64px;
}

#userInfoPage .avatar {
  width: 128px;
  height: 128px;
  border-radius: 50%;
  object-fit: cover;
}
</style>
