<template>
  <div class="synth">
    <div class="synth-header">
      <h1 class="synth-title">Vue Synth</h1>

      <div class="pianoroll-toggle">
        <button 
          class="toggle-btn"
          :class="{ active: showPianoRoll }"
          @click="showPianoRoll = !showPianoRoll"
        >
          {{ showPianoRoll ? 'Hide' : 'Show' }} Piano Roll
        </button>
      </div>
    </div>

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
        <label>Volume</label>
        <Slider v-model="params.volume" :min="0" :max="1" :step="0.01" format="percent" showValue/>
      </div>

      <div class="control-group">
        <label>Octave</label>
        <Slider v-model="params.octave" :min="2" :max="6" :step="1" showValue />
      </div>

      <div class="control-group">
        <label>Attack</label>
        <Knob v-model="params.attack" :min="0.01" :max="1" :step="0.01" format="s" showValue />
      </div>

      <div class="control-group">
        <label>Release</label>
        <Knob v-model="params.release" :min="0.01" :max="2" :step="0.01" format="s" showValue />
      </div>

      <div class="control-group">
        <label>Detune</label>
        <Knob v-model="params.detune" :min="-50" :max="50" :step="1" format="" showValue />
      </div>

    </div>

    <Keyboard 
      :active-notes="activeNotes"
      @playNote="playNote"
      @stopNote="stopNote"
    />

    <PianoRoll 
      v-show="showPianoRoll"
      v-model="sequencerNotes"
      :active-notes="activeNotes"
      @playNote="playNote"
      @stopNote="stopNote"
    />
  </div>
</template>

<script setup>
import { reactive, ref, onMounted, onUnmounted } from 'vue';
import Knob from './Knob.vue';
import Slider from './Slider.vue';
import Keyboard from './Keyboard.vue';
import PianoRoll from './PianoRoll.vue';

const showPianoRoll = ref(false);

const waveforms = ['sine', 'square', 'sawtooth', 'triangle'];
const waveLabels = {
  sine: 'Sine',
  square: 'Square',
  sawtooth: 'Saw',
  triangle: 'Tri'
};

const params = reactive({
  waveform: 'square',
  attack: 0.01,
  release: 0.5,
  detune: 0,
  volume: 0.5,
  octave: 4
});

const noteFrequencies = {
  'C': 261.63, 'C#': 277.18, 'D': 293.66, 'D#': 311.13,
  'E': 329.63, 'F': 349.23, 'F#': 369.99, 'G': 392.00,
  'G#': 415.30, 'A': 440.00, 'A#': 466.16, 'B': 493.88
};

const activeNotes = reactive({});
const sequencerNotes = ref([]);
let audioCtx = null;
let masterGain = null;
const activeOscillators = new Map();

const keyMap = {
  'q': 'C3', '2': 'C#3', 'w': 'D3', '3': 'D#3', 'e': 'E3',
  'r': 'F3', '5': 'F#3', 't': 'G3', '6': 'G#3', 'y': 'A3',
  '7': 'A#3', 'u': 'B3',
  'z': 'C4', 's': 'C#4', 'x': 'D4', 'd': 'D#4', 'c': 'E4',
  'v': 'F4', 'g': 'F#4', 'b': 'G4', 'h': 'G#4', 'n': 'A4',
  'j': 'A#4', 'm': 'B4'
};

function getFrequency(noteWithOctave) {
  const note = noteWithOctave.slice(0, -1);
  const octaveChar = noteWithOctave.slice(-1);
  const octave = parseInt(octaveChar) + (params.octave - 4);
  const baseFreq = noteFrequencies[note];
  return baseFreq * Math.pow(2, octave - 4);
}

function initAudio() {
  if (!audioCtx) {
    audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    masterGain = audioCtx.createGain();
    masterGain.gain.value = params.volume;
    masterGain.connect(audioCtx.destination);
  }
}

function playNote(note) {
  initAudio();
  
  if (audioCtx.state === 'suspended') {
    audioCtx.resume();
  }

  if (activeOscillators.has(note)) {
    stopNote(note);
  }

  const osc = audioCtx.createOscillator();
  const gainNode = audioCtx.createGain();
  
  osc.type = params.waveform;
  osc.frequency.value = getFrequency(note);
  osc.detune.value = params.detune;
  
  gainNode.gain.setValueAtTime(0, audioCtx.currentTime);
  gainNode.gain.linearRampToValueAtTime(params.volume, audioCtx.currentTime + params.attack);
  
  osc.connect(gainNode);
  gainNode.connect(masterGain);
  
  osc.start();
  
  activeOscillators.set(note, { osc, gainNode });
  activeNotes[note] = true;
}

function stopNote(note) {
  const nodes = activeOscillators.get(note);
  if (nodes) {
    const { osc, gainNode } = nodes;
    gainNode.gain.cancelScheduledValues(audioCtx.currentTime);
    gainNode.gain.setValueAtTime(gainNode.gain.value, audioCtx.currentTime);
    gainNode.gain.linearRampToValueAtTime(0, audioCtx.currentTime + params.release);
    
    setTimeout(() => {
      osc.stop();
      osc.disconnect();
      gainNode.disconnect();
    }, params.release * 1000 + 100);
    
    activeOscillators.delete(note);
    delete activeNotes[note];
  }
}

function handleKeyDown(e) {
  if (e.repeat) return;
  const note = keyMap[e.key.toLowerCase()];
  if (note && !activeOscillators.has(note)) {
    playNote(note);
  }
}

function handleKeyUp(e) {
  const note = keyMap[e.key.toLowerCase()];
  if (note) {
    stopNote(note);
  }
}

onMounted(() => {
  document.addEventListener('keydown', handleKeyDown);
  document.addEventListener('keyup', handleKeyUp);
})

onUnmounted(() => {
  document.removeEventListener('keydown', handleKeyDown);
  document.removeEventListener('keyup', handleKeyUp);
})
</script>

<style scoped>
.synth {
  background: linear-gradient(180deg, #0b0b15, #2e2e44);
  border-radius: 20px;
  padding: 30px;
  box-shadow:  0 20px 60px rgba(0, 0, 0, 0.5);
  max-width: 900px;
  width: 100%;
}

.synth-header {
  display: flex;
  flex-direction: row;
}

.synth-header .synth-title {
  flex: 1 1 50%;
}

.synth-header .pianoroll-toggle {
  flex: 1 1 50%;
  justify-content: right;
}

.pianoroll-toggle {
  display: flex;
  margin-bottom: 20px;
}

.toggle-btn {
  padding: 10px 20px;
  border: 1px solid #444;
  background: #2a2a4a;
  color: #888;
  cursor: pointer;
  font-size: 12px;
  text-transform: uppercase;
  transition: all 0.2s;
  border-radius: 5px;
}

.toggle-btn.active {
  background: #00d9ff;
  color: #1a1a2e;
  border-color: #00d9ff;
}

.controls {
  display: flex;
  flex-wrap: wrap;
  margin-bottom: 30px;
}

.control-group {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
  flex: 1 1 calc(33% - 20px);
  margin: 10px 0;
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
  font-size: 12px;
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
