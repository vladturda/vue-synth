<template>
  <div class="keyboard">
    <Key
      v-for="key in keys"
      :key="key.note"
      :type="key.type"
      :label="key.label"
      :is-active="activeNotes[key.note]"
      :style="{ left: key.position + 'px' }"
      @playNote="$emit('playNote', key.note)"
      @stopNote="$emit('stopNote', key.note)"
    />
  </div>
</template>

<script setup>
import { computed } from 'vue';
import Key from './Key.vue';

const props = defineProps({
  activeNotes: {
    type: Object,
    default: () => ({})
  }
});

defineEmits(['playNote', 'stopNote']);

const keyDefinitions = [
  { note: 'C3', type: 'white', label: 'C' },
  { note: 'C#3', type: 'black', label: 'C#' },
  { note: 'D3', type: 'white', label: 'D' },
  { note: 'D#3', type: 'black', label: 'D#' },
  { note: 'E3', type: 'white', label: 'E' },
  { note: 'F3', type: 'white', label: 'F' },
  { note: 'F#3', type: 'black', label: 'F#' },
  { note: 'G3', type: 'white', label: 'G' },
  { note: 'G#3', type: 'black', label: 'G#' },
  { note: 'A3', type: 'white', label: 'A' },
  { note: 'A#3', type: 'black', label: 'A#' },
  { note: 'B3', type: 'white', label: 'B' },
  { note: 'C4', type: 'white', label: 'C' },
  { note: 'C#4', type: 'black', label: 'C#' },
  { note: 'D4', type: 'white', label: 'D' },
  { note: 'D#4', type: 'black', label: 'D#' },
  { note: 'E4', type: 'white', label: 'E' },
  { note: 'F4', type: 'white', label: 'F' },
  { note: 'F#4', type: 'black', label: 'F#' },
  { note: 'G4', type: 'white', label: 'G' },
  { note: 'G#4', type: 'black', label: 'G#' },
  { note: 'A4', type: 'white', label: 'A' },
  { note: 'A#4', type: 'black', label: 'A#' },
  { note: 'B4', type: 'white', label: 'B' }
];

const whiteKeyWidth = 50;
const blackKeyWidth = 35;

const keys = computed(() => {
  let whiteKeyIndex = 0;
  return keyDefinitions.map((key, index) => {
    let position;

    if (key.type === 'white') {
      position = whiteKeyIndex * whiteKeyWidth;
      whiteKeyIndex++;
    } else {
      position = whiteKeyIndex * whiteKeyWidth - blackKeyWidth / 2;
    }
    
    return {
      ...key,
      position
    };
  })
});
</script>

<style scoped>
.keyboard {
  display: block;
  position: relative;
  padding: 30px;
  background: linear-gradient(180deg, #1a1a2e, #0f0f1e);
  border-radius: 10px;
  box-shadow: inset 0 5px 15px rgba(0, 0, 0, 0.5);
  height: 240px;
  width: 760px;
}
</style>
