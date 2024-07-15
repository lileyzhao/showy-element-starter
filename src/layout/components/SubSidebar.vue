<script setup lang="ts" name="Layout-SubSidebar">
import { useAppStore } from '@/store'
import Logo from '@/layout/components/Logo.vue'
import { MenuPositionEnum } from '@/shared'
import { mapRoutesToElMenuItem } from '@/utils/menuUtil'
import { getFullRoutes } from '@/utils'

const props = defineProps({ parentMenuKey: { type: String, required: false } })

// Variables 变量
const { t } = useI18n()
const app = useAppStore()
const route = useRoute()
const fullRoutes = getFullRoutes()

/** Selected Item in sub-menu 副栏菜单选中项 */
const subMenuKey = ref<string | undefined>(route.name as string)

const subMenuRoutes = computed(() => {
  return props.parentMenuKey ? fullRoutes.filter(r => r.meta.parentName === (props.parentMenuKey || route.matched[1].name)) : []
})

/** sub-menu items 副栏菜单项 */
const subMenuItems = computed(() => {
  if ((!app.MenuSetting.subMenu.collapsed || app.MenuSetting.topMenu.showSubMenu) && subMenuRoutes.value.length > 0)
    return fullRoutes.filter(route => route.meta.parentName === props.parentMenuKey).map(route => mapRoutesToElMenuItem(route, fullRoutes, t, true))
  else return []
})

/** Refresh main menu 刷新副栏菜单 */
const refreshSubMenu = () => {
  // This is to manage the selected item by forcing a refresh of the menu component
  // 这里是为了强制刷新菜单组件来达到控制选中项
  subMenuKey.value = undefined
  nextTick(() => {
    subMenuKey.value = route.name as string
  })
}

/** Exposes 公开对象 */
defineExpose({ refreshSubMenu })
</script>

<template>
  <!-- Sidebar (desktop): Sub-sidebar 侧边栏(电脑端):副栏 -->
  <el-aside
    v-if="subMenuItems.length > 0 && (!app.MenuSetting.subMenu.collapsed || app.MenuSetting.topMenu.showSubMenu)"
    :width="`${app.MenuSetting.subMenu.width}px`" class="h-100vh of-x-hidden!"
    style="border-right:1px solid var(--el-border-color);"
  >
    <el-container class="h-full">
      <el-header
        bordered
        :style="{ padding: 0, height: 'auto', lineHeight: 'auto', borderBottom: `1px solid var(--el-border-color)` }"
      >
        <Logo
          v-if="app.MenuSetting.menuPosition === MenuPositionEnum.TOP_BAR && app.MenuSetting.topMenu.showSubMenu"
          flex-y-center p-l-5
        />
        <Logo v-else hide-logo :hide-title="!app.MenuSetting.mainMenu.collapsed" flex-y-center p-l-5 />
      </el-header>
      <el-main class="p0!">
        <!-- Sub-menu 副栏菜单 -->
        <el-menu v-if="subMenuKey" class="sub-menu b-r-none!" unique-opened :default-active="subMenuKey">
          <template #default>
            <component :is="menuItem" v-for="menuItem in subMenuItems" :key="menuItem.key" />
          </template>
        </el-menu>
      </el-main>
    </el-container>
  </el-aside>
</template>

<style scoped lang="scss">
:deep(.sub-menu.el-menu) {
  .el-menu-item {
    --at-apply: pr-0 !;
    padding-right: 0 !important;

    a {
      width: 100%;
    }
  }
}
</style>
