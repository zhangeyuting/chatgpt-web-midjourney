<script setup lang='ts'>
import { computed, onBeforeUnmount, onMounted } from 'vue'
import { NLayout, NLayoutContent } from 'naive-ui'
import { useRouter } from 'vue-router'
import Sider from '../chat/layout/sider/index.vue'
import Permission from '../chat/layout/Permission.vue'
import { useBasicLayout } from '@/hooks/useBasicLayout'
import { gptServerStore, homeStore, useAppStore, useAuthStore, useChatStore } from '@/store'
import { aiFooter } from '@/views/mj'
import aiMobileMenu from '@/views/mj/aiMobileMenu.vue'

const router = useRouter()
const appStore = useAppStore()
const chatStore = useChatStore()
const authStore = useAuthStore()

router.replace({ name: 'draw', params: { uuid: chatStore.active } })
homeStore.setMyData({ local: 'draw' })
const { isMobile } = useBasicLayout()

const collapsed = computed(() => appStore.siderCollapsed)

const needPermission = computed(() => !!authStore.session?.auth && !authStore.token)

const getMobileClass = computed(() => {
  if (isMobile.value)
    return ['rounded-none', 'shadow-none']
  return ['shadow-md', 'dark:border-neutral-800'] // 'border', 'rounded-md',
})

const getContainerClass = computed(() => {
  return [
    'h-full',
    { abc: !isMobile.value && !collapsed.value },
  ]
})

onMounted(() => {
  // 添加监听器接收消息
  window.addEventListener('message', handleMessage)
})

onBeforeUnmount(() => {
  // 组件销毁时移除监听器
  window.removeEventListener('message', handleMessage)
})

function handleMessage(event) {
  if (event.origin !== 'http://localhost:3001')
    return

  if (event?.data?.authCode) {
    gptServerStore.setMyData({ MJ_API_SECRET: event.data.authCode })
    localStorage.setItem('authCode', event.data.authCode)
    window.removeEventListener('message', handleMessage)
  }
}
</script>

<template>
  <div class="dark:bg-[#24272e] transition-all p-0" :class="[isMobile ? 'h55' : 'h-full']">
    <div class="h-full overflow-hidden" :class="getMobileClass">
      <NLayout class="z-40 transition" :class="getContainerClass" has-sider :sider-placement="isMobile ? 'left' : 'right'">
        <!-- <aiSider v-if="!isMobile"/> -->

        <NLayoutContent class="h-full">
          <RouterView v-slot="{ Component, route }">
            <component :is="Component" :key="route.fullPath" />
          </RouterView>
        </NLayoutContent>
        <Sider />
      </NLayout>
    </div>
    <Permission :visible="needPermission" />
  </div>
  <aiMobileMenu v-if="isMobile" />
  <aiFooter />
</template>

<style>
.h55{
  height: calc(100% - 55px);
}
</style>
