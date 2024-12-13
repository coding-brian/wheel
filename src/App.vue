<script setup>
import { reactive, ref } from 'vue'
import confetti from 'canvas-confetti'

const wheel = ref(null)
const participantInput = ref(null)
const style = ref({
  transition: 'transform 4s cubic-bezier(0.25, 0.1, 0.25, 1)',
})
const notify = ref({
  isShow: false,
  message: 'test',
  title: '',
  isShowButton: true,
  color: 'black',
})

const colors = [
  '#ff6384',
  '#36a2eb',
  '#ffce56',
  '#4bc0c0',
  '#9966ff',
  '#ff9f40',
  '#c9cbcf',
  '#ffffff',
]
const buttonGroup = reactive({
  start: {
    isShow: true,
  },
  spin: {
    isShow: false,
  },
})

let segments = reactive([])

const createWheelGradient = (participantCount) => {
  // 遊戲結束
  if (segments.length <= 1) {
    notify.value.title = '遊戲結束！1秒後重新開始！'
    notify.value.message = ''
    notify.value.isShow = true
    notify.value.isShowButton = false
    setTimeout(() => {
      resetGame()
      resetNotify()
    }, 1000)
    return
  }

  const segmentAngle = 360 / participantCount
  const gradientStops = segments.map((segment, index) => {
    const startAngle = index * segmentAngle
    const endAngle = (index + 1) * segmentAngle
    return `${segment.color} ${startAngle}deg ${endAngle}deg`
  })

  wheel.value.style.background = `conic-gradient(${gradientStops.join(', ')})`
  style.value.transition = 'none'
  wheel.value.style.transform = `rotate(0deg)`
  setTimeout(() => {
    style.value.transition = 'transform 4s cubic-bezier(0.25, 0.1, 0.25, 1)'
  }, 10)
}

const spinWheel = () => {
  if (!buttonGroup.spin.isShow) {
    return
  }

  const selectedPlayer = Math.floor(Math.random() * segments.length + 1)

  const segmentAngle = 360 / segments.length
  const targetRotation = -(selectedPlayer * segmentAngle - segmentAngle / 2 - 360 * 10) //轉動至少 10 圈

  wheel.value.style.transform = `rotate(${targetRotation}deg)`

  setTimeout(() => {
    const segmentIndex = selectedPlayer - 1
    const selectedSegment = segments[segmentIndex]
    segments.splice(segmentIndex, 1)

    confetti({
      particleCount: 100,
      spread: 70,
      origin: { y: 0.6 },
    })

    notify.value.message = `${selectedSegment.name}`
    notify.value.isShow = true
    notify.value.title = '恭喜選中'
    notify.value.color = selectedSegment.color
  }, 4000)
}

const resetGame = () => {
  wheel.value.style.transform = 'rotate(0deg)'
  wheel.value.style.background = '' // 清空背景
  buttonGroup.spin.isShow = false
  buttonGroup.start.isShow = true
  participantInput.value = ''
  segments.splice(0, segments.length) // 恢復原始分片
}

const start = () => {
  const participantCount = parseInt(participantInput.value)
  if (isNaN(participantCount) || participantCount < 2 || participantCount > 8) {
    alert('請輸入有效的參與人數 (2-8)')
    return
  }

  if (!buttonGroup.start.isShow) {
    return
  }

  buildSegment(participantCount)
  createWheelGradient(participantCount)
  buttonGroup.start.isShow = false
  buttonGroup.spin.isShow = true
}

const buildSegment = (participantCount) => {
  const shuffledColors = [...colors].sort(() => Math.random() - 0.5).slice(0, participantCount)
  shuffledColors.forEach((color, index) => {
    segments.push({ color, name: `玩家 ${index + 1}` })
  })
}

const resetNotify = () => {
  notify.value.isShow = false
  notify.value.message = ''
  notify.value.isShowButton = true
  notify.value.title = ''
  notify.value.color = 'black'
}

const confirm = () => {
  resetNotify()
  createWheelGradient(segments.length) // 更新輪盤
}
</script>

<template>
  <div>
    <span>輸入參與人數 (2-8)</span>
    <input
      type="number"
      min="2"
      max="8"
      placeholder="輸入參與人數 (2-8)"
      v-model="participantInput"
    />
    <div class="player-container">
      <div v-for="segment in segments" :key="segment.color">
        <input
          type="text"
          name=""
          id=""
          v-model="segment.name"
          :style="{ 'background-color': segment.color }"
        />
      </div>
    </div>
  </div>
  <div class="wheel-container">
    <div class="wheel" id="wheel" ref="wheel" :style="style"></div>
    <div class="pointer"></div>
  </div>
  <div class="button-container">
    <button
      :class="{
        'opacity-70': !buttonGroup.start.isShow,
        'cursor-not-allowed': !buttonGroup.start.isShow,
      }"
      @click="start"
    >
      開始遊戲
    </button>
    <button
      :class="{
        'opacity-70': !buttonGroup.spin.isShow,
        'cursor-not-allowed': !buttonGroup.spin.isShow,
      }"
      @click="spinWheel"
    >
      轉動轉盤
    </button>
    <div>
      <button @click="resetGame">重置</button>
    </div>
  </div>
  <div class="notify-container" v-if="notify.isShow">
    <div class="notify">
      <p v-if="notify.title">
        <span>{{ notify.title }}</span>
      </p>
      <span
        :style="{ color: notify.color, 'font-size': '24px', 'font-weight': 'bold' }"
        v-if="notify.message"
        >{{ notify.message }}</span
      >
      <button @click="confirm" v-if="notify.isShowButton">確認</button>
    </div>
  </div>
</template>

<style scoped></style>
