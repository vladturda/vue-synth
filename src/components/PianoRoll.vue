<template>
  <div class="piano-roll">
    <div class="piano-roll-header">
      <div class="piano-roll-controls">
        <button class="roll-btn" @click="togglePlay">
          {{ isPlaying ? 'Stop' : 'Play' }}
        </button>

        <button class="roll-btn" @click="clearNotes">Clear</button>

        <div class="tempo-control">
          <label>Tempo</label>
          <Slider v-model.number="tempo" :min="60" :max="200" :step="1"/>
          <span class="tempo-value">{{ tempo }} BPM</span>
        </div>
      </div>
    </div>
    
    <div class="piano-roll-container">
      <div class="piano-keys">
        <div 
          v-for="note in displayNotes" 
          :key="note"
          class="piano-key-row"
          :class="{ 'is-black': note.includes('#') }"
        >
          <span class="note-label">{{ note }}</span>
        </div>
      </div>
      
      <div class="roll-grid" ref="gridRef">
        <div class="grid-content">
          <div class="beat-lines">
            <div 
              v-for="beat in totalBeats" 
              :key="beat"
              class="beat-line"
              :class="{ 'major': beat % 4 === 1 }"
              :style="{ left: ((beat - 1) / totalBeats * 100) + '%' }"
            >
              <span class="beat-label">{{ beat }}</span>
            </div>
            <div 
              v-for="tick in totalBeats * 8" 
              :key="'tick-' + tick"
              class="tick-line"
              :style="{ left: ((tick - 1) / (totalBeats * 8) * 100) + '%' }"
            ></div>
          </div>
          
          <div 
            v-if="isPlaying" 
            class="playhead"
            :style="{ left: (currentTick / (totalBeats * ticksPerBeat) * 100) + '%' }"
          ></div>
          
          <div class="note-rows">
            <div 
              v-for="note in displayNotes" 
              :key="note"
              class="note-row"
              :class="{ 'is-black': note.includes('#'), 'is-active': activeNotes[note] }"
              :note="note"
              @click="onRowClick"
            >
                <PianoRollNote
                  v-for="n in notesPerRow[note]"
                  :key="n.id"
                  :start="n.start"
                  :duration="n.duration"
                  :playing="n.playing"
                  :total-beats="totalBeats"
                  @removeNote="removeNote(n)"
                />
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onUnmounted, watch } from 'vue';
import Slider from './Slider.vue';
import PianoRollNote from './PianoRollNote.vue';

const notes = defineModel({ 
  type: Array, 
  default: () => [] 
});

const props = defineProps({
  activeNotes: {
    type: Object,
    default: () => ({})
  }
});

const emit = defineEmits(['playNote', 'stopNote', 'noteTriggered']);

const tempo = ref(120);
const isPlaying = ref(false);
const isPlayingBack = ref(false);
const currentTick = ref(0);
const gridRef = ref(null);

const displayNotes = [
  'B4', 'A#4', 'A4', 'G#4', 'G4', 'F#4', 'F4', 'E4', 'D#4', 'D4', 'C#4', 'C4',
  'B3', 'A#3', 'A3', 'G#3', 'G3', 'F#3', 'F3', 'E3', 'D#3', 'D3', 'C#3', 'C3'
];

const totalBars = 1;
const beatsPerBar = 4;
const totalBeats = totalBars * beatsPerBar;
const beatWidth = 60;
const ticksPerBeat = 8;

const gridWidth = computed(() => {
  if (gridRef.value) {
    return gridRef.value.clientWidth;
  }
  return totalBeats * beatWidth;
});

const noteWidth = computed(() => {
  return gridWidth.value / totalBeats;
});

let playTimeout = null;
let noteId = 0;

const notesPerRow = computed(() => {
  const rows = {};
  for (let noteName of displayNotes) {
    rows[noteName] = notes.value.filter(n => n.note === noteName);
  }
  return rows;
});

function addNote(note, beat) {
  const existing = notes.value.find(n => n.note === note && beat >= n.start && beat < n.start + n.duration);
  
  if (!existing) {
      notes.value.push({
        id: noteId++,
        note,
        start: beat,
        duration: 2/8,
        playing: false
      });
  }
}

function removeNote(note) {
  const index = notes.value.findIndex(n => n.id === note.id);
  if (index > -1) {
    notes.value.splice(index, 1);
  }
}

function onRowClick(event) {
  const rect = event.target.getBoundingClientRect();
  
  const x = event.clientX - rect.left;
  const beat = Math.floor((x / rect.width) * totalBeats * 8) / 8;
  
  addNote(event.target.getAttribute('note'), beat);
}

function clearNotes() {
  notes.value.forEach(n => {
    emit('stopNote', n.note);
  });
  notes.value = [];
}

function togglePlay() {
  if (isPlaying.value) {
    stopPlayback();
  } else {
    startPlayback();
  }
}

function playNextTick() {
  if (!isPlaying.value) return;
  
  const currentTickVal = currentTick.value;
  const totalTicks = totalBeats * ticksPerBeat;
  
  notes.value.forEach(n => {
    const noteStartTick = n.start * ticksPerBeat;
    const noteEndTick = (n.start + n.duration) * ticksPerBeat;
    
    const isNoteStart = currentTickVal === Math.floor(noteStartTick);
    const isNoteEnd = currentTickVal === Math.floor(noteEndTick);

    if(currentTickVal == 0) {
      emit('stopNote', n.note);
      n.playing = false;
    }
    
    if (isNoteStart) {
      emit('playNote', n.note);
      n.playing = true;
    } else if (isNoteEnd || (n.playing && currentTickVal > noteEndTick)) {
      emit('stopNote', n.note);
      n.playing = false;
    }
  })
  
  const tickDuration = 60000 / (tempo.value * ticksPerBeat);
  
  playTimeout = setTimeout(() => {
    if (currentTick.value + 1 >= totalTicks) {
      currentTick.value = 0;
    } else {
      currentTick.value++;
    }
    playNextTick();
  }, tickDuration);
}

function startPlayback() {
  isPlaying.value = true;
  isPlayingBack.value = true;
  currentTick.value = 0;
  playNextTick();
}

function stopPlayback() {
  isPlaying.value = false;
  isPlayingBack.value = false;
  if (playTimeout) {
    clearTimeout(playTimeout);
    playTimeout = null;
  }
  notes.value.forEach(n => {
    if (n.playing) {
      emit('stopNote', n.note);
    }
    n.playing = false;
  })
}

onUnmounted(() => {
  stopPlayback();
});

</script>

<style scoped>
.piano-roll {
  background: linear-gradient(145deg, #2a2a4a, #1a1a2e);
  border-radius: 10px;
  padding: 15px;
  margin-top: 20px;
}

.piano-roll-header {
  margin-bottom: 10px;
}

.piano-roll-controls {
  display: flex;
  align-items: center;
  gap: 15px;
}

.roll-btn {
  padding: 8px 16px;
  background: #00d9ff;
  color: #1a1a2e;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-weight: bold;
  text-transform: uppercase;
  font-size: 12px;
}

.roll-btn:hover {
  background: #00b8d9;
}

.roll-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.7; }
}

.tempo-control {
  display: flex;
  align-items: center;
  gap: 10px;
  color: #888;
  font-size: 12px;
}

.tempo-control label {
  font-size: 12px;
  text-transform: uppercase;
  letter-spacing: 1px;
  color: #888;
}

.tempo-control .tempo-value {
  color: #00d9ff;
  font-size: 14px;
  font-weight: bold;
}

.piano-roll-container {
  display: flex;
  background: #0f0f1e;
  border-radius: 5px;
  overflow: hidden;
  height: 450px;
}

.piano-keys {
  width: 50px;
  flex-shrink: 0;
  background: #1a1a2e;
  border-right: 1px solid #333;
  display: flex;
  flex-direction: column;
}

.piano-key-row {
  flex: 1;
  display: flex;
  align-items: center;
  padding-right: 5px;
  justify-content: flex-end;
  border-bottom: 1px solid #222;
  min-height: 0;
}

.piano-key-row.is-black {
  background: #151525;
}

.note-label {
  font-size: 12px;
  color: #FFF;
}

.roll-grid {
  flex: 1;
  overflow-x: auto;
  overflow-y: hidden;
}

.grid-content {
  position: relative;
  height: 100%;
  overflow: hidden;
}

.beat-lines {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 100%;
  pointer-events: none;
  z-index: 2;
}

.beat-line {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 1px;
  background: #555;
  z-index: 3;
}

.beat-line.major {
  background: #333;
}

.tick-line {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 1px;
  background: #1a1a2e;
}

.beat-label {
  position: absolute;
  top: 1px;
  left: 8px;
  font-size: 12px;
  color: #FFF;
}

.playhead {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 2px;
  background: #ff006e;
  z-index: 10;
  pointer-events: none;
  box-shadow: 0 0 8px #ff006e;
}

.note-rows {
  position: relative;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.note-row {
  flex: 1;
  border-bottom: 1px solid #1a1a2e;
  position: relative;
  min-height: 0;
}

.note-row.is-black {
  background: #151525;
}

.note-row.is-active {
  background: #0f276c;
}
</style>
