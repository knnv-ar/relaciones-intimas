<template>
  <div class="catalog-outer" :style="outerStyle">
    <div class="catalog-inner" ref="inner" :style="innerStyle">
      <div class="card">
        <p class="main-entry">Estévez, Luciana</p>
        <p class="entry">
          Relaciones íntimas : reflexiones artísticas sobre el campo vincular /
          Luciana Estévez … [et al.]. - 1a ed. - Ciudad Autónoma de Buenos Aires :
          Dramáticas UNA. Departamento de Artes Dramáticas, 2026.
        </p>
        <div class="indent-block">
          <p>Libro digital, HTML</p>
        </div>

        <div class="spacer"></div>

        <div class="indent-block">
          <p class="line">Archivo Digital: online</p>
          <p class="line">ISBN 978-987-48853-8-8</p>
        </div>

        <div class="spacer"></div>

        <p class="subjects">1. Artes Escénicas. 2. Música. 3. Danza.</p>
        <p class="author-entry">I. Estévez, Luciana</p>
        <p class="cdd">CDD 700</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const props = defineProps({
  scale: { type: Number, default: 1   },
  align: { type: String, default: 'center' }, // 'left' | 'center' | 'right'
})

const inner         = ref(null)
const naturalHeight = ref(0)

onMounted(() => {
  naturalHeight.value = inner.value.offsetHeight
})

const outerStyle = computed(() => ({
  height:         naturalHeight.value
                    ? `${naturalHeight.value * props.scale}px`
                    : 'auto',
  width:          '100%',
  overflow:       'visible',
  display:        'flex',
  justifyContent: props.align === 'left'  ? 'flex-start' :
                  props.align === 'right' ? 'flex-end'   : 'center',
}))

const innerStyle = computed(() => ({
  transform:       `scale(${props.scale})`,
  transformOrigin: `top ${props.align === 'left'  ? 'left'  :
                          props.align === 'right' ? 'right' : 'center'}`,
  width: '100%',
}))
</script>

<style scoped>
.catalog-outer { }

.catalog-inner {
  width: 100%;
}

.card {
  background: transparent;
  border: 1.5px solid currentColor;
  padding: 1.4rem 1.6rem 1.3rem 1.6rem;
  max-width: 580px;
  width: 100%;
  line-height: 1.55;
  box-shadow: none;
  font-family: 'Linux Libertine', 'Times New Roman', Georgia, serif;
}

.card p,
.card p.main-entry,
.card p.entry,
.card p.subjects,
.card p.author-entry,
.card p.cdd,
.card p.line {
  font-size: 0.97rem !important;
  text-shadow: none !important;
  color: inherit !important;
  margin: 0 !important;
}

.main-entry   { margin-bottom: 0.1rem !important; }
.entry        { text-indent: 2.4rem; }
.indent-block { padding-left: 2.4rem; margin-top: 0.15rem; }
.spacer       { height: 0.75rem; }
.line         { margin-bottom: 0.1rem !important; }
.subjects     { text-indent: 2.4rem; }
.author-entry { text-indent: 2.4rem; }
.cdd          { padding-left: 2.4rem; }
</style>