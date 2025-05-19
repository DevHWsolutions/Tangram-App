<!-- components/RotadorModal.vue -->
<template>
  <div v-if="visible" class="modal-backdrop" @click.self="cerrar">
    <div class="modal-content">
      <h3>Rotar Pieza</h3>
      <div class="botones">
        <button @click="rotar(-15)">⟲ -15°</button>
        <button @click="rotar(15)">⟳ +15°</button>
      </div>
      <button class="cerrar" @click="cerrar">Cerrar</button>
    </div>
  </div>
</template>

<script setup>
import { defineProps, defineEmits } from "vue";

const props = defineProps({
  visible: Boolean,
  selectedId: String,
});

const emit = defineEmits(["close", "rotate"]);

function rotar(grados) {
  emit("rotate", grados);
}

function cerrar() {
  emit("close");
}
</script>

<style scoped>
.modal-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.3);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 999;
}

.modal-content {
  background-color: white;
  padding: 1rem 2rem;
  border-radius: 12px;
  text-align: center;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
}

.botones {
  display: flex;
  justify-content: space-between;
  margin: 1rem 0;
  gap: 10px;
}

button {
  padding: 8px 16px;
  border: none;
  background: #007bff;
  color: white;
  font-size: 1rem;
  border-radius: 6px;
  cursor: pointer;
  transition: background 0.2s;
}

button:hover {
  background: #0056b3;
}

.cerrar {
  background: gray;
}
</style>
