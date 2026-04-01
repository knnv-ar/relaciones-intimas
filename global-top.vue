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

const totalRows   = computed(() => Math.ceil(total.value / COLS))
const getPos      = (page) => ({ row: Math.floor((page - 1) / COLS), col: (page - 1) % COLS })
const getPage     = (row, col) => row * COLS + col + 1
const isActive    = (row, col) => { const p = getPos(currentPage.value); return p.row === row && p.col === col }
const slideExists = (row, col) => { const p = getPage(row, col); return p >= 1 && p <= total.value }

// ─── Navegar desde una página específica (no desde currentPage) ──────────
// fromPage es el snapshot tomado al inicio del gesto, ANTES de que
// Slidev pueda modificar currentPage con su propio handler.
const navigateFrom = (dir, fromPage) => {
  const { row, col } = getPos(fromPage)

  const targets = {
    right: { row,     col: col + 1 },
    left:  { row,     col: col - 1 },
    down:  { row: row + 1, col     },
    up:    { row: row - 1, col     },
  }
  const next = targets[dir]

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

// Wrapper para teclado y minimap que lee currentPage en el momento
const navigate = (dir) => navigateFrom(dir, currentPage.value)

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

// ─── Teclado ─────────────────────────────────────────────────────────────
// El teclado no tiene conflicto con Slidev porque usamos capture + stopImmediatePropagation
const onKey = (e) => {
  const map = { ArrowRight: 'right', ArrowLeft: 'left', ArrowDown: 'down', ArrowUp: 'up' }
  if (!map[e.key]) return
  e.preventDefault()
  e.stopImmediatePropagation()
  navigate(map[e.key])
}

// ─── Swipe táctil ────────────────────────────────────────────────────────
let touchStart = null
const SWIPE_THRESHOLD = 50

const onTouchStart = (e) => {
  touchStart = {
    x:    e.touches[0].clientX,
    y:    e.touches[0].clientY,
    page: currentPage.value,  // ← SNAPSHOT de la página ANTES de cualquier handler
  }
}

const onTouchEnd = (e) => {
  if (!touchStart) return
  const dx       = e.changedTouches[0].clientX - touchStart.x
  const dy       = e.changedTouches[0].clientY - touchStart.y
  const fromPage = touchStart.page   // ← página real de inicio del gesto
  touchStart = null

  // ─── Diferir con setTimeout(0) ────────────────────────────────────────
  // Esto garantiza que el handler de Slidev (sincrónico) dispare PRIMERO.
  // Cuando nuestro handler corre, Slidev ya navegó desde fromPage a fromPage±1.
  // Nosotros también navegamos a fromPage±1 → go() detecta que ya estamos
  // ahí y no produce una segunda transición. Sin setTimeout, nuestro handler
  // correría ANTES que Slidev, Slidev leería el currentPage ya actualizado
  // por nosotros y navegaría un paso más → doble salto.
  setTimeout(() => {
    if (Math.abs(dx) > Math.abs(dy)) {
      // Gesto horizontal
      if (Math.abs(dx) < SWIPE_THRESHOLD) return
      navigateFrom(dx > 0 ? 'left' : 'right', fromPage)
    } else {
      // Gesto vertical — Slidev no tiene handler vertical, no hay conflicto
      if (Math.abs(dy) < SWIPE_THRESHOLD) return
      navigateFrom(dy > 0 ? 'up' : 'down', fromPage)
    }
  }, 0)
}

// ─── Lifecycle ───────────────────────────────────────────────────────────
onMounted(() => {
  window.addEventListener('keydown',    onKey,        true)
  window.addEventListener('touchstart', onTouchStart, { passive: true })
  window.addEventListener('touchend',   onTouchEnd,   { passive: true })
})

onUnmounted(() => {
  window.removeEventListener('keydown',    onKey,        true)
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
```
