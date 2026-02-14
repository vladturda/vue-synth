<template>
  <div 
    class="key"
    :class="[type, { active: isActive }]"
    @mousedown="$emit('play')"
    @mouseup="$emit('stop')"
    @mouseleave="$emit('stop')"
  >
    <span class="key-label">{{ label }}</span>
  </div>
</template>

<script setup>
defineProps({
  type: {
    type: String,
    required: true
  },
  label: {
    type: String,
    required: true
  },
  isActive: {
    type: Boolean,
    default: false
  }
})

defineEmits(['play', 'stop'])
</script>

<style scoped>
.key {
  cursor: pointer;
  position: absolute;
  top: 20px;
  transition: all 0.1s ease;
}

.key.white {
  width: 50px;
  height: 180px;
  background: linear-gradient(180deg, #f5f5f5 0%, #e0e0e0 100%);
  border: 1px solid #ccc;
  border-radius: 0 0 5px 5px;
  box-shadow: 
    0 5px 15px rgba(0, 0, 0, 0.3),
    inset 0 -5px 10px rgba(0, 0, 0, 0.1);
}

.key.white:hover {
  background: linear-gradient(180deg, #ffffff 0%, #f0f0f0 100%);
}

.key.white:active, .key.white.active {
  background: linear-gradient(180deg, #00d9ff 0%, #0099cc 100%);
  transform: translateY(3px);
  box-shadow: 
    0 2px 10px rgba(0, 217, 255, 0.5),
    inset 0 -2px 5px rgba(0, 0, 0, 0.2);
}

.key.black {
  width: 35px;
  height: 110px;
  background: linear-gradient(180deg, #2a2a2a 0%, #1a1a1a 100%);
  border: 1px solid #000;
  z-index: 2;
  box-shadow: 
    0 5px 15px rgba(0, 0, 0, 0.5),
    inset 0 -3px 5px rgba(0, 0, 0, 0.5);
}

.key.black:hover {
  background: linear-gradient(180deg, #3a3a3a 0%, #2a2a2a 100%);
}

.key.black:active, .key.black.active {
  background: linear-gradient(180deg, #ff006e 0%, #cc0058 100%);
  transform: translateY(3px);
  box-shadow: 
    0 2px 10px rgba(255, 0, 110, 0.5),
    inset 0 -2px 5px rgba(0, 0, 0, 0.3);
}

.key-label {
  position: absolute;
  bottom: 15px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 10px;
  color: #666;
  text-transform: uppercase;
  pointer-events: none;
}

.key.black .key-label {
  color: #666;
  font-size: 9px;
}
</style>
