<script setup lang="ts">
import { NTabPane, NTabs } from 'naive-ui'
import { onMounted, ref, watch } from 'vue'
import aiDrawInputItem from './aiDrawInputItem.vue'
import aiFace from './aiFace.vue'
import aiBlend from './aiBlend.vue'
import { useBasicLayout } from '@/hooks/useBasicLayout'
import { SvgIcon } from '@/components/common'
import { gptServerStore } from '@/store'
import { mlog } from '@/api'
const $emit = defineEmits(['drawSent', 'close'])
const drawSent = (d: any) => $emit('drawSent', d)
const { isMobile } = useBasicLayout()

const st = ref({ drawType: 'draw' })

onMounted(() => {
  // st.value.drawType='draw'
  if (gptServerStore.myData.DRAW_TYPE)
    st.value.drawType = gptServerStore.myData.DRAW_TYPE
})

watch(() => st.value.drawType, (n: string) => {
  mlog('st.value.drawType', n)
  gptServerStore.setMyData({ DRAW_TYPE: n })
})
</script>

<template>
  <div class="overflow-y-auto bg-[#fafbfc] pt-2 dark:bg-[#18181c] h-full ">
    <NTabs v-model:value="st.drawType" type="line" animated default-value="draw">
      <NTabPane name="start" tab="" />
      <NTabPane name="draw" tab="MidJourney">
        <!--  -->
        <NTabs type="segment" animated default-value="draw23" size="small">
          <NTabPane name="draw23" :tab="$t('mjchat.draw')">
            <aiDrawInputItem @draw-sent="drawSent" @close="$emit('close')" />
          </NTabPane>
          <!-- <n-tab-pane name="chap2" tab="第二章">2</n-tab-pane>
        <n-tab-pane name="chap3" tab="第三章">3</n-tab-pane> -->
          <NTabPane name="face" :tab="$t('mjchat.face')">
            <div class="p-4">
              <aiFace />
            </div>
          </NTabPane>
          <NTabPane name="blend" :tab="$t('mjchat.blend')">
            <div class="p-4">
              <aiBlend />
            </div>
          </NTabPane>
        </NTabs>
      </NTabPane>

      <!-- <n-tab-pane name="dall3" tab="Dall.E">
     <div class="p-4"><aiDall  /></div>
    </n-tab-pane>

    <n-tab-pane name="ideogram" tab="IdeoGram">
     <div class="p-2"> <aiIdeoInput/> </div>
    </n-tab-pane> -->

      <NTabPane v-if="isMobile" name="Close">
        <template #tab>
          <div class=" text-center flex justify-center items-center" @click="$emit('close')">
            <SvgIcon icon="ri:close-circle-line" />
          </div>
        </template>
        <div class="p-4">
          <div class=" justify-center items-center flex" @click="$emit('close')">
            <SvgIcon icon="ri:close-circle-line" /> Close By Click me
          </div>
        </div>
      </NTabPane>
    </NTabs>
  </div>
</template>
