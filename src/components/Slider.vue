<template>
  <div class="slider-container">
    <input 
      type="range" 
      class="slider" 
      :min="min" 
      :max="max" 
      :step="step"
      :value="model"
      @input="model = parseFloat($event.target.value)"
    >
  </div>
  <span class="slider-value" v-if="showValue">{{ displayValue }}</span>
</template>

<script setup>
import { computed } from 'vue';

const model = defineModel({
  type: Number,
  required: true
});

const props = defineProps({
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
  showValue: {
    type: Boolean,
    default: false,
  },
  format: {
    type: String,
    default: ''
  }
});

const displayValue = computed(() => {
  if (props.format === 'percent') {
    return Math.round(model.value * 100) + '%';
  }
  if (props.format === 's') {
    return model.value.toFixed(2) + 's';
  }
  return model.value;
});
</script>

<style scoped>
.slider-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
  margin: 12px 0;
}

.slider {
  -webkit-appearance: none;
  appearance: none;
  width: 120px;
  height: 8px;
  border-radius: 4px;
  background: linear-gradient(90deg, #1a1a2e, #2a2a4a);
  outline: none;
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.5);
}

.slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: linear-gradient(145deg, #3a3a5a, #2a2a4a);
  cursor: pointer;
  box-shadow: 
    0 3px 10px rgba(0, 0, 0, 0.4),
    inset 0 1px 0 rgba(255, 255, 255, 0.2);
  border: 2px solid #00d9ff;
}

.slider::-moz-range-thumb {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: linear-gradient(145deg, #3a3a5a, #2a2a4a);
  cursor: pointer;
  box-shadow: 0 3px 10px rgba(0, 0, 0, 0.4);
  border: 2px solid #00d9ff;
}

.slider-value {
  font-size: 14px;
  font-weight: bold;
  color: #00d9ff;
}
</style>
