<script setup lang="ts">
import { computed, ref, watch } from 'vue'
import { NDrawer, NDrawerContent } from 'naive-ui'
import aiDrawInput from './aiDrawInput.vue'
import { SvgIcon } from '@/components/common'
import { homeStore } from '@/store'
import { router } from '@/router'

import { isDisableMenu } from '@/api'
const st = ref({ show: true })

const goHome = computed(() => {
  // router.push('/')
  return router.currentRoute.value.name
})
function drawSent(e: any) {
  st.value.show = false
  // $emit('drawSent', e)
  homeStore.setMyData({ act: 'draw', actData: e })
}

watch(() => homeStore.myData.act, (n: string) => {
  if (n == 'showChat')
    router.push('/chat')

  if (n == 'showDraw') {
    router.push('/draw')
    st.value.show = true
  }
  if (n == 'draw')
    st.value.show = false
})
</script>

<template>
  <div class=" bg-gray-100 dark:bg-[#282832] h-[55px] flex  justify-around  items-center dark:text-white/70 ">
    <!-- <div class="flex items-center justify-center flex-col"  @click="homeStore.setMyData({act:'showChat'}) "   :class="[ goHome =='Chat' ? 'active' : '']" >
        <SvgIcon icon="ri:wechat-line" class="text-3xl"></SvgIcon>
        <div class="text-[13px]">{{$t('mjtab.chat')}}</div>
      </div>
      <div  v-if="!isDisableMenu ( 'gpts')"  class="flex items-center justify-center flex-col "  @click="homeStore.setMyData({act:'showgpts'}) " >
        <SvgIcon icon="ri:apps-fill" class="text-3xl"></SvgIcon>
        <div class="text-[13px]">GPTs</div>
      </div>

      <div v-if="!isDisableMenu ( 'realtime')"    class="flex items-center justify-center flex-col "  @click="homeStore.setMyData({act:'openRealtime'}) " >
        <SvgIcon icon="ri:mic-fill" class="text-3xl"></SvgIcon>
        <div class="text-[13px]">{{$t('mj.rttab')}}</div>
      </div> -->

    <div v-if="!isDisableMenu ('draws')" class="flex items-center justify-center flex-col " :class="[goHome == 'draw' ? 'active' : '']" @click="homeStore.setMyData({ act: 'showDraw' }) ">
      <SvgIcon icon="ic:outline-palette" class="text-3xl" />
      <div class="text-[13px]">
        {{ $t('mjtab.draw') }}
      </div>
    </div>
    <!-- <div  v-if="!isDisableMenu ( 'gallery')"  class="flex items-center justify-center flex-col " @click="homeStore.setMyData({act:'gallery'})" >
        <SvgIcon icon="material-symbols:imagesmode-outline" class="text-3xl"></SvgIcon>
        <div class="text-[13px]">{{$t('mjtab.gallery')}}</div>
      </div>  -->
  </div>

  <NDrawer v-if="goHome == 'draw'" v-model:show="st.show" class="!h-[90vh] !max-h-[660px]" placement="bottom">
    <NDrawerContent style="--n-body-padding:0" class="h-full">
      <aiDrawInput @draw-sent="drawSent" @close="st.show = false" />
    </NDrawerContent>
  </NDrawer>
</template>
