<template>
  <div class="knob-container">
    <div 
      class="knob" 
      :style="{ transform: `rotate(${rotation}deg)` }"
      @mousedown="startDrag"
    ></div>
  </div>
  <span class="knob-value">{{ displayValue }}</span>
</template>

<script setup>
import { computed, ref } from 'vue'

const props = defineProps({
  modelValue: {
    type: Number,
    required: true
  },
  min: {
    type: Number,
    default: 0
  },
  max: {
    type: Number,
    default: 1
  },
  step: {
    type: Number,
    default: 0.01
  },
  format: {
    type: String,
    default: ''
  }
})

const emit = defineEmits(['update:modelValue'])

const isDragging = ref(false)
const startY = ref(0)
const startValue = ref(0)

const normalizedValue = computed(() => {
  return (props.modelValue - props.min) / (props.max - props.min)
})

const rotation = computed(() => {
  return normalizedValue.value * 270 - 135
})

const displayValue = computed(() => {
  if (props.format === 's') {
    return props.modelValue.toFixed(2) + 's'
  }
  return Math.round(props.modelValue)
})

function startDrag(e) {
  isDragging.value = true
  startY.value = e.clientY
  startValue.value = normalizedValue.value
  
  document.addEventListener('mousemove', onDrag)
  document.addEventListener('mouseup', stopDrag)
}

function onDrag(e) {
  if (!isDragging.value) return
  
  const delta = (startY.value - e.clientY) / 100
  let newValue = startValue.value + delta
  newValue = Math.max(0, Math.min(1, newValue))
  
  const rawValue = props.min + newValue * (props.max - props.min)
  const steppedValue = Math.round(rawValue / props.step) * props.step
  
  emit('update:modelValue', steppedValue)
}

function stopDrag() {
  isDragging.value = false
  document.removeEventListener('mousemove', onDrag)
  document.removeEventListener('mouseup', stopDrag)
}
</script>

<style scoped>
.knob-container {
  position: relative;
  width: 70px;
  height: 70px;
}

.knob {
  width: 70px;
  height: 70px;
  border-radius: 50%;
  background: conic-gradient(from 225deg, #00d9ff, #ff006e, #00d9ff);
  cursor: pointer;
  position: relative;
  box-shadow: 
    0 5px 15px rgba(0, 0, 0, 0.4),
    inset 0 2px 4px rgba(255, 255, 255, 0.2);
  transition: transform 0.05s ease-out;
}

.knob::before {
  content: '';
  position: absolute;
  top: 8px;
  left: 8px;
  right: 8px;
  bottom: 8px;
  border-radius: 50%;
  background: linear-gradient(145deg, #2a2a4a, #1a1a2e);
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.5);
}

.knob::after {
  content: '';
  position: absolute;
  top: 12px;
  left: 50%;
  width: 4px;
  height: 12px;
  background: #00d9ff;
  border-radius: 2px;
  transform-origin: center 23px;
  transform: translateX(-50%);
  box-shadow: 0 0 8px #00d9ff;
}

.knob-value {
  font-size: 14px;
  font-weight: bold;
  color: #00d9ff;
  text-align: center;
  margin-top: 5px;
}
</style>
