<template>
  <div class="container">
    <h1 style="text-align: center; margin-bottom: 2rem;">穿越时间（资本版）</h1>
    <audio ref="bgmRef" src="/bgm.mp3" autoplay loop />

    <!-- 选择框 -->
    <el-select v-model="selectedTime" placeholder="选择穿越时间" style="width: 100%; margin-bottom: 1rem">
      <el-option label="穿越1秒" :value="1" />
      <el-option label="穿越10秒" :value="10" />
      <el-option label="穿越100秒" :value="100" />
      <el-option label="自定义穿越" value="custom" />
    </el-select>

    <!-- 自定义输入 -->
    <el-input-number v-if="selectedTime === 'custom'" v-model="customSeconds" :min="0.1" :step="0.1" label="自定义秒数"
      placeholder="请输入秒数" style="width: 100%; margin-bottom: 1rem" />

    <!-- 自定义进度条 -->
    <div v-if="isTraveling" class="progress-container">
      <svg class="progress-ring" width="200" height="200">
        <circle class="progress-ring__background" cx="100" cy="100" r="90" />
        <circle class="progress-ring__circle" :stroke="currentColor" cx="100" cy="100" r="90"
          :stroke-dasharray="strokeDasharray" :stroke-dashoffset="strokeDashoffset" />
        <text x="50%" y="50%" text-anchor="middle" dy="0.3em" font-size="20" fill="#333">
          {{ formatProgress(percentRef) }}
        </text>
      </svg>
      <div style="font-size: 1.1rem; margin-top: 1rem;">已穿越 {{ elapsedTime }} ms</div>
      <el-button type="danger" style="margin-top: 1rem" @click="stopTravel">停止</el-button>
    </div>

    <!-- 开始按钮 -->
    <div v-else style="text-align: center; margin-top: 2rem">
      <el-button type="primary" @click="startTravel">开始穿越</el-button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'


// 状态变量
const selectedTime = ref<number | 'custom' | ''>('')         // 用户选择的秒数
const customSeconds = ref(1)                                 // 自定义秒数
const isTraveling = ref(false)                               // 是否正在穿越
const percentRef = ref(0)                                    // 进度百分比
const elapsedTime = ref(0)                                   // 已穿越时间（毫秒）

let timer: number | undefined = undefined                    // 定时器句柄

// 颜色分段
const colors = [
  { color: '#f56c6c', percentage: 20 },
  { color: '#e6a23c', percentage: 40 },
  { color: '#5cb87a', percentage: 60 },
  { color: '#1989fa', percentage: 80 },
  { color: '#6f7ad3', percentage: 100 },
]

// 启动穿越
const startTravel = () => {
  const seconds = selectedTime.value === 'custom' ? customSeconds.value : selectedTime.value as number

  if (!seconds || seconds <= 0) {
    ElMessage.error('请输入有效的时间！')
    return
  }

  isTraveling.value = true
  percentRef.value = 0
  elapsedTime.value = 0

  const totalMs = seconds * 1000
  const interval = 16 // 每帧间隔时间（毫秒）
  const step = (interval / totalMs) * 100 // 每次增长的百分比

  timer = window.setInterval(() => {
    percentRef.value += step
    elapsedTime.value += interval

    if (percentRef.value >= 100) {
      percentRef.value = 100
      elapsedTime.value = totalMs
      clearInterval(timer)
      timer = undefined
      isTraveling.value = false

      // 弹窗提示
      ElMessageBox.alert(`你被资本骗了 ${seconds} 秒`, '穿越结束', {
        confirmButtonText: '好吧',
        type: 'warning',
      })

      reset()
    }
  }, interval)
}

// 停止穿越
const stopTravel = () => {
  if (timer !== undefined) {
    clearInterval(timer)
    timer = undefined
  }
  isTraveling.value = false
  reset()
}

// 百分比格式化函数
const formatProgress = (percent: number) => `${percent.toFixed(2)}%`

// 重置表单
const reset = () => {
  percentRef.value = 0
  elapsedTime.value = 0
  selectedTime.value = ''
  customSeconds.value = 1
}

// 圆形进度条计算
const RADIUS = 90
const CIRCUMFERENCE = 2 * Math.PI * RADIUS
const strokeDasharray = `${CIRCUMFERENCE}`
const strokeDashoffset = computed(() => {
  return CIRCUMFERENCE - (percentRef.value / 100) * CIRCUMFERENCE
})

const currentColor = computed(() => {
  for (const c of colors) {
    if (percentRef.value <= c.percentage) {
      return c.color
    }
  }
  return colors[colors.length - 1].color
})
</script>

<style scoped>
* {
  box-sizing: border-box;
}

.container {
  padding: 2rem;
  max-width: 500px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.progress-container {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.progress-ring {
  transform: rotate(0deg);
}

.progress-ring__background {
  fill: none;
  stroke: #eee;
  stroke-width: 12;
}

.progress-ring__circle {
  fill: none;
  stroke-width: 12;
  transition: none;
}

@media (max-width: 600px) {
  .container {
    padding: 1rem;
  }
}
</style>
