<script setup lang="ts">
import zhCn from 'element-plus/es/locale/lang/zh-cn'
import { useCssVar } from '@vueuse/core'
// import { useAppStore } from '@/store/modules/app'

// const app = useAppStore()

const data = ref({
  borderRadius: 4,
  colorPrimary: '#646CFF',
})

useCssVar('--color')

// 主题变量：https://github.com/element-plus/element-plus/blob/dev/packages/theme-chalk/src/common/var.scss
const el = ref(null)
const primaryColor = useCssVar('--el-color-primary', el)
primaryColor.value = data.value.colorPrimary

const globalConfig = reactive({
  size: 'default' as 'default' | 'small' | 'large',
  zIndex: 3000,
  locale: zhCn,
  buttonConfig: { autoInsertSpace: false },
  messageConfig: { max: 3 },
})
</script>

<template>
  <el-config-provider
    :size="globalConfig.size" :z-index="globalConfig.zIndex" :locale="globalConfig.locale"
    :button="globalConfig.buttonConfig" :message="globalConfig.messageConfig"
  >
    <router-view />
  </el-config-provider>
</template>
