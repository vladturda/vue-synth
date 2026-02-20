<template>
  <div 
    class="note"
    :class="{ 'is-playing': playing }"
    :style="{ 
      left: (start / totalBeats * 100) + '%',
      width: (duration / totalBeats * 100) + '%'
    }"
    @mousedown.stop="removeNote"
  ></div>
</template>

<script setup>
defineProps({
  start: {
    type: Number,
    required: true
  },
  duration: {
    type: Number,
    required: true
  },
  playing: {
    type: Boolean,
    default: false
  },
  totalBeats: {
    type: Number,
    required: true
  }
});

const emit = defineEmits(['removeNote']);

function removeNote() {
  emit('removeNote');
}
</script>

<style scoped>
.note {
  position: absolute;
  top: 10%;
  height: 80%;
  background: #00d9ff;
  border-radius: 2px;
  cursor: pointer;
  transition: background 0.1s;
  z-index: 3;
}

.note:hover {
  background: #ff006e;
}

.note.is-playing {
  background: #00ff88;
  box-shadow: 0 0 5px #00ff88;
}
</style>
