<template>
  <div class="estado-juego">
    <p>⏱ Tiempo restante: {{ tiempoRestante }}s</p>
    <p>💥 Intentos restantes: {{ intentosRestantes }}</p>

    <div v-if="juegoFinalizado">
      <p v-if="gano">🎉 ¡Ganaste!</p>
      <p v-else>❌ Fallaste. Intenta de nuevo.</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from "vue";

const props = defineProps({
  totalPiezas: Number,
  piezasCorrectas: Number,
  intentosIniciales: Number,
  duracion: Number, // duración del juego en segundos
});

const emit = defineEmits(["finalizado"]);

const tiempoRestante = ref(props.duracion);
const intentosRestantes = ref(props.intentosIniciales);
const juegoFinalizado = ref(false);

const gano = computed(() => {
  return props.piezasCorrectas / props.totalPiezas >= 0.6;
});

let interval = null;

onMounted(() => {
  interval = setInterval(() => {
    tiempoRestante.value--;
    if (tiempoRestante.value <= 0) {
      finalizarJuego();
    }
  }, 1000);
});

watch(
  () => props.piezasCorrectas,
  (nuevo) => {
    if (nuevo === props.totalPiezas) {
      finalizarJuego();
    }
  }
);

function perderIntento() {
  if (juegoFinalizado.value) return;
  intentosRestantes.value--;
  if (intentosRestantes.value <= 0) {
    finalizarJuego();
  }
}

function finalizarJuego() {
  juegoFinalizado.value = true;
  clearInterval(interval);
  emit("finalizado", { gano: gano.value });
}

defineExpose({ perderIntento });
</script>

<style scoped>
.estado-juego {
  border: 1px solid #999;
  padding: 10px;
  margin-top: 20px;
  font-family: Arial, sans-serif;
}
</style>
