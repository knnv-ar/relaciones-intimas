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

// ─── Limpiar data-nav-dir tras la transición ─────────────
// NO usamos transitionend porque se dispara en TODOS los elementos
// (dots, botones, overlays) mucho antes que termine la transición
// principal del slide (450ms), cayendo al fade fallback.
let navDirTimer = null

const setNavDir = (dir) => {
  document.body.dataset.navDir = dir
  clearTimeout(navDirTimer)
  navDirTimer = setTimeout(() => {
    delete document.body.dataset.navDir
  }, 500) // 500ms > 450ms de la transición CSS en style.css
}

// ─── Navegar desde un snapshot de página ─────────────────
const navigateFrom = (dir, fromPage) => {
  const { row, col } = getPos(fromPage)

  const next = {
    right: { row,     col: col + 1 },
    left:  { row,     col: col - 1 },
    down:  { row: row + 1, col     },
    up:    { row: row - 1, col     },
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
  go(target)
}

const navigate = (dir) => {
  setNavDir(dir)
  navigateFrom(dir, currentPage.value)
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
  setNavDir(dir)
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
    x:    e.touches[0].clientX,
    y:    e.touches[0].clientY,
    page: currentPage.value,
  }
}

const onTouchEnd = (e) => {
  if (!touchStart) return
  const dx       = e.changedTouches[0].clientX - touchStart.x
  const dy       = e.changedTouches[0].clientY - touchStart.y
  const fromPage = touchStart.page
  touchStart = null

  let dir = null
  if (Math.abs(dx) > Math.abs(dy)) {
    if (Math.abs(dx) >= SWIPE_THRESHOLD) dir = dx > 0 ? 'left' : 'right'
  } else {
    if (Math.abs(dy) >= SWIPE_THRESHOLD) dir = dy > 0 ? 'up' : 'down'
  }

  if (!dir) return

  setNavDir(dir)   // ← síncrono: la transición CSS ya tiene la dirección correcta
  window.dispatchEvent(new CustomEvent('grid:stop-audio'))

  setTimeout(() => {
    navigateFrom(dir, fromPage)
  }, 0)
}

// ─── Lifecycle ───────────────────────────────────────────
onMounted(() => {
  window.addEventListener('keydown',    onKey,        true)
  window.addEventListener('touchstart', onTouchStart, { passive: true })
  window.addEventListener('touchend',   onTouchEnd,   { passive: true })
})

onUnmounted(() => {
  clearTimeout(navDirTimer)
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

---

## Resumen del bug y el fix
```
ANTES (bug):
  setea data-nav-dir = "right"
  dot.transitionend (200ms) → borra data-nav-dir   ← demasiado pronto!
  slide.transition (450ms)  → lee body sin atributo → fade fallback ✗

AHORA:
  setNavDir("right") → data-nav-dir = "right"
  setTimeout(500ms)  → borra data-nav-dir           ← después de los 450ms ✓
  slide.transition (450ms) → lee "right" → slide-right ✓