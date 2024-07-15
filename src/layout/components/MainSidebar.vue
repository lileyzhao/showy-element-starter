<script setup lang="ts" name="Layout-MainSidebar">
import Logo from '@/layout/components/Logo.vue'
import { useAppStore } from '@/store'
import { mapRoutesToElMenuItemMain } from '@/utils/menuUtil'
import { getFullRoutes } from '@/utils'

const emit = defineEmits(['keyChange'])

const { t } = useI18n()
const app = useAppStore()
const route = useRoute()
const fullRoutes = getFullRoutes()
const mainMenuRoutes = fullRoutes.filter(route => route.meta.parentName === 'root').filter(route => !route.meta?.hidden) ?? []

/** Selected Item in main-menu 主栏菜单选中项 */
const mainMenuKey = ref<string>()

/** Collapsed State of main-menu 主栏菜单收缩状态 */
const collMainMenu = computed({
  get: () => app.MenuSetting.mainMenu.collapsed,
  set: val => app.setMenuSetting({ mainMenu: { collapsed: val } }),
})

/** Collapsed State of sub-menu 副栏菜单收缩状态 */
const collSubMenu = computed({
  get: () => app.MenuSetting.subMenu.collapsed,
  set: (val) => {
    if (app.MenuSetting.subMenu.collapsed !== val)
      app.setMenuSetting({ subMenu: { collapsed: val } })
  },
})

/** Main column reverse color 主栏反转颜色 */
const mainMenuInverted = computed({
  get: () => app.MenuSetting.mainMenu.inverted,
  set: val => app.setMenuSetting({ mainMenu: { inverted: val } }),
})

/** main-menu item 主栏菜单项 */
const mainMenuItems = computed(() => {
  const mainMenuSetting = app.MenuSetting.mainMenu
  return mainMenuRoutes.map(route => mapRoutesToElMenuItemMain(route, fullRoutes, t, app.MenuSetting.subMenu.collapsed, mainMenuSetting))
})

/** Handle main menu key change 处理主菜单键变化 */
const handleMainMenuKeyChange = (key: string) => emit('keyChange', key)

/** Refresh main menu 刷新主菜单 */
const refreshMainMenu = () => {
  // This is to manage the selected item by forcing a refresh of the menu component
  // 这里是为了强制刷新菜单组件来达到控制选中项
  mainMenuKey.value = undefined
  nextTick(() => {
    mainMenuKey.value = (app.MenuSetting.subMenu.collapsed ? route.name : route.matched[1].name) as string
    console.log('refreshMainMenu:mainMenuKey.value:', mainMenuKey.value)
    handleMainMenuKeyChange(mainMenuKey.value)
  })
}

/** Style class for logo logo的样式类 */
const logoClass = computed(() => !app.IsDarkMode && app.MenuSetting.mainMenu.inverted ? 'xb-bl-gray2! xb-b-1!' : '')

onMounted(refreshMainMenu)

watch(() => app.MenuSetting.subMenu.collapsed, refreshMainMenu)

/** Exposes 公开对象 */
defineExpose({ refreshMainMenu })
</script>

<template>
  <!-- Sidebar (Desktop): Main Column 侧边栏(电脑端):主栏 -->
  <el-aside
    :width="`${collMainMenu ? app.MenuSetting.mainMenu.widthColl : app.MenuSetting.subMenu.collapsed ? app.MenuSetting.mainMenu.widthSingle : app.MenuSetting.mainMenu.width}px`"
    style="height:100vh;border-right:1px solid var(--el-border-color);z-index:1;"
  >
    <el-container class="h-full">
      <el-header
        :class="logoClass" :style="{
          backgroundColor: mainMenuInverted ? '#18181c' : 'transparent',
          padding: 0,
          height: 'auto',
          lineHeight: 'auto',
          borderBottom: mainMenuInverted && !app.IsDarkMode ? '1px solid #4c4d4f' : `1px solid var(--el-border-color)`,
        }"
      >
        <Logo w-full :hide-title="collMainMenu" />
      </el-header>
      <!-- Main Menu 主栏菜单 -->
      <el-main class="of-x-hidden! p0!">
        <el-menu
          v-if="mainMenuKey" class="main-menu h-full! w-full! b-r-none!"
          :background-color="mainMenuInverted ? '#18181c' : undefined"
          :text-color="mainMenuInverted ? '#bbb' : undefined" unique-opened :default-active="mainMenuKey"
          :collapse="collMainMenu && collSubMenu" @select="handleMainMenuKeyChange"
        >
          <template #default>
            <component :is="menuItem" v-for="menuItem in mainMenuItems" :key="menuItem.key" />
          </template>
        </el-menu>
      </el-main>
      <el-footer
        class="h-auto! px-0! py-3!"
        :style="`background-color:${mainMenuInverted ? '#18181c' : 'transparent'};`"
      >
        <div v-if="!app.IsDarkMode && !collMainMenu" relative h-20px w-full>
          <div
            :class="`left-50% -translate-x-1/2 i-line-md:${mainMenuInverted ? 'sunny-filled hover:text-yellow text-white' : 'moon-filled hover:text-purple'}`"
            absolute cursor-pointer text-18px @click="mainMenuInverted = !mainMenuInverted"
          />
          <div
            v-if="collSubMenu" :class="`i-carbon:side-panel-close ${mainMenuInverted ? 'text-white' : ''}`" absolute
            right-2 rotate-180 cursor-pointer text-18px @click="collSubMenu = !collSubMenu"
          />
          <div
            v-else :class="`i-carbon:side-panel-close ${mainMenuInverted ? 'text-white' : ''}`" absolute right-2
            cursor-pointer text-18px @click="collSubMenu = !collSubMenu"
          />
        </div>
        <div v-else-if="!app.IsDarkMode && collMainMenu" bottom-12px w-full flex justify-center gap-2.5>
          <div
            :class="`i-line-md:${mainMenuInverted ? 'sunny-filled hover:text-yellow text-white' : 'moon-filled hover:text-purple'}`"
            cursor-pointer text-18px @click="mainMenuInverted = !mainMenuInverted"
          />
          <div
            v-if="collSubMenu" :class="`i-carbon:side-panel-close ${mainMenuInverted ? 'text-white' : ''}`"
            rotate-180 cursor-pointer text-18px @click="collSubMenu = !collSubMenu"
          />

          <div
            v-else :class="`i-carbon:side-panel-close ${mainMenuInverted ? 'text-white' : ''}`" cursor-pointer
            text-18px @click="collSubMenu = !collSubMenu"
          />
        </div>
      </el-footer>
    </el-container>
  </el-aside>
</template>

<style scoped lang="scss">
.main-menu {
  :deep(.el-sub-menu__title) {
    --at-apply: h-auto!;
  }
}
</style>
