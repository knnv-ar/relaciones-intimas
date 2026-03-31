<template>
  <div
    class="fullscreen-card"
    :style="fullImageUrl ? { backgroundImage: `url('${fullImageUrl}')` } : {}"
  >
    <div class="overlay" />
    <div class="content">
      <slot />
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  image: { type: String, default: '' },
})

// ← Resuelve la ruta tanto en local (base="/") como en GitHub Pages (base="/relaciones-intimas/")
const fullImageUrl = computed(() => {
  if (!props.image) return ''
  const base = import.meta.env.BASE_URL.replace(/\/$/, '')
  const path = props.image.startsWith('/') ? props.image : '/' + props.image
  return base + path
})
</script>

<style scoped>
.fullscreen-card {
  position: relative;
  width: 100%;
  height: 100%;
  background-color: #111;
  background-size: cover;
  background-position: center;
  display: flex;
  align-items: flex-end;
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
.content {
  position: relative;
  z-index: 1;
  padding: 3rem 3.5rem;
  width: 100%;
  color: white;
}
.content :deep(h1) {
  color: white;
  font-size: 2.8rem;
  font-weight: 700;
  margin: 0 0 0.6rem;
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.9);
  line-height: 1.1;
}
.content :deep(p) {
  color: rgba(255, 255, 255, 0.88);
  font-size: 1.25rem;
  margin: 0 0 1.8rem;
  text-shadow: 0 1px 5px rgba(0, 0, 0, 0.8);
  line-height: 1.5;
}
:deep(.slidev-nav) { display: none !important; }
</style>