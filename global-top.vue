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

const COLS = 3

const { currentPage, go, total } = useNav()

const totalRows = computed(() => Math.ceil(total.value / COLS))

const getPos      = (page) => ({ row: Math.floor((page - 1) / COLS), col: (page - 1) % COLS })
const getPage     = (row, col) => row * COLS + col + 1
const isActive    = (row, col) => { const p = getPos(currentPage.value); return p.row === row && p.col === col }
const slideExists = (row, col) => { const p = getPage(row, col); return p >= 1 && p <= total.value }

// ─── Navegar ─────────────────────────────────────────────
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
  if (target < 1 || target > total.value) return

  window.dispatchEvent(new CustomEvent('grid:stop-audio'))
  document.body.dataset.navDir = dir
  go(target)
}

const goTo = (row, col) => {
  if (!slideExists(row, col)) return
  const cur  = getPos(currentPage.value)
  const dCol = col - cur.col
  const dRow = row - cur.row
  let dir = 'right'
  if      (Math.abs(dCol) >= Math.abs(dRow)) dir = dCol >= 0 ? 'right' : 'left'
  else                                        dir = dRow >= 0 ? 'down'  : 'up'

  window.dispatchEvent(new CustomEvent('grid:stop-audio'))
  document.body.dataset.navDir = dir
  go(getPage(row, col))
}

// ─── Teclado ─────────────────────────────────────────────
const onKey = (e) => {
  const map = { ArrowRight: 'right', ArrowLeft: 'left', ArrowDown: 'down', ArrowUp: 'up' }
  if (!map[e.key]) return
  e.preventDefault()
  e.stopImmediatePropagation()
  navigate(map[e.key])
}

// ─── Swipe táctil ────────────────────────────────────────
let touchStart = null
const SWIPE_THRESHOLD = 50

const onTouchStart = (e) => {
  touchStart = {
    x: e.touches[0].clientX,
    y: e.touches[0].clientY,
    blocked: false,
  }
}

// Intercepta en capture para que Slidev no reciba gestos horizontales
const onTouchMove = (e) => {
  if (!touchStart) return
  const dx = Math.abs(e.touches[0].clientX - touchStart.x)
  const dy = Math.abs(e.touches[0].clientY - touchStart.y)
  if (dx > dy && dx > 10) {
    e.preventDefault()
    e.stopImmediatePropagation()
    touchStart.blocked = true
  }
}

const onTouchEnd = (e) => {
  if (!touchStart) return
  const dx = e.changedTouches[0].clientX - touchStart.x
  const dy = e.changedTouches[0].clientY - touchStart.y
  touchStart = null

  if (Math.abs(dx) > Math.abs(dy)) {
    if (Math.abs(dx) < SWIPE_THRESHOLD) return
    navigate(dx > 0 ? 'left' : 'right')
  } else {
    if (Math.abs(dy) < SWIPE_THRESHOLD) return
    navigate(dy > 0 ? 'up' : 'down')
  }
}

// ─── Lifecycle ───────────────────────────────────────────
onMounted(() => {
  window.addEventListener('keydown',    onKey,        true)
  window.addEventListener('touchstart', onTouchStart, { passive: true })
  window.addEventListener('touchmove',  onTouchMove,  { passive: false, capture: true })
  window.addEventListener('touchend',   onTouchEnd,   { passive: true })
})

onUnmounted(() => {
  window.removeEventListener('keydown',    onKey,        true)
  window.removeEventListener('touchstart', onTouchStart)
  window.removeEventListener('touchmove',  onTouchMove,  { capture: true })
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
.indicator-row { display: flex; gap: 6px; }
.indicator-dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.25);
  border: 1px solid rgba(255, 255, 255, 0.4);
  cursor: pointer;
  transition: all 0.2s ease;
}
.indicator-dot.exists:hover { background: rgba(255, 255, 255, 0.5); transform: scale(1.2); }
.indicator-dot.active       { background: white; box-shadow: 0 0 8px rgba(255, 255, 255, 0.8); transform: scale(1.3); }
</style>