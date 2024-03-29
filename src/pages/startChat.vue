<script setup lang="ts">
import { getSetting, getWorkedAry, getWorkedTimeAry, setWorkedAry, setWorkedTimeAry } from '@/lib/localStorage'
import { delay } from '@/lib/time'
import router from '@/router'
import copiedModal from '../components/copiedModal.vue'
import { ref } from 'vue'

// 作業時間、作業予定(複数)
/**
 * 本日の業務開始します。
 * 11:00~17:00, 21:00~23:00で作業予定です
 * 【作業予定】
 * 新会社売上, 収支計算
 */
// 設定取得
const isAutocomplete = !!getSetting('isAutocomplete') // null回避
// localStorageの次回作業内容を初期値とする
const nextWorkedAry = getWorkedAry('nextWorkedAry')
const nextTimeAry = getWorkedTimeAry('nextTimeAry')

// 作業内容
const willWorkAry = nextWorkedAry && isAutocomplete ? ref(nextWorkedAry) : ref(['', ''])
// 作業時間
const workTimeAry = nextTimeAry && isAutocomplete ? ref(nextTimeAry) : ref([
  { startTime: '11:00', endTime: '17:00' },
  { startTime: '21:00', endTime: '23:00' }
])

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
  // localStorageに保存
  setWorkedAry('willWorkAry', willWorkAry.value)
  setWorkedTimeAry('workTimeAry', workTimeAry.value)
  // クリップボードにコピー
  // コピー内容を選択する.
  const output = document.getElementById('output')
  setTimeout(async () => {
    // textContntからじゃないとコピーが上手くいかない。
    await navigator.clipboard.writeText(output?.textContent ?? 'can not be copied')
    await showCopiedModal()
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
  // localStorageのwillWorkAryをクリア
  setWorkedAry('nextWorkedAry', ['', ''])
  setWorkedTimeAry('workTimeAry', [
    { startTime: '11:00', endTime: '17:00' },
    { startTime: '21:00', endTime: '23:00' }
  ])
}

// コピー完了モーダル表示処理
const isCopiedModal = ref(false)
const showCopiedModal = async () => {
  isCopiedModal.value = true
  await delay(1)
  isCopiedModal.value = false
}
</script>

<template>
  <button @click="clickBack" class="icon">
    <font-awesome-icon icon="fa-regular fa-arrow-alt-circle-left" />
  </button>
  <div id="startChat">
    <p class="page-title">作業開始文言作成</p>
    <div class="line-parent">
      <div class="line"></div>
    </div>
    <div id="main-input">
      <div class="button">
        <span>作業予定時間</span>
        <button id="TimePlus" class="formButton" @click="clickTodayTimePlus">+</button>
        <button id="TimeMin" class="formButton" @click="clickTodayTimeMin">-</button>
      </div>
      <div v-for="(i, index) of workTimeAry" :key="index">
        <input type="time" v-model="i.startTime" class="time" />
        <span> ~ </span>
        <input type="time" v-model="i.endTime" class="time" />
      </div>
      <div class="button">
        <span>作業予定内容</span>
        <button id="workedPlus" class="formButton" @click="clickTodayPlus">+</button>
        <button id="workedMin" class="formButton" @click="clickTodayMin">-</button>
      </div>
      <div class="form" v-for="(item, index) of willWorkAry" :key="index">
        <input type="text" v-model="willWorkAry[index]" class="work" />
      </div>

      <div>
        <button id="create" @click="clickCreateBtn" class="finButton">作成</button>
        <button @click="clickClear" class="finButton">クリア</button>
      </div>
    </div>
    <p id="output">{{ msg }}</p>
  </div>
  <copiedModal v-if="isCopiedModal" class="copiedModal"></copiedModal>
</template>

<style scoped>
/* 画面全体 */
#startChat {
  margin: 140px 0px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

/* コンテンツ部分 */
.startChat-backBtn {
  width: 100px;
}

/* タイトル */
.page-title {
  font-size: 35px;
}

/* 入力欄 */
#main-input {
  margin-left: 10rem;
  width: 25em;
  display: flex;
  flex-direction: column;
  align-items: left;
}

/* 作業内容入力欄 */
.work {
  margin-top: 3px;
}

/* コピー用領域非表示 */
#output {
  display: none;
}
</style>
