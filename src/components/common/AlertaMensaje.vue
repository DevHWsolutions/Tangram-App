<template>
  <transition name="fade">
    <div v-if="visible" class="estado-juego contenedor-alerta">
      <div class="contenido-alerta">
        <span class="emoji-alerta">⚠️</span>
        <p class="texto-alerta">{{ mensaje }}</p>
      </div>
    </div>
  </transition>
</template>

<script setup>
import { ref, onMounted } from "vue";

const props = defineProps({
  mensaje: { type: String, required: true },
  duracion: { type: Number, default: 5000 }, // duración en milisegundos
});

const visible = ref(true);

onMounted(() => {
  setTimeout(() => {
    visible.value = false;
  }, props.duracion);
});
</script>

<style scoped>
.estado-juego {
  max-width: 400px;
  border: 1px solid #999;
  padding: 10px 15px;
  font-family: Arial, sans-serif;
  background-color: #fff8dc;
  color: #444;
  position: fixed;
  top: 20px; /* <- POSICIÓN ARRIBA */
  left: 50%; /* <- CENTRADO */
  transform: translateX(-50%);
  z-index: 9999;
  box-shadow: 0 0 8px rgba(0, 0, 0, 0.1);
  border-left: 5px solid orange;
  border-radius: 5px;
  text-align: left;
}

.contenido-alerta {
  display: flex;
  align-items: flex-start;
  gap: 8px;
}

.emoji-alerta {
  font-size: 20px;
  margin-top: 2px;
}

.texto-alerta {
  font-size: 14px;
  margin: 0;
  line-height: 1.3;
}

/* Fade transition */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
