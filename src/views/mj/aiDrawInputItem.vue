<script setup lang="ts">
import { computed, onMounted, ref, watch } from 'vue'
import { NButton, NInput, NInputNumber, NPopover, NSelect, NTag, useMessage } from 'naive-ui'
import config from './draw.json'
import AiMsg from './aiMsg.vue'
import { SvgIcon } from '@/components/common'
import { useBasicLayout } from '@/hooks/useBasicLayout'
// import aiFace from './aiFace.vue'
import { getMjAll, mjFetch, mlog, train, upImg } from '@/api'
// import {copyText3} from "@/utils/format";
import { homeStore, useChatStore } from '@/store'
import { t } from '@/locales'
const props = defineProps({ buttonDisabled: Boolean })
const $emit = defineEmits(['drawSent', 'close'])
const { isMobile } = useBasicLayout()
const chatStore = useChatStore()
// import { upImg } from "./mj";

const vf = [{ s: 'width: 100%; height: 100%;', label: '1:1' },
  { s: 'width: 100%; height: 75%;', label: '4:3' },
  { s: 'width: 75%; height: 100%;', label: '3:4' },
  { s: 'width: 100%; height: 50%;', label: '16:9' },
  { s: 'width: 50%; height: 100%;', label: '9:16' },
]

const f = ref({ bili: -1, quality: '', view: '', light: '', shot: '', style: '', styles: '', version: '--v 6.1', speed: 'fast', sref: '', cref: '', cw: '' })
const st = ref({
  text: '',
  isDisabled: false,
  isLoad: false,
  fileBase64: [],
  bot: '',
  showFace: false,
  upType: '',
})
const farr = [
  { k: 'style', v: t('mjchat.tStyle') },
  { k: 'view', v: t('mjchat.tView') },
  { k: 'shot', v: t('mjchat.tShot') },
  { k: 'light', v: t('mjchat.tLight') },
  { k: 'quality', v: t('mjchat.tQuality') },
  { k: 'styles', v: t('mjchat.tStyles') },
  { k: 'version', v: t('mjchat.tVersion') },
]

const drawlocalized = computed(() => {
  const localizedConfig = {}
  Object.keys(config).forEach((key) => {
    localizedConfig[key] = config[key].map((option) => {
      // 假设 labelKey 如 "draw.qualityList.general"
      const path = option.labelKey // 直接使用 labelKey 作为路径
      return {
        ...option,
        label: t(path), // 从 i18n 中获取本地化的标签
      }
    })
  })
  return localizedConfig
})

const msgRef = ref()
const fsRef = ref()
const fsRef2 = ref()
const fsRef3 = ref()
const isDisabled = computed(() => {
  return props.buttonDisabled || st.value.isLoad || st.value.text.trim() == ''
})
const ms = useMessage()
function create() {
  st.value.isLoad = true
  train(st.value.text.trim()).then((ps) => {
    const rz = { prompt: st.value.text.trim(), drawText: createPrompt(ps) }
    if (ps)
      drawSent(rz)
    // st.value.text=''
    st.value.isLoad = false
  }).catch((err) => {
    msgRef.value.showError(err)
    st.value.isLoad = false
  })
}

const shorten = () => {
  if (st.value.text.trim() == '') {
    mlog('empty')
    msgRef.value.showError(t('mjchat.placeInput'))
    return
  }

  const obj = {
    action: 'shorten',
    data: { prompt: st.value.text.trim(), botType: st.value.bot == 'NIJI_JOURNEY' ? 'NIJI_JOURNEY' : 'MID_JOURNEY' },
  }
  homeStore.setMyData({ act: 'draw', actData: obj })
}
function drawSent(rz: any) {
  const rz2 = rz
  if (st.value.fileBase64)
    rz2.fileBase64 = st.value.fileBase64

  if (st.value.bot == 'NIJI_JOURNEY')
    rz2.bot = 'NIJI_JOURNEY'

  $emit('drawSent', rz2)
  st.value.fileBase64 = []
}
function createPrompt(rz: string) {
  if (rz == '') {
    msgRef.value.showError(t('mjchat.placeInput'))
    return ''
  }

  // for(let v of farr){
  //     if( ! f.value[v.k] || f.value[v.k]==null || f.value[v.k]=='' ) continue;
  //      mlog('k ', rz,  f.value  );
  //     if(v.k=='quality') rz +=`  --q ${f.value.quality}`;
  //     else if(v.k=='styles') { if( f.value.styles ) rz +=` ${f.value.styles}`;}
  //     else if(v.k=='version') {
  //         st.value.bot= '';
  //        if(['MID_JOURNEY','NIJI_JOURNEY'].indexOf(f.value.version)>-1 ){
  //              st.value.bot= f.value.version ;
  //        } else   rz +=` ${f.value.version}`;
  //     }
  //     else if( f.value[v.k] ) rz +=` , ${f.value[v.k]}`;
  // }
  // mlog('createPrompt ', rz,  f.value  );
  // if(f.value.bili>-1) rz +=` --ar ${vf[f.value.bili].label}`;
  let rzp = '' // 参数组合字符串
  let rzk = '' // 描述词组合字符串
  for (const v of farr) {
    if (!f.value[v.k] || f.value[v.k] == null || f.value[v.k] == '')
      continue
    mlog('k ', rz, f.value)
    if (v.k == 'quality') { rzp += `  --q ${f.value.quality}` }
    else if (v.k == 'styles') {
      if (f.value.styles)
        rzp += ` ${f.value.styles} `
    }
    else if (v.k == 'version') {
      st.value.bot = ''
      if (['MID_JOURNEY', 'NIJI_JOURNEY'].includes(f.value.version))
        st.value.bot = f.value.version
      else rzp += ` ${f.value.version}`
    }
    else if (f.value[v.k]) { rzk += `${f.value[v.k]},` }
  }

  mlog('createPrompt ', rz, f.value)
  rzp += ` --${f.value.speed}`
  if (f.value.sref.trim() != '')
    rzp += ` --sref ${f.value.sref}`
  if (f.value.cref.trim() != '')
    rzp += ` --cref ${f.value.cref}`
  if (f.value.cw && f.value.cw != '')
    rzp += ` --cw ${f.value.cw}`
  if (f.value.bili > -1)
    rzp += ` --ar ${vf[f.value.bili].label}`
  rz = rzk + rz + rzp
  return rz
}

// const copy=()=>{
//     copyText3( '哦们sd').then(()=>msgRef.value.showMsg('复制成功345！'));
// }
// const copy2= ()=>{
//     copyRef.value.click();
// }
function selectFile(input: any) {
  if (st.value.fileBase64.length >= 5) {
    ms.error(t('mjchat.more5sb'))
    return
  }
  upImg(input.target.files[0]).then((d: any) => {
    const index = st.value.fileBase64.findIndex(v => v == d)
    if (index > -1) {
      ms.error(t('mjchat.no2add'))
      return
    }
    st.value.fileBase64.push(d)
    fsRef.value.value = ''
  }).catch(e => msgRef.value.showError(e))
}

// 图生文
function selectFile2(input: any) {
  upImg(input.target.files[0]).then((d) => {
    mlog('f2base64>> ', d)
    const obj = {
      action: 'img2txt',
      data: {
        base64: d,
        botType: 'MID_JOURNEY',
      },
    }
    homeStore.setMyData({ act: 'draw', actData: obj })
    // input.value.value='';
    fsRef2.value.value = ''
  })
    .catch(e => msgRef.value.showError(e))
}

const same2 = () => {
  st.value.text = homeStore.myData.actData.prompt
  f.value.version = ''
  f.value.quality = ''
}
watch(() => homeStore.myData.act, (n) => {
  // n=='copy' && copy2();
  n == 'same2' && same2()
})
onMounted(() => {
  homeStore.myData.act == 'same2' && same2()
})

const exportToTxt = async () => {
  let txtContent = ''
  mlog('sss', txtContent, chatStore.$state.chat.length)

  const d = await getMjAll(chatStore.$state)
  if (d.length == 0) {
    // ms.info('暂时没作品');
    ms.info(t('mjchat.noproducet'))
    return
  }
  d.forEach((v: Chat.Chat, i: number) => {
    if (v.opt && v.opt?.status == 'SUCCESS' && v.opt?.imageUrl)
      txtContent += `${v.opt?.imageUrl}\n\n`
  })
  if (txtContent == '') {
    ms.info(t('mjchat.noproducet'))
    return
  }
  const blob = new Blob([txtContent], { type: 'text/plain' })
  const a = document.createElement('a')
  a.href = URL.createObjectURL(blob)
  a.download = t('mjchat.downloadSave')
  a.click()
  ms.success(t('mjchat.exSuccess'))
}

const clearAll = () => {
  st.value.fileBase64 = []
  st.value.text = ''
  f.value.bili = -1
  f.value.version = '--v 6.1'
  f.value.quality = ''
  f.value.shot = ''
  f.value.light = ''
  f.value.style = ''
  f.value.styles = ''
  f.value.view = ''
  f.value.cref = ''
  f.value.cw = ''
  f.value.sref = ''
  f.value.speed = 'fast'
}

const uploader = (type: string) => {
  st.value.upType = type
  fsRef3.value.click()
}
const selectFile3 = (input: any) => {
  ms.loading('上传中...')
  upImg(input.target.files[0]).then(async (d) => {
    mlog('selectFile3>> ', d)
    const data = {
      action: 'img2txt',
      data: {
        base64Array: [d],
      },
    }
    // homeStore.setMyData({act:'draw',actData:obj});
    // input.value.value='';
    try {
      d = await mjFetch('/mj/submit/upload-discord-images', data.data)
      mlog('selectFile3>> ', d)
      fsRef3.value.value = ''
      if (d.code == 1) {
        if (st.value.upType == 'cref')
          f.value.cref = d.result[0]
        else
          f.value.sref = d.result[0]

        ms.success(t('mj.uploadSuccess'))
      }
    }
    catch (e) {
      msgRef.value.showError(e)
    }
  })
    .catch(e => msgRef.value.showError(e))
}
const changeSpeed = (value: string) => {
  f.value.speed = value
//   fsRef3.value.click()
}
const getSpeed = (value: string) => {
  mlog(f.value.speed)
  if (value === f.value.speed)
    return true

  else
    return false
}
</script>

<template>
  <AiMsg ref="msgRef" />
  <input ref="fsRef" type="file" style="display: none" accept="image/jpeg, image/jpg, image/png, image/gif" @change="selectFile">
  <input ref="fsRef2" type="file" style="display: none" accept="image/jpeg, image/jpg, image/png, image/gif" @change="selectFile2">
  <input ref="fsRef3" type="file" style="display: none" accept="image/jpeg, image/jpg, image/png, image/gif" @change="selectFile3">

  <div class="overflow-y-auto bg-[#fafbfc] px-4 dark:bg-[#18181c] h-full ">
    <section class="mb-4">
      <div class="mr-1  mb-2 flex justify-between items-center">
        <div class="text-sm">
          {{ $t('mjchat.imgBili') }}
        </div>
        <div>
          <NPopover trigger="hover">
            <template #trigger>
              <SvgIcon icon="iconoir:database-export" class="text-lg cursor-pointer" @click="exportToTxt" />
            </template>
            <div>{{ $t('mjchat.imagEx') }}</div>
          </NPopover>
        </div>
      </div>
      <div class=" flex items-center justify-between space-x-1">
        <template v-for="(item, index) in vf">
          <section class="aspect-item flex-1 rounded border-2 dark:border-neutral-700 cursor-pointer" :class="{ active: index == f.bili }" @click="f.bili = index">
            <div class="aspect-box-wrapper mx-auto my-2 flex h-5 w-5 items-center justify-center">
              <div class="aspect-box rounded border-2 dark:border-neutral-700" :style="item.s" />
            </div>
            <p class="mb-1 text-center text-sm">
              {{ item.label }}
            </p>
          </section>
        </template>
      </div>
    </section>
    <section v-for=" v in farr" class="mb-4 flex justify-between items-center">
      <div>{{ v.v }}</div>
      <NSelect v-model:value="f[v.k]" :options="drawlocalized[`${v.k}List`]" size="small" class="!w-[60%]" :clearable="true" />
    </section>
    <section class="mb-4 flex justify-between items-center">
      <div class="">
        生成速度
      </div>
      <div class="!w-[60%] flex flex-row justify-between gap-3">
        <NTag type="success" round size="small" style="cursor: pointer;  " class="flex flex-row justify-center flex-1" :checked="getSpeed('relax')" :checkable="getSpeed('relax')" :bordered="false" @click="changeSpeed('relax')">
          慢速
        </NTag>
        <NTag type="success" round size="small" style="cursor: pointer;  " class="flex flex-row justify-center  flex-1" :checked="getSpeed('fast')" :checkable="getSpeed('fast')" :bordered="false" @click="changeSpeed('fast')">
          快速
        </NTag>
        <NTag v-if="f.version === '--v 6.1'" type="success" round size="small" style="cursor: pointer; " class="flex flex-row justify-center  flex-1" :checked="getSpeed('turbo')" :checkable="getSpeed('turbo')" :bordered="false" @click="changeSpeed('turbo')">
          超速
        </NTag>
      </div>
    </section>
    <!-- <template  >  </template> -->
    <section class="mb-4 flex justify-between items-center">
      <div>cw(0-100)</div>
      <NInputNumber v-model:value="f.cw" :min="0" :max="100" class="!w-[60%]" size="small" clearable placeholder="0-100 角色参考程度" />
    </section>

    <section class="mb-4 flex justify-between items-center">
      <div class="w-[45px]">
        sref
      </div>
      <NInput v-model:value="f.sref" size="small" placeholder="图片url 生成风格一致的图像" clearable>
        <template #suffix>
          <SvgIcon icon="ri:upload-line" class="cursor-pointer" @click="uploader('sref')" />
        </template>
      </NInput>
    </section>
    <section class="mb-4 flex justify-between items-center">
      <div class="w-[45px]">
        cref
      </div>
      <NInput v-model:value="f.cref" size="small" placeholder="图片url 生成角色一致的图像" clearable>
        <template #suffix>
          <SvgIcon icon="ri:upload-line" class="cursor-pointer" @click="uploader('cref')" />
        </template>
      </NInput>
    </section>

    <div class="mb-1">
      <NInput
        v-model:value="st.text" type="textarea" :placeholder="$t('mjchat.prompt')" round clearable maxlength="2000" show-count
        :autosize="{ minRows: 2, maxRows: 5 }"
      />
    </div>
    <div class="mb-4 flex justify-between items-center">
      <div class="flex justify-start items-center flex-wrap">
        <div class="pt-1 pr-1 ">
          <NPopover trigger="hover">
            <template #trigger>
              <NTag v-if="st.fileBase64.length" type="error" round size="small" style="cursor: pointer; " :bordered="false" @click="fsRef.click()">
                <div style="display: flex;">
                  <SvgIcon icon="mdi:file-chart-check-outline" /> {{ $t('mjchat.imgCYes') }}
                </div>
              </NTag>
              <NTag v-else="st.fileBase64" type="warning" round size="small" style="cursor: pointer; " :bordered="false" @click="fsRef.click()">
                <div style="display: flex;">
                  <SvgIcon icon="mdi:file-document-plus-outline" />  {{ $t('mjchat.imgCUpload') }}
                </div>
              </NTag>
            </template>
            <div style="max-width: 240px;">
              <p v-html="$t('mjchat.imgCInfo')" />

              3.<a class="text-green-500 cursor-pointer" @click="fsRef.click()" v-html="$t('mjchat.imgCadd')" /><br>
              <div v-if="st.fileBase64.length > 0" class="flex justify-start items-baseline">
                <div v-for="(v) in st.fileBase64" class="p-1">
                  <img class="w-[60px]" :src="v">
                  <br>
                  <NButton size="small" type="warning" @click="st.fileBase64 = st.fileBase64.filter((item) => item != v) ">
                    {{ $t('mjchat.del') }}
                  </NButton>
                </div>
              </div>
            </div>
          </NPopover>
        </div>

        <div class="pr-1 pt-1">
          <NPopover trigger="hover">
            <template #trigger>
              <NTag type="warning" round size="small" style="cursor: pointer; " :bordered="false" @click="fsRef2.click()">
                <div style="display: flex;">
                  <SvgIcon icon="fluent:image-edit-16-regular" />  {{ $t('mjchat.img2text') }}
                </div>
              </NTag>
            </template>
            <div style="max-width: 240px;" v-html="$t('mjchat.img2textinfo')" />
          </NPopover>
        </div>
        <div class="pt-1">
          <NTag type="success" round size="small" style="cursor: pointer; " :bordered="false" @click="shorten()">
            <div style="display: flex;">
              <SvgIcon icon="game-icons:bouncing-spring" /> Shorten
            </div>
          </NTag>
        </div>
      </div>

      <!-- <div class="flex "  v-if="$t('mjchat.imgcreate').indexOf('生成图片')!==-1">
         <n-button type="primary" :block="true" :disabled="isDisabled"  @click="create()">
            <SvgIcon icon="mingcute:send-plane-fill" />

            <template v-if="st.isLoad">{{$t('mjchat.traning')}} </template>
            <template v-else> {{$t('mjchat.imgcreate')}}</template>

        </n-button>
        </div> -->
    </div>

    <div class="flex">
      <NButton type="primary" :block="true" :disabled="isDisabled" @click="create()">
        <SvgIcon icon="mingcute:send-plane-fill" />

        <template v-if="st.isLoad">
          {{ $t('mjchat.traning') }}
        </template>
        <template v-else>
          {{ $t('mjchat.imgcreate') }}
        </template>
      </NButton>
    </div>
    <div class="flex justify-start items-center py-1">
      <div>
        <NTag type="success" round size="small" style="cursor: pointer; " :bordered="false" @click="clearAll()">
          <div style="display: flex;">
            <SvgIcon icon="ant-design:clear-outlined" />{{ $t('mj.clearAll') }}
          </div>
        </NTag>
      </div>
    </div>

    <!-- <div>
    <NDivider dashed>
        <NTag type="success" round ><div class="cursor-pointer" @click="st.showFace= !st.showFace">换脸服务</div></NTag>
    </NDivider>
    <aiFace v-if="st.showFace" />
    </div> -->
    <!-- <div class="mb-4 flex justify-between items-center">
        <div @click="copy()" ref="copyRef">复制</div>
        <div @click="copy2()"  >复制2</div>
    </div> -->

    <ul v-if="!isMobile" class="pt-4" v-html="$t('mjchat.imginfo')" />
  </div>
</template>

<style>
    .aspect-item.active, .aspect-item.active .aspect-box{
        border-color:#86dfba ;

    }
</style>
