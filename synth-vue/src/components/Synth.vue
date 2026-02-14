<template>
  <div class="synth">
    <div class="controls">
      <div class="control-group">
        <label>Oscillator</label>
        <div class="waveform-selector">
          <button 
            v-for="wave in waveforms" 
            :key="wave"
            class="wave-btn"
            :class="{ active: params.waveform === wave }"
            @click="params.waveform = wave"
          >
            {{ waveLabels[wave] }}
          </button>
        </div>
      </div>

      <div class="control-group">
        <label>Attack</label>
        <Knob v-model="params.attack" :min="0.01" :max="1" :step="0.01" format="s" />
      </div>

      <div class="control-group">
        <label>Release</label>
        <Knob v-model="params.release" :min="0.01" :max="2" :step="0.01" format="s" />
      </div>

      <div class="control-group">
        <label>Detune</label>
        <Knob v-model="params.detune" :min="-50" :max="50" :step="1" format="" />
      </div>

      <div class="control-group">
        <label>Volume</label>
        <Slider v-model="params.volume" :min="0" :max="1" :step="0.01" format="percent" />
      </div>

      <div class="control-group">
        <label>Octave</label>
        <Slider v-model="params.octave" :min="2" :max="6" :step="1" />
      </div>
    </div>

    <div class="keyboard">
      <div
        v-for="key in keyboardNotes"
        :key="key.note"
        class="key"
        :class="[key.type, { active: activeNotes.has(key.note) }]"
        :style="{ left: getKeyPosition(key) + 'px' }"
        @mousedown="playNote(key.note)"
        @mouseup="stopNote(key.note)"
        @mouseleave="stopNote(key.note)"
      >
        <span class="key-label">{{ key.note.slice(0, -1) }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, onMounted, onUnmounted } from 'vue'
import Knob from './Knob.vue'
import Slider from './Slider.vue'

const waveforms = ['sine', 'square', 'sawtooth', 'triangle']
const waveLabels = {
  sine: 'Sine',
  square: 'Square',
  sawtooth: 'Saw',
  triangle: 'Tri'
}

const params = reactive({
  waveform: 'sine',
  attack: 0.1,
  release: 0.5,
  detune: 0,
  volume: 0.5,
  octave: 4
})

const noteFrequencies = {
  'C': 261.63, 'C#': 277.18, 'D': 293.66, 'D#': 311.13,
  'E': 329.63, 'F': 349.23, 'F#': 369.99, 'G': 392.00,
  'G#': 415.30, 'A': 440.00, 'A#': 466.16, 'B': 493.88
}

const keyboardNotes = [
  { note: 'C3', type: 'white' }, { note: 'C#3', type: 'black' },
  { note: 'D3', type: 'white' }, { note: 'D#3', type: 'black' },
  { note: 'E3', type: 'white' },
  { note: 'F3', type: 'white' }, { note: 'F#3', type: 'black' },
  { note: 'G3', type: 'white' }, { note: 'G#3', type: 'black' },
  { note: 'A3', type: 'white' }, { note: 'A#3', type: 'black' },
  { note: 'B3', type: 'white' },
  { note: 'C4', type: 'white' }, { note: 'C#4', type: 'black' },
  { note: 'D4', type: 'white' }, { note: 'D#4', type: 'black' },
  { note: 'E4', type: 'white' },
  { note: 'F4', type: 'white' }, { note: 'F#4', type: 'black' },
  { note: 'G4', type: 'white' }, { note: 'G#4', type: 'black' },
  { note: 'A4', type: 'white' }, { note: 'A#4', type: 'black' },
  { note: 'B4', type: 'white' }
]

const activeNotes = new Set()
let audioCtx = null
let masterGain = null
const activeOscillators = new Map()

const keyMap = {
  'a': 'C3', 'w': 'C#3', 's': 'D3', 'e': 'D#3', 'd': 'E3',
  'f': 'F3', 't': 'F#3', 'g': 'G3', 'y': 'G#3', 'h': 'A3',
  'u': 'A#3', 'j': 'B3',
  'k': 'C4', 'o': 'C#4', 'l': 'D4', 'p': 'D#4', ';': 'E4'
}

const whiteKeyWidth = 52
const blackKeyWidth = 35
const offset = 50

function getKeyPosition(key) {
  const whiteKeysBefore = keyboardNotes.slice(0, keyboardNotes.indexOf(key))
    .filter(k => k.type === 'white').length
  
  if (key.type === 'white') {
    return whiteKeysBefore * whiteKeyWidth + offset
  } else {
    return whiteKeysBefore * whiteKeyWidth - blackKeyWidth / 2 + offset
  }
}

function getFrequency(noteWithOctave) {
  const note = noteWithOctave.slice(0, -1)
  const octaveChar = noteWithOctave.slice(-1)
  const octave = parseInt(octaveChar) + (params.octave - 4)
  const baseFreq = noteFrequencies[note]
  return baseFreq * Math.pow(2, octave - 4)
}

function initAudio() {
  if (!audioCtx) {
    audioCtx = new (window.AudioContext || window.webkitAudioContext)()
    masterGain = audioCtx.createGain()
    masterGain.gain.value = params.volume
    masterGain.connect(audioCtx.destination)
  }
}

function playNote(note) {
  initAudio()
  
  if (audioCtx.state === 'suspended') {
    audioCtx.resume()
  }

  if (activeOscillators.has(note)) {
    stopNote(note)
  }

  const osc = audioCtx.createOscillator()
  const gainNode = audioCtx.createGain()
  
  osc.type = params.waveform
  osc.frequency.value = getFrequency(note)
  osc.detune.value = params.detune
  
  gainNode.gain.setValueAtTime(0, audioCtx.currentTime)
  gainNode.gain.linearRampToValueAtTime(params.volume, audioCtx.currentTime + params.attack)
  
  osc.connect(gainNode)
  gainNode.connect(masterGain)
  
  osc.start()
  
  activeOscillators.set(note, { osc, gainNode })
  activeNotes.add(note)
}

function stopNote(note) {
  const nodes = activeOscillators.get(note)
  if (nodes) {
    const { osc, gainNode } = nodes
    gainNode.gain.cancelScheduledValues(audioCtx.currentTime)
    gainNode.gain.setValueAtTime(gainNode.gain.value, audioCtx.currentTime)
    gainNode.gain.linearRampToValueAtTime(0, audioCtx.currentTime + params.release)
    
    setTimeout(() => {
      osc.stop()
      osc.disconnect()
      gainNode.disconnect()
    }, params.release * 1000 + 100)
    
    activeOscillators.delete(note)
    activeNotes.delete(note)
  }
}

function handleKeyDown(e) {
  if (e.repeat) return
  const note = keyMap[e.key.toLowerCase()]
  if (note && !activeOscillators.has(note)) {
    playNote(note)
  }
}

function handleKeyUp(e) {
  const note = keyMap[e.key.toLowerCase()]
  if (note) {
    stopNote(note)
  }
}

onMounted(() => {
  document.addEventListener('keydown', handleKeyDown)
  document.addEventListener('keyup', handleKeyUp)
})

onUnmounted(() => {
  document.removeEventListener('keydown', handleKeyDown)
  document.removeEventListener('keyup', handleKeyUp)
})
</script>

<style scoped>
.synth {
  background: linear-gradient(145deg, #2a2a4a, #1a1a2e);
  border-radius: 20px;
  padding: 30px;
  box-shadow: 
    0 20px 60px rgba(0, 0, 0, 0.5),
    inset 0 1px 0 rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.1);
  max-width: 900px;
  width: 100%;
}

.controls {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 25px;
  margin-bottom: 30px;
}

.control-group {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}

.control-group label {
  font-size: 12px;
  text-transform: uppercase;
  letter-spacing: 1px;
  color: #888;
}

.knob-value {
  font-size: 14px;
  font-weight: bold;
  color: #00d9ff;
}

.keyboard {
  display: block;
  position: relative;
  padding: 20px;
  background: linear-gradient(180deg, #1a1a2e, #0f0f1e);
  border-radius: 10px;
  box-shadow: inset 0 5px 15px rgba(0, 0, 0, 0.5);
  height: 240px;
  width: 820px;
}

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

.waveform-selector {
  display: flex;
  gap: 5px;
}

.wave-btn {
  padding: 8px 12px;
  border: 1px solid #444;
  background: #2a2a4a;
  color: #888;
  cursor: pointer;
  font-size: 11px;
  text-transform: uppercase;
  transition: all 0.2s;
}

.wave-btn:first-child {
  border-radius: 5px 0 0 5px;
}

.wave-btn:last-child {
  border-radius: 0 5px 5px 0;
}

.wave-btn.active {
  background: #00d9ff;
  color: #1a1a2e;
  border-color: #00d9ff;
}
</style>
