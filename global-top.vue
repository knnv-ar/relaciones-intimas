<template>
  <div class="grid-indicator">
    <div v-for="r in totalRows" :key="r" class="indicator-row">
      <div
        v-for="c in COLS"
        :key="c"
        class="indicator-dot"
        :class="{ active: isActive(r - 1, c - 1), exists: slideExists(r - 1, c - 1) }"
        @click="goTo(r - 1, c - 1)"
        :title="`Slide ${(r - 1) * COLS + c}`"
      />
    </div>
  </div>
</template>

<script setup>
import { computed, onMounted, onUnmounted } from 'vue'
import { useNav } from '@slidev/client'

// ─── Configuración del grid ───────────────────────────────
const COLS = 3   // ← Cambiá este número si modificás el layout del grid

const { currentPage, go, total } = useNav()

const totalRows = computed(() => Math.ceil(total.value / COLS))

// ─── Utilidades de posición ───────────────────────────────
const getPos = (page) => ({
  row: Math.floor((page - 1) / COLS),
  col: (page - 1) % COLS,
})

const getPage = (row, col) => row * COLS + col + 1

const isActive = (row, col) => {
  const pos = getPos(currentPage.value)
  return pos.row === row && pos.col === col
}

const slideExists = (row, col) => {
  const page = getPage(row, col)
  return page >= 1 && page <= total.value
}

// ─── Navegación 2D ───────────────────────────────────────
const navigate = (dir) => {
  const { row, col } = getPos(currentPage.value)

  const next = {
    right: { row, col: col + 1 },
    left:  { row, col: col - 1 },
    down:  { row: row + 1, col },
    up:    { row: row - 1, col },
  }[dir]

  if (
    next.row < 0 ||
    next.col < 0 ||
    next.col >= COLS ||
    next.row >= totalRows.value
  ) return

  const target = getPage(next.row, next.col)
  if (target >= 1 && target <= total.value) go(target)
}

const goTo = (row, col) => {
  if (!slideExists(row, col)) return
  go(getPage(row, col))
}

// ─── Teclado ─────────────────────────────────────────────
const onKey = (e) => {
  const map = {
    ArrowRight: 'right',
    ArrowLeft:  'left',
    ArrowDown:  'down',
    ArrowUp:    'up',
  }
  if (!map[e.key]) return
  e.preventDefault()
  e.stopImmediatePropagation()   // evita que Slidev también maneje la tecla
  navigate(map[e.key])
}

// ─── Swipe táctil ────────────────────────────────────────
let touchStart = null
const SWIPE_THRESHOLD = 50       // píxeles mínimos para considerar swipe

const onTouchStart = (e) => {
  touchStart = {
    x: e.touches[0].clientX,
    y: e.touches[0].clientY,
  }
}

const onTouchEnd = (e) => {
  if (!touchStart) return
  const dx = e.changedTouches[0].clientX - touchStart.x
  const dy = e.changedTouches[0].clientY - touchStart.y
  touchStart = null

  if (Math.abs(dx) > Math.abs(dy)) {
    if (Math.abs(dx) < SWIPE_THRESHOLD) return
    navigate(dx > 0 ? 'left' : 'right')   // swipe derecha = ir a slide izquierda
  } else {
    if (Math.abs(dy) < SWIPE_THRESHOLD) return
    navigate(dy > 0 ? 'up' : 'down')      // swipe abajo = ir a slide arriba
  }
}

// ─── Lifecycle ───────────────────────────────────────────
onMounted(() => {
  window.addEventListener('keydown', onKey, true)                        // capture phase
  window.addEventListener('touchstart', onTouchStart, { passive: true })
  window.addEventListener('touchend',   onTouchEnd,   { passive: true })
})

onUnmounted(() => {
  window.removeEventListener('keydown', onKey, true)
  window.removeEventListener('touchstart', onTouchStart)
  window.removeEventListener('touchend',   onTouchEnd)
})
</script>

<style scoped>
.grid-indicator {
  position: fixed;
  bottom: 20px;
  right: 20px;
  z-index: 1000;
  display: flex;
  flex-direction: column;
  gap: 6px;
  background: rgba(0, 0, 0, 0.45);
  backdrop-filter: blur(8px);
  padding: 10px;
  border-radius: 10px;
  border: 1px solid rgba(255, 255, 255, 0.15);
}

.indicator-row {
  display: flex;
  gap: 6px;
}

.indicator-dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.25);
  border: 1px solid rgba(255, 255, 255, 0.4);
  cursor: pointer;
  transition: all 0.2s ease;
}

.indicator-dot.exists:hover {
  background: rgba(255, 255, 255, 0.5);
  transform: scale(1.2);
}

.indicator-dot.active {
  background: white;
  box-shadow: 0 0 8px rgba(255, 255, 255, 0.8);
  transform: scale(1.3);
}
</style>