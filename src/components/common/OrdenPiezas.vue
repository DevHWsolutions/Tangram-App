<template>
  <div class="orden-piezas">
    <h3>Orden de las piezas</h3>
    <ol>
      <li v-for="(pieza, index) in piezas" :key="pieza.id">
        {{ pieza.nombre }} - Color:
        <span :style="{ color: pieza.fill }">{{ pieza.fill }}</span>
      </li>
    </ol>
  </div>
</template>

<script setup>
defineProps({
  piezas: {
    type: Array,
    required: true,
  },
});
</script>

<style scoped>
.orden-piezas {
  background: #f0f0f0;
  padding: 1em;
  border-radius: 8px;
  max-width: 400px;
  font-family: sans-serif;
}
h3 {
  color: var(--black-000000, #000);

  /* Button - Consola DEV */
  font-family: Montserrat;
  font-size: 20px;
  font-style: normal;
  font-weight: 700;
  line-height: normal;
}
ol {
  padding-left: 1.2em;
}
</style>
