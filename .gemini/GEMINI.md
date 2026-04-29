# Relaciones Íntimas - Project Documentation

## Project Overview
Este proyecto es una presentación interactiva y multimedia desarrollada con **Slidev**. Su propósito es explorar la temática de las "Relaciones Íntimas" a través de una narrativa visual y sonora, diseñada específicamente para el ámbito de las **Artes Digitales**. La estructura combina texto poético/evocativo con soporte de audio y fotografía a pantalla completa.

## Tech Stack
*   **Slidev**: Core del motor de presentaciones.
*   **Vue.js 3**: Framework para componentes personalizados (`<script setup>`).
*   **Howler.js**: Librería para la gestión de audio en el componente `AudioPlayer`.
*   **Vite**: Bundler y servidor de desarrollo.
*   **Vanilla CSS**: Estilizado mediante bloques `<style scoped>` en componentes Vue.

## Coding Standards

### Scripts & Vue Components
*   **Composition API**: Utilizar siempre `<script setup>`.
*   **Dynamic Paths**: Para asegurar que los assets carguen correctamente tanto en local como en despliegues (Netlify, GitHub Pages), se debe usar `import.meta.env.BASE_URL` para resolver rutas de imágenes y audio.
    *   Ejemplo: `const fullUrl = computed(() => base + path)`.
*   **Naming Conventions**:
    *   Componentes: PascalCase (ej: `AudioPlayer.vue`).
    *   Layouts: kebab-case (ej: `fullscreen-card.vue`).
    *   Assets: kebab-case y preferiblemente numerados (ej: `slide-01.jpg`).

### Assets & Media
*   **Imágenes**: Almacenadas en `public/img/`. Formato sugerido: `.jpg` para fotografías de fondo.
*   **Audio**: Almacenados en `public/audio/`. Formato sugerido: `.mp3`.
*   **Componentes de Media**: El uso de `AudioPlayer` es obligatorio en slides que requieran ambiente sonoro.

### Slidev Configuration
*   **Frontmatter**: Cada slide debe definir su `layout` y `image` si corresponde.
*   **Transitions**: El proyecto utiliza `transition: grid-nav` por defecto para mantener la consistencia visual.

## Contexto de Contenido
*   **Audiencia**: Estudiantes universitarios.
*   **Tono**: Un equilibrio entre el **rigor académico** (teoría de la comunicación, psicología, artes visuales) y la **experimentación creativa** propia de las Artes Digitales.
*   **Complejidad**: Se busca profundidad conceptual sin perder la accesibilidad visual. Al generar contenido, priorizar metáforas potentes y lenguaje que invite a la reflexión crítica sobre la tecnología y la afectividad.

## Git Workflow
*   **Ignored Files**: No procesar ni leer contenidos dentro de `node_modules`, `dist`, `.DS_Store` o archivos de configuración local identificados en `.gitignore`.
