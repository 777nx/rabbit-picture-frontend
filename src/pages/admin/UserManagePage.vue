<template>
  <div id="userManagePage">
    <!-- 搜索表单 -->
    <a-form layout="inline" :model="searchParams" @finish="doSearch">
      <a-form-item label="账号">
        <a-input v-model:value="searchParams.userAccount" placeholder="输入账号" allow-clear />
      </a-form-item>
      <a-form-item label="用户名">
        <a-input v-model:value="searchParams.userName" placeholder="输入用户名" allow-clear />
      </a-form-item>
      <a-form-item>
        <a-button type="primary" html-type="submit">搜索</a-button>
      </a-form-item>
    </a-form>
    <div style="margin-bottom: 16px" />
    <!-- 表格 -->
    <a-table
      :columns="columns"
      :data-source="dataList"
      :pagination="pagination"
      @change="doTableChange"
    >
      <template #bodyCell="{ column, record }">
        <template
          v-if="['userName', 'userAvatar', 'userProfile', 'userRole'].includes(column.dataIndex)"
        >
          <div>
            <a-input
              v-if="editableData[record.id]"
              v-model:value="editableData[record.id][column.dataIndex]"
            />
            <template v-else>
              <template v-if="column.dataIndex === 'userName'">
                {{ record.userName }}
              </template>
              <template v-else-if="column.dataIndex === 'userAvatar'">
                <a-image :src="record.userAvatar" :width="120" />
              </template>
              <template v-else-if="column.dataIndex === 'userProfile'">
                {{ record.userProfile }}
              </template>
              <template v-else-if="column.dataIndex === 'userRole'">
                <div v-if="record.userRole === 'admin'">
                  <a-tag color="green">管理员</a-tag>
                </div>
                <div v-else>
                  <a-tag color="blue">普通用户</a-tag>
                </div>
              </template>
              <!--              <template v-else-if="column.dataIndex === 'createTime'">-->
              <!--                {{ dayjs(record.createTime).format('YYYY-MM-DD HH:mm:ss') }}-->
              <!--              </template>-->
            </template>
          </div>
        </template>
        <template v-else-if="column.key === 'action'">
          <div class="editable-row-operations">
            <span v-if="editableData[record.id]">
              <a-space>
                <a-typography-link @click="save(record.id)">保存</a-typography-link>
                <a-popconfirm title="确认取消？" @confirm="cancel(record.id)">
                  <a>取消</a>
                </a-popconfirm>
              </a-space>
            </span>
            <a-space v-else>
              <a-button primary @click="edit(record.id)">编辑</a-button>
              <a-button danger @click="doDelete(record.id)">删除</a-button>
            </a-space>
          </div>
        </template>
      </template>
    </a-table>
  </div>
</template>
<script lang="ts" setup>
import { computed, onMounted, reactive, ref, type UnwrapRef } from 'vue'
import {
  deleteUserUsingPost,
  listUserVoByPageUsingPost,
  updateUserUsingPost,
} from '@/api/userController.ts'
import { message } from 'ant-design-vue'
import { cloneDeep } from 'lodash-es'
import dayjs from 'dayjs'

const columns = [
  {
    title: 'id',
    dataIndex: 'id',
  },
  {
    title: '账号',
    dataIndex: 'userAccount',
  },
  {
    title: '用户名',
    dataIndex: 'userName',
  },
  {
    title: '头像',
    dataIndex: 'userAvatar',
  },
  {
    title: '简介',
    dataIndex: 'userProfile',
  },
  {
    title: '用户角色',
    dataIndex: 'userRole',
  },
  {
    title: '创建时间',
    dataIndex: 'createTime',
  },
  {
    title: '操作',
    key: 'action',
  },
]

// 定义数据
const dataList = ref<API.UserVO[]>([])
const total = ref(0)

// 搜索条件
const searchParams = reactive<API.UserQueryRequest>({
  current: 1,
  pageSize: 10,
  sortField: 'createTime',
  sortOrder: 'ascend',
})

// 获取数据
const fetchData = async () => {
  const res = await listUserVoByPageUsingPost({
    ...searchParams,
  })
  if (res.data.code === 0 && res.data.data) {
    dataList.value = res.data.data.records ?? []
    dataList.value.forEach(item => {
      item.createTime = dayjs(item.createTime).format('YYYY-MM-DD HH:mm:ss')
    })
    total.value = res.data.data.total ?? 0
  } else {
    message.error('获取数据失败：' + res.data.message)
  }
}

// 页面加载时获取数据，请求一次
onMounted(() => {
  fetchData()
})

// 分页参数
const pagination = computed(() => {
  return {
    current: searchParams.current,
    pageSize: searchParams.pageSize,
    total: total.value - 0, // 这里 -0 会将字符串转数值
    showSizeChanger: true,
    showTotal: (total) => `共 ${total} 条`,
  }
})

// 表格变化之后，重新获取数据
const doTableChange = (page: any) => {
  searchParams.current = page.current
  searchParams.pageSize = page.pageSize
  fetchData()
}

// 搜索数据
const doSearch = () => {
  // 重置页码
  searchParams.current = 1
  fetchData()
}

// 删除数据
const doDelete = async (id: string) => {
  if (!id) {
    return
  }
  const res = await deleteUserUsingPost({ id })
  if (res.data.code === 0) {
    message.success('删除成功')
    // 刷新数据
    fetchData()
  } else {
    message.error('删除失败')
  }
}

const editableData: UnwrapRef<Record<string, API.UserUpdateRequest>> = reactive({})

const edit = (id: string) => {
  editableData[id] = cloneDeep(dataList.value.filter((item) => id === item.id)[0])
}
const save = async (id: string) => {
  const res = await updateUserUsingPost({
    id: editableData[id].id,
    userName: editableData[id].userName,
    userAvatar: editableData[id].userAvatar,
    userProfile: editableData[id].userProfile,
    userRole: editableData[id].userRole,
  })
  if (res.data.code === 0) {
    message.success('保存成功')
    fetchData()
  } else {
    message.error('保存失败' + res.data.message)
  }
  delete editableData[id]
}
const cancel = (id: string) => {
  delete editableData[id]
}
</script>
