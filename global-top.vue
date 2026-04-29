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

// ─── Gestión de data-nav-dir ──────────────────────────────
let navDirTimer = null

const setNavDir = (dir) => {
  document.body.dataset.navDir = dir
  clearTimeout(navDirTimer)
  navDirTimer = setTimeout(() => {
    delete document.body.dataset.navDir
  }, 500)
}

// ─── Calcular página destino sin navegar ──────────────────
const getTarget = (dir, fromPage) => {
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
  ) return null

  const target = getPage(next.row, next.col)
  return (target >= 1 && target <= total.value) ? target : null
}

// ─── Navegar ──────────────────────────────────────────────
const navigateFrom = (dir, fromPage) => {
  const target = getTarget(dir, fromPage)
  if (target === null) return
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
const DIR_DETECT_PX   = 12

const onTouchStart = (e) => {
  touchStart = {
    x:    e.touches[0].clientX,
    y:    e.touches[0].clientY,
    page: currentPage.value,
    dir:  null,
  }
}

// Detectar dirección en touchmove para que data-nav-dir esté
// presente antes de que touchend dispare la transición
const onTouchMove = (e) => {
  if (!touchStart || touchStart.dir) return

  const dx = e.touches[0].clientX - touchStart.x
  const dy = e.touches[0].clientY - touchStart.y

  if (Math.abs(dx) < DIR_DETECT_PX && Math.abs(dy) < DIR_DETECT_PX) return

  touchStart.dir = Math.abs(dx) > Math.abs(dy)
    ? (dx > 0 ? 'left' : 'right')
    : (dy > 0 ? 'up'   : 'down')

  setNavDir(touchStart.dir)
}

const onTouchEnd = (e) => {
  if (!touchStart) return

  const dx       = e.changedTouches[0].clientX - touchStart.x
  const dy       = e.changedTouches[0].clientY - touchStart.y
  const fromPage = touchStart.page
  const dir      = touchStart.dir
  touchStart = null

  if (!dir) return
  if (Math.abs(dx) > Math.abs(dy) && Math.abs(dx) < SWIPE_THRESHOLD) return
  if (Math.abs(dy) >= Math.abs(dx) && Math.abs(dy) < SWIPE_THRESHOLD) return

  window.dispatchEvent(new CustomEvent('grid:stop-audio'))

  const isHorizontal = Math.abs(dx) > Math.abs(dy)

  if (isHorizontal) {
    // ── Horizontal: Slidev tiene handler propio → diferimos y solo
    //    navegamos si Slidev NO llegó al destino correcto (evita doble go)
    const target = getTarget(dir, fromPage)
    setTimeout(() => {
      if (target !== null && currentPage.value !== target) {
        go(target)
      }
    }, 0)
  } else {
    // ── Vertical: Slidev NO tiene handler → navegamos directamente,
    //    sin setTimeout para evitar el frame de flash
    navigateFrom(dir, fromPage)
  }
}

// ─── Lifecycle ───────────────────────────────────────────
onMounted(() => {
  window.addEventListener('keydown',    onKey,        true)
  window.addEventListener('touchstart', onTouchStart, { passive: true })
  window.addEventListener('touchmove',  onTouchMove,  { passive: true })
  window.addEventListener('touchend',   onTouchEnd,   { passive: true })
})

onUnmounted(() => {
  clearTimeout(navDirTimer)
  window.removeEventListener('keydown',    onKey,        true)
  window.removeEventListener('touchstart', onTouchStart)
  window.removeEventListener('touchmove',  onTouchMove)
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

## Resumen del fix por caso
```
Swipe derecha (navegar "left") — horizontal:
  ANTES: Slidev go(1) ✓ → setTimeout: nuestro go(1) → flash ✗
  AHORA: Slidev go(1) ✓ → setTimeout: currentPage(1) === target(1) → no-op ✓

Swipe arriba (navegar "up") — vertical:
  ANTES: setTimeout(0) → delay de 1 frame → flash del slide origen ✗
  AHORA: go() inmediato sin setTimeout → sin delay → sin flash ✓