<template>
    <template v-if="init">
        <Screen v-if="screening" />
        <Editor v-else-if="_isPC" />
        <Mobile v-else />
    </template>
</template>

<script lang="ts" setup>
import { onMounted, ref } from 'vue'
import { storeToRefs } from 'pinia'
import { useScreenStore, useMainStore, useSnapshotStore, useSlidesStore } from '@/store'
import { LOCALSTORAGE_KEY_DISCARDED_DB } from '@/configs/storage'
import { deleteDiscardedDB } from '@/utils/database'
import { isPC } from './utils/common'
import {useRoute} from 'vue-router'
const slidesStore = useSlidesStore()

import Editor from './views/Editor/index.vue'
import Screen from './views/Screen/index.vue'
import Mobile from './views/Mobile/index.vue'

const _isPC = isPC()
const init = ref(false)
const route = useRoute()

const mainStore = useMainStore()
const snapshotStore = useSnapshotStore()
const { databaseId } = storeToRefs(mainStore)
const { screening } = storeToRefs(useScreenStore())

if (process.env.NODE_ENV === 'production') {
  window.onbeforeunload = () => false
}

async function getDetail(url) {
  const response = await fetch(
    url
  )
  if (!response.ok) {
    throw new Error('Network response was not ok')
  }
  const data = await response.text()
  

  slidesStore.setSlides(JSON.parse(data))
  init.value = true
}

onMounted(async () => {
  await deleteDiscardedDB()
  const urlParams = new URLSearchParams(window.location.search)
  const url = urlParams.get('url') 
 
  await getDetail(url)
  snapshotStore.initSnapshotDatabase()
  mainStore.setAvailableFonts()
})

// 应用注销时向 localStorage 中记录下本次 indexedDB 的数据库ID，用于之后清除数据库
window.addEventListener('unload', () => {
  const discardedDB = localStorage.getItem(LOCALSTORAGE_KEY_DISCARDED_DB)
  const discardedDBList: string[] = discardedDB ? JSON.parse(discardedDB) : []

  discardedDBList.push(databaseId.value)

  const newDiscardedDB = JSON.stringify(discardedDBList)
  localStorage.setItem(LOCALSTORAGE_KEY_DISCARDED_DB, newDiscardedDB)
})
</script>

<style lang="scss">
#app {
    height: 100%;
}
</style>
