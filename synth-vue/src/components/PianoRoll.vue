<template>
  <div class="piano-roll">
    <div class="piano-roll-header">
      <div class="piano-roll-controls">
        <button class="roll-btn" @click="togglePlay">
          {{ isPlaying ? 'Stop' : 'Play' }}
        </button>
        <button class="roll-btn" @click="clearNotes">Clear</button>
        <div class="tempo-control">
          <label>Tempo:</label>
          <input type="range" v-model.number="tempo" min="60" max="200" step="1">
          <span>{{ tempo }} BPM</span>
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
      
      <div class="roll-grid" ref="gridRef" @scroll="handleScroll">
        <div class="grid-content" :style="{ width: gridWidth + 'px' }">
          <div class="beat-lines">
            <div 
              v-for="beat in totalBeats" 
              :key="beat"
              class="beat-line"
              :class="{ 'major': beat % 4 === 1 }"
              :style="{ left: (beat - 1) * beatsPerBar * beatWidth + 'px' }"
            >
              <span class="beat-label">{{ beat }}</span>
            </div>
          </div>
          
          <div class="note-rows" ref="noteRowsRef" @click="onGridClick">
            <div 
              v-for="note in displayNotes" 
              :key="note"
              class="note-row"
              :class="{ 'is-black': note.includes('#') }"
            >
              <div 
                v-for="n in getNotesForRow(note)" 
                :key="n.id"
                class="note"
                :class="{ 'is-playing': n.playing }"
                :style="{ 
                  left: n.start * noteWidth + 'px',
                  width: n.duration * noteWidth + 'px'
                }"
                @mousedown.stop="removeNote(n)"
              ></div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onUnmounted, watch } from 'vue'

const props = defineProps({
  modelValue: {
    type: Array,
    default: () => []
  }
})

const emit = defineEmits(['update:modelValue', 'playNote', 'stopNote'])

const tempo = ref(120)
const isPlaying = ref(false)
const currentBeat = ref(0)
const gridRef = ref(null)
const noteRowsRef = ref(null)

const displayNotes = [
  'C5', 'B4', 'A#4', 'A4', 'G#4', 'G4', 'F#4', 'F4', 'E4', 'D#4', 'D4', 'C#4', 'C4',
  'B3', 'A#3', 'A3', 'G#3', 'G3', 'F#3', 'F3', 'E3', 'D#3', 'D3', 'C#3', 'C3'
]

const totalBars = 4
const beatsPerBar = 4
const totalBeats = totalBars * beatsPerBar
const beatWidth = 60
const noteWidth = 30
const gridWidth = totalBeats * beatWidth

const notes = ref([...props.modelValue])

let playTimeout = null
let noteId = 0

const getNotesForRow = (noteName) => {
  return notes.value.filter(n => n.note === noteName)
}

function addNote(note, beat) {
  const existing = notes.value.find(n => n.note === note && 
    beat >= n.start && beat < n.start + n.duration)
  
  if (!existing) {
    notes.value.push({
      id: noteId++,
      note,
      start: beat,
      duration: 1,
      playing: false
    })
    emit('update:modelValue', notes.value)
  }
}

function removeNote(note) {
  const index = notes.value.findIndex(n => n.id === note.id)
  if (index > -1) {
    notes.value.splice(index, 1)
    emit('update:modelValue', notes.value)
  }
}

function clearNotes() {
  notes.value = []
  emit('update:modelValue', notes.value)
}

function togglePlay() {
  if (isPlaying.value) {
    stopPlayback()
  } else {
    startPlayback()
  }
}

function playNextBeat() {
  if (!isPlaying.value) return
  
  const currentNotes = notes.value.filter(n => n.start === currentBeat.value)
  
  notes.value.forEach(n => {
    const wasPlaying = n.playing
    n.playing = n.start === currentBeat.value
    
    if (wasPlaying && !n.playing) {
      emit('stop-note', n.note)
    }
  })
  
  currentNotes.forEach(n => {
    emit('play-note', n.note)
  })
  
  currentBeat.value++
  if (currentBeat.value >= totalBeats) {
    currentBeat.value = 0
  }
  
  const beatDuration = 60000 / tempo.value
  playTimeout = setTimeout(playNextBeat, beatDuration)
}

function startPlayback() {
  isPlaying.value = true
  currentBeat.value = 0
  playNextBeat()
}

function stopPlayback() {
  isPlaying.value = false
  if (playTimeout) {
    clearTimeout(playTimeout)
    playTimeout = null
  }
  notes.value.forEach(n => {
    if (n.playing) {
      emit('stop-note', n.note)
    }
    n.playing = false
  })
}

function handleScroll(e) {
  // Handle scroll sync if needed
}

// Handle click on grid to add notes
function onGridClick(e) {
  if (!noteRowsRef.value) return
  
  const rect = noteRowsRef.value.getBoundingClientRect()
  const scrollLeft = gridRef.value ? gridRef.value.scrollLeft : 0
  
  const x = e.clientX - rect.left + scrollLeft
  const y = e.clientY - rect.top
  
  const rowHeight = rect.height / displayNotes.length
  const noteIndex = Math.floor(y / rowHeight)
  const beat = Math.floor(x / noteWidth)
  
  if (noteIndex >= 0 && noteIndex < displayNotes.length) {
    addNote(displayNotes[noteIndex], beat)
  }
}

onUnmounted(() => {
  stopPlayback()
})

watch(() => props.modelValue, (newVal) => {
  notes.value = [...newVal]
}, { deep: true })
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
}

.roll-btn:hover {
  background: #00b8d9;
}

.tempo-control {
  display: flex;
  align-items: center;
  gap: 10px;
  color: #888;
  font-size: 12px;
}

.tempo-control input {
  width: 80px;
}

.tempo-control span {
  color: #00d9ff;
  font-weight: bold;
}

.piano-roll-container {
  display: flex;
  background: #0f0f1e;
  border-radius: 5px;
  overflow: hidden;
  height: 300px;
}

.piano-keys {
  width: 50px;
  flex-shrink: 0;
  background: #1a1a2e;
  border-right: 1px solid #333;
}

.piano-key-row {
  height: 12.5px;
  display: flex;
  align-items: center;
  padding-right: 5px;
  justify-content: flex-end;
  border-bottom: 1px solid #222;
}

.piano-key-row.is-black {
  background: #151525;
}

.note-label {
  font-size: 8px;
  color: #666;
}

.roll-grid {
  flex: 1;
  overflow-x: auto;
  overflow-y: hidden;
}

.grid-content {
  position: relative;
  height: 100%;
}

.beat-lines {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 100%;
  pointer-events: none;
}

.beat-line {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 1px;
  background: #222;
}

.beat-line.major {
  background: #333;
}

.beat-label {
  position: absolute;
  top: 2px;
  left: 4px;
  font-size: 8px;
  color: #555;
}

.note-rows {
  position: relative;
  height: 100%;
}

.note-row {
  height: 12.5px;
  border-bottom: 1px solid #1a1a2e;
  position: relative;
}

.note-row.is-black {
  background: #151525;
}

.note {
  position: absolute;
  top: 1px;
  height: 10px;
  background: #00d9ff;
  border-radius: 2px;
  cursor: pointer;
  transition: background 0.1s;
}

.note:hover {
  background: #ff006e;
}

.note.is-playing {
  background: #00ff88;
  box-shadow: 0 0 5px #00ff88;
}
</style>
