<template>
  <div
    class="fullscreen-two-cols"
    :style="fullImageUrl ? { backgroundImage: `url('${fullImageUrl}')` } : {}"
  >
    <div class="overlay" />
    <div class="cols-wrapper">
      <div class="col col-left">
        <slot name="left" />
      </div>
      <div class="col col-right">
        <slot name="right" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  image:    { type: String, default: '' },
  position: { type: String, default: 'top' },
  // 'top'    → contenido desde arriba
  // 'center' → contenido centrado verticalmente
})

const fullImageUrl = computed(() => {
  if (!props.image) return ''
  const base = import.meta.env.BASE_URL.replace(/\/$/, '')
  const path = props.image.startsWith('/') ? props.image : '/' + props.image
  return base + path
})
</script>

<style scoped>
.fullscreen-two-cols {
  position: relative;
  width: 100%;
  height: 100%;
  background-color: #111;
  background-size: cover;
  background-position: center;
  overflow: hidden;
}

.overlay {
  position: absolute;
  inset: 0;
  background: linear-gradient(
    to top,
    rgba(0, 0, 0, 0.88) 0%,
    rgba(0, 0, 0, 0.35) 50%,
    rgba(0, 0, 0, 0.10) 100%
  );
}

.cols-wrapper {
  position: relative;
  z-index: 1;
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 2rem;
  width: 100%;
  height: 100%;
  padding: 2rem 2.5rem;
  box-sizing: border-box;
  align-items: v-bind("props.position === 'center' ? 'center' : 'start'");
}

.col {
  color: white;
  overflow: hidden;
}

/* Neutralizar text-shadow heredado del tema en párrafos */
.col :deep(p),
.col :deep(h1),
.col :deep(h2),
.col :deep(h3),
.col :deep(h4),
.col :deep(h5) {
  text-shadow: none;
  color: white;
}
</style>