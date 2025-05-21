<template>
  <div>
    <div class="menu">
      <button @click="currentView = 'abc'">ABC</button>
      <button @click="currentView = 'mentor'">MENTOR</button>
      <button @click="currentView = 'elite'">ELITE</button>

      <button @click="currentView = 'ConfiguracionDibujos'" v-show="false">
        Diseño Dibujos
      </button>
    </div>

    <div class="contenido">
      <ABCComponent v-if="currentView === 'abc'" />
      <MentorComponent v-else-if="currentView === 'mentor'" />
      <EliteComponent v-else-if="currentView === 'elite'" />
      <Configuracion
        v-else-if="currentView === 'ConfiguracionDibujos'"
        v-show="false"
      />
      <p v-else>Selecciona una opción</p>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import ABCComponent from "./components/Modalidades/ABC/abc.vue";
import MentorComponent from "./components/Modalidades/MENTOR/mentor.vue";
import EliteComponent from "./components/Modalidades/ELITE/elite.vue";
import Configuracion from "./Settings/ConfiguracionDibujos.vue";
const currentView = ref("");
</script>

<style scoped>
.menu {
  display: flex;
  margin-right: 10px;
  margin-bottom: 20px;
}
.contenido {
  border-top: 1px solid #ccc;
  padding-top: 20px;
}
</style>
