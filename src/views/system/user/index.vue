<template>
  <div class="table-page">
    <a-row justify="space-between" align="center" class="header page_header">
      <a-space wrap>
        <div class="title">用户管理</div>
      </a-space>
    </a-row>
    <a-row align="stretch" :gutter="14" class="h-full page_content">
      <a-col :xs="0" :sm="0" :md="6" :lg="5" :xl="5" :xxl="4" class="h-full ov-hidden">
        <DeptTree placeholder="请输入名称" @node-click="handleSelectDept" />
      </a-col>
      <a-col :xs="24" :sm="24" :md="18" :lg="19" :xl="19" :xxl="20" class="h-full ov-hidden">
        <GiTable
          row-key="id"
          :data="dataList"
          :columns="columns"
          :loading="loading"
          :scroll="{ x: '100%', y: '100%', minWidth: 1500 }"
          :pagination="pagination"
          :disabled-tools="['size']"
          :disabled-column-keys="['username']"
          @refresh="search"
        >
          <template #top>
            <GiForm v-model="queryForm" :options="options" :columns="queryFormColumns" @search="search" @reset="reset"></GiForm>
          </template>
          <template #toolbar-left>
            <a-button v-permission="['system:user:add']" type="primary" @click="onAdd">
              <template #icon><icon-plus /></template>
              <template #default>新增</template>
            </a-button>
            <a-button v-permission="['system:user:import']" @click="onImport">
              <template #icon><icon-upload /></template>
              <template #default>导入</template>
            </a-button>
          </template>
          <template #toolbar-right>
            <a-button v-permission="['system:user:export']" @click="onExport">
              <template #icon><icon-download /></template>
              <template #default>导出</template>
            </a-button>
          </template>
          <template #username="{ record }">
            <GiCellAvatar :avatar="getAvatar(record.avatar, record.gender)" :name="record.username" is-link
                          @click="onDetail(record)" />
          </template>
          <template #gender="{ record }">
            <GiCellGender :gender="record.gender" />
          </template>
          <template #roleNames="{ record }">
            <GiCellTags :data="record.roleNames" />
          </template>
          <template #status="{ record }">
            <GiCellStatus :status="record.status" />
          </template>
          <template #isSystem="{ record }">
            <a-tag v-if="record.isSystem" color="red" size="small">是</a-tag>
            <a-tag v-else color="arcoblue" size="small">否</a-tag>
          </template>
          <template #action="{ record }">
            <a-space>
              <a-link v-permission="['system:user:update']" @click="onUpdate(record)">修改</a-link>
              <a-link
                v-permission="['system:user:delete']"
                status="danger"
                :title="record.isSystem ? '系统内置数据不能删除' : '删除'"
                :disabled="record.disabled"
                @click="onDelete(record)"
              >
                删除
              </a-link>
              <a-dropdown>
                <a-button v-if="has.hasPermOr(['system:user:resetPwd'])" type="text" size="mini">
                  <template #icon>
                    <icon-more :size="16" />
                  </template>
                </a-button>
                <template #content>
                  <a-doption v-permission="['system:user:resetPwd']" @click="onResetPwd(record)">重置密码</a-doption>
                </template>
              </a-dropdown>
            </a-space>
          </template>
        </GiTable>
      </a-col>
    </a-row>

    <UserAddDrawer ref="UserAddDrawerRef" @save-success="search" />
    <UserImportDrawer ref="UserImportDrawerRef" @save-success="search" />
    <UserDetailDrawer ref="UserDetailDrawerRef" />
    <UserResetPwdModal ref="UserResetPwdModalRef" />
  </div>
</template>

<script setup lang="ts">
import DeptTree from './dept/index.vue'
import UserAddDrawer from './UserAddDrawer.vue'
import UserImportDrawer from './UserImportDrawer.vue'
import UserDetailDrawer from './UserDetailDrawer.vue'
import UserResetPwdModal from './UserResetPwdModal.vue'
import { type UserQuery, type UserResp, deleteUser, exportUser, listUser } from '@/apis/system'
import type { Columns, Options } from '@/components/GiForm'
import type { TableInstanceColumns } from '@/components/GiTable/type'
import { useDownload, useTable } from '@/hooks'
import { isMobile } from '@/utils'
import getAvatar from '@/utils/avatar'
import has from '@/utils/has'
import { DisEnableStatusList } from '@/constant/common'

defineOptions({ name: 'SystemUser' })

const queryForm = reactive<UserQuery>({
  sort: ['t1.createTime,desc']
})
const {
  tableData: dataList,
  loading,
  pagination,
  search,
  handleDelete
} = useTable((page) => listUser({ ...queryForm, ...page }), { immediate: false })

const options: Options = reactive({
  form: { layout: 'inline' },
  col: { xs: 24, sm: 24, md: 5, lg: 4, xl: 4, xxl: 4 },
  btns: { col: { xs: 24, sm: 24, md: 7, lg: 8, xl: 6, xxl: 6 } },
  fold: { enable: true, index: 1, defaultCollapsed: true }
})

const queryFormColumns: Columns = reactive([
  {
    type: 'input',
    field: 'description',
    item: {
      hideLabel: true
    },
    props: {
      placeholder: '用户名/昵称/描述',
      allowClear: true
    }
  },
  {
    type: 'select',
    field: 'status',
    options: DisEnableStatusList,
    item: {
      hideLabel: true
    },
    props: {
      placeholder: '请选择状态',
      allowClear: true
    }
  },
  {
    type: 'range-picker',
    field: 'createTime',
    item: {
      hideLabel: true
    },
    col: { xs: 24, sm: 24, md: 10, lg: 9.5, xl: 9, xxl: 8 }
  }
])

const columns: TableInstanceColumns[] = [
  {
    title: '序号',
    width: 66,
    align: 'center',
    render: ({ rowIndex }) => h('span', {}, rowIndex + 1 + (pagination.current - 1) * pagination.pageSize),
    fixed: !isMobile() ? 'left' : undefined
  },
  {
    title: '用户名',
    dataIndex: 'username',
    slotName: 'username',
    width: 140,
    ellipsis: true,
    tooltip: true,
    fixed: !isMobile() ? 'left' : undefined
  },
  { title: '昵称', dataIndex: 'nickname', width: 120, ellipsis: true, tooltip: true },
  { title: '状态', slotName: 'status', align: 'center', width: 80 },
  { title: '性别', slotName: 'gender', align: 'center', width: 100 },
  { title: '所属部门', dataIndex: 'deptName', ellipsis: true, tooltip: true, width: 180 },
  { title: '角色', dataIndex: 'roleNames', slotName: 'roleNames', width: 160 },
  { title: '手机号', dataIndex: 'phone', width: 170, ellipsis: true, tooltip: true },
  { title: '邮箱', dataIndex: 'email', width: 170, ellipsis: true, tooltip: true },
  { title: '系统内置', slotName: 'isSystem', width: 100, align: 'center', show: false },
  { title: '描述', dataIndex: 'description', width: 130, ellipsis: true, tooltip: true },
  { title: '创建人', dataIndex: 'createUserString', ellipsis: true, tooltip: true, width: 140, show: false },
  { title: '创建时间', dataIndex: 'createTime', width: 180 },
  { title: '修改人', dataIndex: 'updateUserString', ellipsis: true, tooltip: true, width: 140, show: false },
  { title: '修改时间', dataIndex: 'updateTime', width: 180, show: false },
  {
    title: '操作',
    slotName: 'action',
    width: 150,
    align: 'center',
    fixed: !isMobile() ? 'right' : undefined,
    show: has.hasPermOr(['system:user:update', 'system:user:delete', 'system:user:resetPwd'])
  }
]

// 重置
const reset = () => {
  queryForm.description = undefined
  queryForm.status = undefined
  queryForm.createTime = []
  search()
}

// 删除
const onDelete = (record: UserResp) => {
  return handleDelete(() => deleteUser(record.id), {
    content: `是否确定删除 [${record.nickname}(${record.username})]？`,
    showModal: true
  })
}

// 导出
const onExport = () => {
  useDownload(() => exportUser(queryForm))
}

// 根据选中部门查询
const handleSelectDept = (keys: Array<any>) => {
  queryForm.deptId = keys.length === 1 ? keys[0] : undefined
  search()
}

const UserAddDrawerRef = ref<InstanceType<typeof UserAddDrawer>>()
// 新增
const onAdd = () => {
  UserAddDrawerRef.value?.onAdd()
}

const UserImportDrawerRef = ref<InstanceType<typeof UserImportDrawer>>()
// 导入
const onImport = () => {
  UserImportDrawerRef.value?.onImport()
}

// 修改
const onUpdate = (record: UserResp) => {
  UserAddDrawerRef.value?.onUpdate(record.id)
}

const UserDetailDrawerRef = ref<InstanceType<typeof UserDetailDrawer>>()
// 详情
const onDetail = (record: UserResp) => {
  UserDetailDrawerRef.value?.onDetail(record.id)
}

const UserResetPwdModalRef = ref<InstanceType<typeof UserResetPwdModal>>()
// 重置密码
const onResetPwd = (record: UserResp) => {
  UserResetPwdModalRef.value?.onReset(record.id)
}
</script>

<style lang="scss" scoped>
.page_header {
  flex: 0 0 auto;
}

.page_content {
  flex: 1;
  overflow: auto;
}
</style>
