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
  },
  octave: {
    type: Number,
    default: 4
  }
});

defineEmits(['playNote', 'stopNote']);

const keyRoll = [ 'C', 'C#', 'D', 'D#', 'E', 'F', 'F#', 'G', 'G#', 'A', 'A#', 'B'];
const keyDefinitions = computed(() => {
  const twoOctaves = [];
  for (let currentOctave of [ props.octave-1, props.octave ]) {
    for (let currentNote of keyRoll) {
      twoOctaves.push({
        note: currentNote + currentOctave,
        type: currentNote.includes('#') ? 'black' : 'white',
        label: currentNote
      });
    }
  }
  return twoOctaves;
});

const whiteKeyWidth = 50;
const blackKeyWidth = 35;

const keys = computed(() => {
  let whiteKeyIndex = 0;
  return keyDefinitions.value.map((key, index) => {
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
