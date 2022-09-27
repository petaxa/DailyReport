<script setup lang="ts">
import router from '@/router'
import { ref } from 'vue'

// 作業時間、作業予定(複数)
/**
 * 本日の業務開始します。
 * 11:00~17:00, 21:00~23:00で作業予定です
 * 【作業予定】
 * OPAL-Tutorialの作成
 * 新会社売上, 収支計算
 */
const workTimeAry = ref([
  { startTime: '11:00', endTime: '17:00' },
  { startTime: '21:00', endTime: '23:00' }
])
const willWorkAry = ref(['', ''])

// 今回増加
const clickTodayPlus = () => {
  willWorkAry.value.push('')
}
// 今回減少
const clickTodayMin = () => {
  willWorkAry.value.pop()
}

// 今回増加
const clickTodayTimePlus = () => {
  workTimeAry.value.push({ startTime: '', endTime: '' })
}
// 今回減少
const clickTodayTimeMin = () => {
  workTimeAry.value.pop()
}
// 本文
const msg = ref('')

const clickCreateBtn = () => {
  msg.value = `本日の業務開始します。\n${createTimesMsg()}で作業予定です\n【作業予定】\n${willWorkAry.value.join('\n')}`
  // クリップボードにコピー
  // コピー内容を選択する.
  const output = document.getElementById('output')
  setTimeout(async () => {
    // textContntからじゃないとコピーが上手くいかない。
    await navigator.clipboard.writeText(output?.textContent ?? 'can not be copied')
    console.log('copied')
  }, 1000)
}

// メッセージ用にworkTimeAryを成型
const createTimesMsg = () => {
  return workTimeAry.value
    .map(times => `${times.startTime}~${times.endTime}`)
    .join(' ')
}

// 戻るボタン押下時処理
const clickBack = () => {
  router.push('/')
}

// クリアボタン押下処理
const clickClear = () => {
  willWorkAry.value.forEach((w, index) => {
    willWorkAry.value[index] = ''
  })
  // 作業時間をクリア
  workTimeAry.value = [
    { startTime: '11:00', endTime: '17:00' },
    { startTime: '21:00', endTime: '23:00' }
  ]
}
</script>

<template>
  <div class="button">
    <button id="TimePlus" class="formButton" @click="clickTodayTimePlus">+</button>
    <button id="TimeMin" class="formButton" @click="clickTodayTimeMin">-</button>
  </div>
  <div v-for="(i, index) of workTimeAry" :key="index">
    <input type="time" v-model="i.startTime" class="time" />
    <input type="time" v-model="i.endTime" class="time" />
  </div>
  <div class="button">
    <button id="workedPlus" class="formButton" @click="clickTodayPlus">+</button>
    <button id="workedMin" class="formButton" @click="clickTodayMin">-</button>
  </div>
  <div class="form" v-for="(item, index) of willWorkAry" :key="index">
    <input type="text" v-model="willWorkAry[index]" class="work" />
  </div>

  <div>
    <button id="create" @click="clickCreateBtn" class="finButton">作成</button>
    <button @click="clickBack" class="finButton">戻る</button>
    <button @click="clickClear" class="finButton">クリア</button>
  </div>
  <p id="output">{{msg}}</p>
</template>