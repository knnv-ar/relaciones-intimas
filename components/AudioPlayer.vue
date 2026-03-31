<template>
  <div class="audio-player">
    <button class="btn" @click="rewind" title="Rebobinar">⏮</button>
    <button class="btn play-btn" @click="togglePlay" title="Play / Pause">
      <span v-if="!isPlaying">▶</span>
      <span v-else>⏸</span>
    </button>
    <div class="progress-bar" @click="seek">
      <div class="progress-fill" :style="{ width: progress + '%' }"></div>
    </div>
    <span class="time">{{ currentTime }} / {{ duration }}</span>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { Howl } from 'howler'

const props = defineProps({
  src: { type: String, required: true }
})

const isPlaying  = ref(false)
const progress   = ref(0)
const currentTime = ref('0:00')
const duration   = ref('0:00')
let sound = null
let rafId = null

const formatTime = (secs) => {
  const m = Math.floor(secs / 60)
  const s = Math.floor(secs % 60).toString().padStart(2, '0')
  return `${m}:${s}`
}

const updateProgress = () => {
  if (sound && isPlaying.value) {
    const seek = sound.seek() || 0
    const dur  = sound.duration() || 1
    progress.value    = (seek / dur) * 100
    currentTime.value = formatTime(seek)
    rafId = requestAnimationFrame(updateProgress)
  }
}

// ← NEW: parar este player cuando otro slide pida silencio
const stopHandler = () => {
  if (isPlaying.value) {
    sound?.pause()
    isPlaying.value = false
    cancelAnimationFrame(rafId)
  }
}

onMounted(() => {
  sound = new Howl({
    src: [props.src],
    html5: true,
    onload: () => { duration.value = formatTime(sound.duration()) },
    onend: () => {
      isPlaying.value   = false
      progress.value    = 0
      currentTime.value = '0:00'
    }
  })

  // ← NEW: escuchar el evento global de parada
  window.addEventListener('grid:stop-audio', stopHandler)
})

onUnmounted(() => {
  cancelAnimationFrame(rafId)
  sound?.unload()
  // ← NEW: limpiar el listener para evitar memory leaks
  window.removeEventListener('grid:stop-audio', stopHandler)
})

const togglePlay = () => {
  if (isPlaying.value) {
    sound.pause()
    isPlaying.value = false
    cancelAnimationFrame(rafId)
  } else {
    sound.play()
    isPlaying.value = true
    updateProgress()
  }
}

const rewind = () => {
  sound.seek(0)
  progress.value    = 0
  currentTime.value = '0:00'
  if (!isPlaying.value) {
    sound.play()
    isPlaying.value = true
    updateProgress()
  }
}

const seek = (e) => {
  const rect  = e.currentTarget.getBoundingClientRect()
  const ratio = (e.clientX - rect.left) / rect.width
  sound.seek(ratio * sound.duration())
}
</script>

<style scoped>
.audio-player {
  display: flex;
  align-items: center;
  gap: 10px;
  background: rgba(0, 0, 0, 0.55);
  backdrop-filter: blur(8px);
  border-radius: 999px;
  padding: 8px 16px;
  width: fit-content;
  color: white;
  font-size: 0.85rem;
  border: 1px solid rgba(255, 255, 255, 0.2);
}
.btn { background: none; border: none; color: white; cursor: pointer; font-size: 1.1rem; padding: 0 4px; transition: transform 0.1s; }
.btn:hover { transform: scale(1.2); }
.play-btn { font-size: 1.3rem; }
.progress-bar {
  width: 120px;
  height: 4px;
  background: rgba(255, 255, 255, 0.3);
  border-radius: 2px;
  cursor: pointer;
}
.progress-fill { height: 100%; background: white; border-radius: 2px; transition: width 0.1s linear; }
.time { font-size: 0.75rem; opacity: 0.8; white-space: nowrap; }
</style>