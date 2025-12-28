<template>
  <div class="timer-section">
    <h2 class="timer-title">Timer</h2>
    
    <div class="timer-display">
      <div class="time-display" :class="{ 'ticking': isRunning, 'paused': isPaused }">
        {{ formattedTime }}
      </div>
    </div>

    <div class="timer-controls">
      <div class="preset-buttons">
        <button
          v-for="preset in presets"
          :key="preset"
          @click="setTime(preset)"
          class="preset-button"
          :class="{ active: selectedPreset === preset }"
        >
          {{ preset === 0.5 ? '30s' : preset + 'm' }}
        </button>
      </div>

      <div class="custom-time">
        <label class="time-label">Custom Time (min):</label>
        <input
          v-model.number="customMinutes"
          type="number"
          min="1"
          max="999"
          class="time-input"
          @keyup.enter="setCustomTime"
        />
        <button @click="setCustomTime" class="set-button">Set</button>
      </div>

      <div class="action-buttons">
        <button
          v-if="!isRunning && !isPaused && timeRemaining > 0"
          @click="startTimer"
          class="action-button start-button"
        >
          Start
        </button>
        <button
          v-if="isRunning"
          @click="pauseTimer"
          class="action-button pause-button"
        >
          Pause
        </button>
        <button
          v-if="isPaused"
          @click="resumeTimer"
          class="action-button resume-button"
        >
          Resume
        </button>
        <button
          @click="resetTimer"
          class="action-button reset-button"
          :class="{ 'stop-button': isBeeping }"
        >
          {{ isBeeping ? 'Stop' : 'Reset' }}
        </button>
        <button
          v-if="isRunning || isPaused"
          @click="isFullscreen = true"
          class="action-button expand-button"
        >
          ⛶ Expand
        </button>
      </div>
    </div>

    <div v-if="timeRemaining === 0 && hasStarted" class="timer-alert">
      ⏰ Time's Up!
      <button @click="stopBeeping" class="stop-beep-button">Stop Sound</button>
    </div>
  </div>

  <!-- Fullscreen Timer Overlay -->
  <div v-if="isFullscreen" class="fullscreen-overlay" @click.self="isFullscreen = false">
    <div class="fullscreen-timer">
      <button @click="isFullscreen = false" class="close-button">×</button>
      <div class="fullscreen-time-display" :class="{ 'ticking': isRunning, 'paused': isPaused }">
        {{ formattedTime }}
      </div>
      <div class="fullscreen-controls">
        <button
          v-if="isRunning"
          @click="pauseTimer"
          class="action-button pause-button"
        >
          Pause
        </button>
        <button
          v-if="isPaused"
          @click="resumeTimer"
          class="action-button resume-button"
        >
          Resume
        </button>
        <button
          @click="resetTimer"
          class="action-button reset-button"
          :class="{ 'stop-button': isBeeping }"
        >
          {{ isBeeping ? 'Stop' : 'Reset' }}
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onUnmounted } from 'vue'

const presets = [0.5, 1, 5, 10, 30]
const selectedPreset = ref(null)
const customMinutes = ref(5)
const timeRemaining = ref(0) // in seconds
const isRunning = ref(false)
const isPaused = ref(false)
const hasStarted = ref(false)
const isFullscreen = ref(false)
const isBeeping = ref(false)
let intervalId = null
let beepInterval = null

const formattedTime = computed(() => {
  const minutes = Math.floor(timeRemaining.value / 60)
  const seconds = timeRemaining.value % 60
  return `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`
})

const setTime = (minutes) => {
  selectedPreset.value = minutes
  timeRemaining.value = minutes * 60
  hasStarted.value = false
  isRunning.value = false
  isPaused.value = false
  stopTimer()
}

const setCustomTime = () => {
  if (customMinutes.value && customMinutes.value > 0) {
    selectedPreset.value = null
    timeRemaining.value = customMinutes.value * 60
    hasStarted.value = false
    isRunning.value = false
    isPaused.value = false
    stopTimer()
  }
}

const startTimer = () => {
  if (timeRemaining.value > 0) {
    isRunning.value = true
    isPaused.value = false
    hasStarted.value = true
    startInterval()
  }
}

const pauseTimer = () => {
  isRunning.value = false
  isPaused.value = true
  stopTimer()
}

const resumeTimer = () => {
  isRunning.value = true
  isPaused.value = false
  startInterval()
}

const resetTimer = () => {
  stopTimer()
  stopBeeping()
  timeRemaining.value = 0
  isRunning.value = false
  isPaused.value = false
  hasStarted.value = false
  selectedPreset.value = null
}

const playBeep = () => {
  // Create a simple beep sound using Web Audio API
  const audioContext = new (window.AudioContext || window.webkitAudioContext)()
  const oscillator = audioContext.createOscillator()
  const gainNode = audioContext.createGain()
  
  oscillator.connect(gainNode)
  gainNode.connect(audioContext.destination)
  
  oscillator.frequency.value = 800 // Beep frequency
  oscillator.type = 'sine'
  
  gainNode.gain.setValueAtTime(0.3, audioContext.currentTime)
  gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.2)
  
  oscillator.start(audioContext.currentTime)
  oscillator.stop(audioContext.currentTime + 0.1)
}

const startBeeping = () => {
  // Stop any existing beeping first
  stopBeeping()
  // Play beep immediately, then repeat every 0.5 seconds
  isBeeping.value = true
  playBeep()
  beepInterval = setInterval(() => {
    playBeep()
  }, 500)
}

const stopBeeping = () => {
  if (beepInterval) {
    clearInterval(beepInterval)
    beepInterval = null
  }
  isBeeping.value = false
  // Reset the timer state so the alert disappears
  hasStarted.value = false
}

const startInterval = () => {
  stopTimer()
  stopBeeping()
  intervalId = setInterval(() => {
    if (timeRemaining.value > 0) {
      timeRemaining.value--
    } else {
      stopTimer()
      isRunning.value = false
      isPaused.value = false
      // Start beeping when timer completes
      startBeeping()
      // Show browser notification if permission granted
      if ('Notification' in window && Notification.permission === 'granted') {
        new Notification('Timer Complete!', {
          body: 'Your timer has finished.',
          icon: '⏰'
        })
      } else if ('Notification' in window && Notification.permission !== 'denied') {
        Notification.requestPermission().then(permission => {
          if (permission === 'granted') {
            new Notification('Timer Complete!', {
              body: 'Your timer has finished.',
              icon: '⏰'
            })
          }
        })
      }
    }
  }, 1000)
}

const stopTimer = () => {
  if (intervalId) {
    clearInterval(intervalId)
    intervalId = null
  }
}

onUnmounted(() => {
  stopTimer()
  stopBeeping()
})
</script>

<style scoped>
.timer-section {
  background: white;
  border: 3px solid #000;
  box-shadow: 6px 6px 0px 0px #000;
  padding: 1rem;
  margin-top: 0;
  margin-bottom: 0;
}

@media (min-width: 768px) and (max-width: 1024px) {
  .timer-section {
    padding: 0.875rem;
  }
  
  .timer-title {
    font-size: 1.125rem;
    margin-bottom: 0.625rem;
  }
  
  .time-display {
    font-size: 2.25rem;
    padding: 0.625rem 1.25rem;
    min-width: 110px;
  }
  
  .timer-display {
    margin-bottom: 0.875rem;
  }
}

.timer-title {
  font-size: 1.25rem;
  font-weight: 900;
  text-transform: uppercase;
  margin-bottom: 0.75rem;
  color: #000;
  text-align: center;
}

.timer-display {
  display: flex;
  justify-content: center;
  margin-bottom: 1rem;
}

.time-display {
  font-size: 2.5rem;
  font-weight: 900;
  font-family: 'Courier New', monospace;
  color: #000;
  background: #f0f0f0;
  border: 3px solid #000;
  padding: 0.75rem 1.5rem;
  box-shadow: 4px 4px 0px 0px #000;
  min-width: 120px;
  text-align: center;
}

.time-display.ticking {
  animation: tickAnimation 1s ease-in-out infinite;
}

.time-display.paused {
  animation-play-state: paused;
  transition: none;
}

@keyframes tickAnimation {
  0%, 100% {
    transform: scale(1);
    box-shadow: 4px 4px 0px 0px #000;
  }
  50% {
    transform: scale(1.05);
    box-shadow: 6px 6px 0px 0px #000;
  }
}

@keyframes stopButtonAnimation {
  0%, 100% {
    transform: scale(1.2);
    box-shadow: 6px 6px 0px 0px #000;
  }
  50% {
    transform: scale(1.27);
    box-shadow: 8px 8px 0px 0px #000;
  }
}

.timer-controls {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

@media (min-width: 768px) and (max-width: 1024px) {
  .timer-controls {
    gap: 0.625rem;
  }
}

.preset-buttons {
  display: flex;
  gap: 1rem;
  justify-content: center;
  flex-wrap: wrap;
}

@media (min-width: 768px) and (max-width: 1024px) {
  .preset-buttons {
    gap: 0.75rem;
  }
  
  .preset-button {
    padding: 0.4375rem 0.875rem;
    font-size: 0.8125rem;
  }
}

.preset-button {
  padding: 0.5rem 1rem;
  font-size: 0.875rem;
  font-weight: 700;
  text-transform: uppercase;
  border: 3px solid #000;
  background: white;
  color: #000;
  cursor: pointer;
  box-shadow: 4px 4px 0px 0px #000;
  transition: all 0.1s ease;
  font-family: system-ui, -apple-system, sans-serif;
}

.preset-button:hover {
  transform: translate(3px, 3px);
  box-shadow: 3px 3px 0px 0px #000;
}

.preset-button.active {
  background: #000;
  color: white;
}

.custom-time {
  display: flex;
  align-items: center;
  gap: 1rem;
  justify-content: center;
  flex-wrap: wrap;
}

@media (min-width: 768px) and (max-width: 1024px) {
  .custom-time {
    gap: 0.75rem;
    flex-wrap: nowrap;
  }
  
  .time-label {
    font-size: 0.8125rem;
    white-space: nowrap;
  }
}

.time-label {
  font-weight: 700;
  text-transform: uppercase;
  font-size: 0.875rem;
}

.time-input {
  padding: 0.5rem;
  border: 3px solid #000;
  font-size: 0.875rem;
  font-weight: 600;
  width: 80px;
  box-shadow: 3px 3px 0px 0px #000;
  transition: all 0.1s ease;
}

.time-input:focus {
  outline: none;
  transform: translate(2px, 2px);
  box-shadow: 2px 2px 0px 0px #000;
}

.set-button {
  padding: 0.5rem 1rem;
  font-size: 0.875rem;
  font-weight: 700;
  text-transform: uppercase;
  border: 3px solid #000;
  background: #9C27B0;
  color: white;
  cursor: pointer;
  box-shadow: 4px 4px 0px 0px #000;
  transition: all 0.1s ease;
  font-family: system-ui, -apple-system, sans-serif;
}

.set-button:hover {
  transform: translate(3px, 3px);
  box-shadow: 3px 3px 0px 0px #000;
}

.action-buttons {
  display: flex;
  gap: 1rem;
  justify-content: center;
  flex-wrap: wrap;
}

@media (min-width: 768px) and (max-width: 1024px) {
  .action-buttons {
    gap: 0.75rem;
  }
  
  .action-button {
    padding: 0.4375rem 0.875rem;
    font-size: 0.8125rem;
    flex: 1;
    min-width: 100px;
  }
}

.action-button {
  padding: 0.5rem 1rem;
  font-size: 0.875rem;
  font-weight: 700;
  text-transform: uppercase;
  border: 3px solid #000;
  cursor: pointer;
  box-shadow: 4px 4px 0px 0px #000;
  transition: all 0.1s ease;
  font-family: system-ui, -apple-system, sans-serif;
}

.start-button {
  background: #9C27B0;
  color: white;
}

.pause-button {
  background: white;
  color: #000;
}

.resume-button {
  background: white;
  color: #000;
}

.reset-button {
  background: #f0f0f0;
  color: #000;
}

.reset-button.stop-button {
  background: #ff0000;
  color: white;
  padding: 1.5rem 3rem;
  animation: stopButtonAnimation 1s ease-in-out infinite;
  font-size: 2.25rem;
  font-weight: 900;
}

.action-button:hover {
  transform: translate(2px, 2px);
  box-shadow: 2px 2px 0px 0px #000;
}

.timer-alert {
  margin-top: 0.75rem;
  padding: 0.75rem;
  background: #ff0000;
  color: white;
  border: 3px solid #000;
  font-size: 1rem;
  font-weight: 900;
  text-transform: uppercase;
  text-align: center;
  box-shadow: 4px 4px 0px 0px #000;
  animation: pulse 1s infinite;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  flex-wrap: wrap;
}

.stop-beep-button {
  padding: 0.5rem 1rem;
  font-size: 0.875rem;
  font-weight: 700;
  text-transform: uppercase;
  border: 3px solid #fff;
  padding-top: 0.5rem;
  background: white;
  color: #ff0000;
  cursor: pointer;
  box-shadow: 4px 4px 0px 0px #fff;
  transition: all 0.1s ease;
  font-family: system-ui, -apple-system, sans-serif;
}

.stop-beep-button:hover {
  transform: translate(2px, 2px);
  box-shadow: 2px 2px 0px 0px #fff;
}

.expand-button {
  background: white;
  color: #000;
}

.fullscreen-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 9999;
}

.fullscreen-timer {
  background: white;
  border: 6px solid #000;
  box-shadow: 18px 18px 0px 0px #000;
  padding: 4.5rem;
  position: relative;
  min-width: 600px;
  text-align: center;
}

.close-button {
  position: absolute;
  top: 1.5rem;
  right: 1.5rem;
  width: 72px;
  height: 72px;
  background: #ff0000;
  color: white;
  border: 4.5px solid #000;
  font-size: 3rem;
  font-weight: bold;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 6px 6px 0px 0px #000;
  transition: all 0.1s ease;
}

.close-button:hover {
  transform: translate(2px, 2px);
  box-shadow: 2px 2px 0px 0px #000;
}

.fullscreen-time-display {
  font-size: 12rem;
  font-weight: 900;
  font-family: 'Courier New', monospace;
  color: #000;
  margin: 3rem 0;
  text-align: center;
}

.fullscreen-time-display.ticking {
  animation: tickAnimationFullscreen 1s ease-in-out infinite;
}

.fullscreen-time-display.paused {
  animation-play-state: paused;
  transition: none;
}

@keyframes tickAnimationFullscreen {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.03);
  }
}

.fullscreen-controls {
  display: flex;
  gap: 1.5rem;
  justify-content: center;
  flex-wrap: wrap;
}

.fullscreen-controls .action-button {
  padding: 0.75rem 1.5rem;
  font-size: 1.125rem;
}


.stop-beep-button {
  padding: 0.5rem 1rem;
  font-size: 0.875rem;
  font-weight: 700;
  text-transform: uppercase;
  border: 3px solid #fff;
  background: white;
  color: #ff0000;
  cursor: pointer;
  box-shadow: 4px 4px 0px 0px #fff;
  transition: all 0.1s ease;
  font-family: system-ui, -apple-system, sans-serif;
}

.stop-beep-button:hover {
  transform: translate(2px, 2px);
  box-shadow: 2px 2px 0px 0px #fff;
}

@keyframes pulse {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.05);
  }
}

@media (min-width: 768px) and (max-width: 1024px) {
  .timer-alert {
    padding: 0.625rem;
    font-size: 0.9375rem;
    gap: 0.75rem;
  }
}

@media (max-width: 640px) {
  .time-display {
    font-size: 2.5rem;
    padding: 1rem 2rem;
  }
  
  .timer-title {
    font-size: 1.5rem;
  }
}
</style>

