<template>
  <div ref="containerRef" class="detail">
    <div class="detail_header">
      <a-affix :target="(containerRef as HTMLElement)">
        <a-page-header title="通知公告" :subtitle="type === 'edit' ? '修改' : '新增'" @back="onBack">
          <template #extra>
            <a-button type="primary" @click="onReleased">
              <template #icon>
                <icon-save v-if="type === 'edit'" />
                <icon-send v-else />
              </template>
              <template #default>
                {{ type === 'edit' ? '保存' : '发布' }}
              </template>
            </a-button>
          </template>
        </a-page-header>
      </a-affix>
    </div>
    <div class="detail_content" style="display: flex; flex-direction: column;">
      <GiForm ref="formRef" v-model="form" :options="options" :columns="columns">
        <template #noticeUsers>
          <UserSelect v-model:value="form.noticeUsers" :multiple="true" class="w-full" />
        </template>
      </GiForm>
      <div style="flex: 1;">
        <AiEditor v-model="form.content" />
      </div>
    </div>
  </div>
</template>

<script setup lang="tsx">
import { Message } from '@arco-design/web-vue'
import AiEditor from '../components/edit/index.vue'
import { useTabsStore } from '@/stores'
import { type Columns, GiForm, type Options } from '@/components/GiForm'
import { addNotice, getNotice, updateNotice } from '@/apis/system'
import { useForm } from '@/hooks'
import { useDict } from '@/hooks/app'

const { notice_type } = useDict('notice_type')
const containerRef = ref<HTMLElement | null>()
const tabsStore = useTabsStore()
const route = useRoute()
const formRef = ref<InstanceType<typeof GiForm>>()
const { id, type } = route.query
const { form, resetForm } = useForm({
  title: '',
  type: '',
  effectiveTime: '',
  terminateTime: '',
  content: '',
  noticeScope: 1
})
const options: Options = {
  form: { size: 'large' },
  col: { xs: 24, sm: 24, md: 12, lg: 12, xl: 12, xxl: 12 },
  btns: { hide: true }
}

const columns: Columns = reactive([
  {
    label: '标题',
    field: 'title',
    type: 'input',
    props: {
      maxLength: 150,
      showWordLimit: true
    },
    rules: [{ required: true, message: '请输入标题' }]
  },
  {
    label: '类型',
    field: 'type',
    type: 'select',
    options: notice_type,
    rules: [{ required: true, message: '请输入类型' }]
  },
  {
    label: '生效时间',
    field: 'effectiveTime',
    type: 'date-picker',
    props: {
      showTime: true
    }

  },
  {
    label: '终止时间',
    field: 'terminateTime',
    type: 'date-picker',
    props: {
      showTime: true
    }
  },
  {
    label: '通知范围',
    field: 'noticeScope',
    type: 'radio-group',
    options: [{ label: '所有人', value: 1 }, { label: '指定用户', value: 2 }],
    rules: [{ required: true, message: '请选择通知范围' }]
  },
  {
    label: '指定用户',
    field: 'noticeUsers',
    type: 'input',
    hide: () => {
      return form.noticeScope === 1
    },
    rules: [{ required: true, message: '请选择指定用户' }]
  }

])
// 修改
const onUpdate = async (id: string) => {
  resetForm()
  const res = await getNotice(id)
  Object.assign(form, res.data)
}
const onBack = () => {
  tabsStore.closeCurrent(route.path)
}
const onReleased = async () => {
  const isInvalid = await formRef.value?.formRef?.validate()
  if (isInvalid) return false
  try {
    // 通知范围 所有人 去除指定用户
    form.noticeUsers = form.noticeScope === 1 ? null : form.noticeUsers
    if (type === 'edit') {
      await updateNotice(form, id as string)
      Message.success('修改成功')
    } else {
      await addNotice(form)
      Message.success('新增成功')
    }
    onBack()
    return true
  } catch (error) {
    console.error(error)
    return false
  }
}
onMounted(() => {
  // 当id存在与type为edit时，执行修改操作
  if (id && type === 'edit') {
    onUpdate(id as string)
  }
})
</script>

<style scoped lang="less"></style>
