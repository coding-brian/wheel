<script setup>
import { reactive, ref } from 'vue'
const wheel = ref(null)
const participantInput = ref(null)

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
let currentRotation = 0

const createWheelGradient = (participantCount) => {
  const segmentAngle = 360 / participantCount
  const gradientStops = segments.map((segment, index) => {
    const startAngle = index * segmentAngle
    const endAngle = (index + 1) * segmentAngle
    return `${segment.color} ${startAngle}deg ${endAngle}deg`
  })

  wheel.value.style.background = `conic-gradient(${gradientStops.join(', ')})`
}

const spinWheel = () => {
  if (!buttonGroup.spin.isShow) {
    return
  }

  const selectedPlayer = Math.floor(Math.random() * segments.length + 1)

  const segmentAngle = 360 / segments.length
  const targetRotation = -(selectedPlayer * segmentAngle - segmentAngle / 2 - 360 * 10) //轉動至少 10 圈
  currentRotation += targetRotation

  wheel.value.style.transform = `rotate(${currentRotation}deg)`

  setTimeout(() => {
    const segmentIndex = selectedPlayer - 1
    const selectedSegment = segments[segmentIndex]
    alert(`選中：${selectedSegment.name}`)
    segments.splice(segmentIndex, 1)

    if (segments.length > 1) {
      createWheelGradient(segments.length) // 更新輪盤
    } else {
      alert('遊戲結束！重新開始！')
      resetGame()
    }
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
</script>

<template>
  <div>
    <div>
      <button @click="resetGame">重置</button>
    </div>

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
    <div class="wheel" id="wheel" ref="wheel"></div>
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
  </div>
</template>

<style scoped></style>
