<script setup>
import { reactive, ref } from 'vue'
const wheel = ref(null)
const spinButton = ref(null)
const startGameButton = ref(null)
const participantInput = ref(null)

const colors = [
  '#ff6384',
  '#36a2eb',
  '#ffce56',
  '#4bc0c0',
  '#9966ff',
  '#ff9f40',
  '#c9cbcf',
  '#000000',
]

let segments = reactive([])
let currentRotation = 0
let initialSegments = []

function createWheelGradient(participantCount) {
  const segmentAngle = 360 / participantCount
  const gradientStops = segments.map((segment, index) => {
    const startAngle = index * segmentAngle
    const endAngle = (index + 1) * segmentAngle
    return `${segment.color} ${startAngle}deg ${endAngle}deg`
  })

  wheel.value.style.background = `conic-gradient(${gradientStops.join(', ')})`
}

function spinWheel() {
  const randomSegmentIndex = Math.floor(Math.random() * segments.length)
  const segmentAngle = 360 / segments.length
  const targetRotation = randomSegmentIndex * segmentAngle + 360 * 10 // 轉動至少 10 圈
  currentRotation += targetRotation

  wheel.value.style.transform = `rotate(${currentRotation}deg)`

  setTimeout(() => {
    const selectedSegment = segments[randomSegmentIndex]
    alert(`選中：${selectedSegment.label}`)
    segments.splice(randomSegmentIndex, 1)

    if (segments.length > 1) {
      createWheelGradient(segments.length) // 更新輪盤
    } else {
      alert('遊戲結束！重新開始！')
      resetGame()
    }
  }, 4000)
}

function resetGame() {
  wheel.value.style.transform = 'rotate(0deg)'
  wheel.value.style.background = '' // 清空背景
  spinButton.value.style.display = 'none'
  startGameButton.value.style.display = 'block'
  participantInput.value = ''
  segments = [...initialSegments] // 恢復原始分片
}

const start = () => {
  const participantCount = parseInt(participantInput.value)
  if (isNaN(participantCount) || participantCount < 2 || participantCount > 8) {
    alert('請輸入有效的參與人數 (2-8)')
    return
  }

  const shuffledColors = [...colors].sort(() => Math.random() - 0.5).slice(0, participantCount)
  initialSegments = shuffledColors.map((color, index) => {
    segments.push({ color, label: `玩家 ${index + 1}` })
  })

  createWheelGradient(participantCount)
  startGameButton.value.style.display = 'none'
  spinButton.value.style.display = 'block'
}
</script>

<template>
  <div style="display: flex; justify-content: space-evenly">
    <div v-for="segment in segments" :key="segment.color">
      <span>{{ segment.label }}</span>
      <div :style="{ width: '20px', height: '20px', 'background-color': segment.color }"></div>
    </div>
  </div>

  <div>
    <span>輸入參與人數 (2-8):</span>
    <input
      type="number"
      min="2"
      max="8"
      placeholder="輸入參與人數 (2-8)"
      v-model="participantInput"
    />
    <div>
      <button id="startGameButton" @click="start" ref="startGameButton">開始遊戲</button>
    </div>
  </div>
  <div class="wheel-container">
    <div class="wheel" id="wheel" ref="wheel"></div>
    <div class="pointer"></div>
  </div>
  <button id="spinButton" style="display: none" @click="spinWheel" ref="spinButton">
    轉動轉盤
  </button>
</template>

<style scoped></style>
