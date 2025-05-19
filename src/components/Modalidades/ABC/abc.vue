<template>
  <div class="contenedor-principal">
    <!-- Puedes colocarlos donde quieras en tu interfaz -->

    <!-- Canvas a la izquierda -->
    <v-stage
      ref="stage"
      :config="stageSize"
      @mousedown="handleStageMouseDown"
      @touchstart="handleStageMouseDown"
    >
      <v-layer ref="layer">
        <!-- Piezas objetivo -->
        <v-group
          v-for="(templatePiece, index) in triangulo"
          :key="templatePiece.id"
        >
          <v-shape :config="getTemplateShapeConfig(templatePiece)" />
          <v-text :config="getTemplateTextConfig(templatePiece, index + 1)" />
        </v-group>

        <!-- Piezas del usuario -->
        <v-shape
          v-for="piece in userPieces"
          :key="piece.id"
          :config="getUserShapeConfig(piece)"
          @dragend="handleDragEnd"
        />
      </v-layer>
    </v-stage>

    <!-- <RotadorModal
      :visible="mostrarModalRotar"
      :selectedId="selectedShapeId"
      @close="mostrarModalRotar = false"
      @rotate="rotarFigura"
    /> -->

    <!-- Info a la derecha -->
    <div class="panel-lateral">
      <p>Piezas correctas: {{ correctPiecesCount }} / {{ triangulo.length }}</p>
      <OrdenPiezas :piezas="userPieces" />
      <!-- Componente para el estatus del Juego  -->

      <!-- <EstadoJuego
        ref="estadoJuego"
        :totalPiezas="triangulo.length"
        :piezasCorrectas="correctPiecesCount"
        :intentosIniciales="6"
        :duracion="60"
        @finalizado="manejarFinJuego"
      /> -->
    </div>
    <div v-if="selectedShapeId" class="acciones-rotacion">
      <span class="titulo-acciones-rotacion">Acciones:</span>
      <span>Pieza:</span>
      <span>{{ selectedShapeId }}</span>
      <button @click="rotarSeleccion(45)">Girar 45° Derecha</button>
      <button @click="rotarSeleccion(-45)">Girar 45° Izquierda</button>
    </div>
  </div>
</template>

<style scoped>
.contenedor-principal {
  display: flex;
  align-items: flex-start;
  gap: 20px;
}

.panel-lateral {
  width: 250px;
  padding: 10px;
  border-left: 2px solid #ccc;
}
</style>

<script setup>
import { ref, computed } from "vue";
import { triangulo as trianguloData } from "../../../trianguloABC.js"; // Asegúrate de que la ruta sea correcta
import OrdenPiezas from "../../common/OrdenPiezas.vue";
import EstadoJuego from "../../common/EstadoJuego.vue";
import RotalModal from "../../common/RotarModal.vue";
const estadoJuego = ref(null);

const mostrarModalRotar = ref(false);
function manejarFinJuego(resultado) {
  if (resultado.gano) {
    alert("¡Felicidades, ganaste!");
  } else {
    alert("Lo siento, no alcanzaste el 60% de piezas correctas.");
  }
}
const stageSize = ref({
  width: 900,
  height: 650,
});
const triangulo = ref(trianguloData);
const userPieces = ref([
  {
    id: "up1",
    templateId: "p1",
    nombre: "Triángulo Grande 1",
    points: [...trianguloData.find((p) => p.id === "p1").points],
    x: 750,
    y: 0,
    fill: "skyblue",
    rotation: 45,
    draggable: true,
  },
  {
    id: "up2",
    templateId: "p2",
    nombre: "Triángulo Grande 2",
    points: [...trianguloData.find((p) => p.id === "p2").points],
    x: 750,
    y: 150,
    fill: "orange",
    rotation: 45,
    draggable: true,
  },
  {
    id: "up3",
    templateId: "p3",
    nombre: "Triángulo Mediano",
    points: [...trianguloData.find((p) => p.id === "p3").points],
    x: 685,
    y: 310,
    fill: "magenta",
    rotation: 0,
    draggable: true,
  },
  {
    id: "up4",
    templateId: "p4",
    nombre: "Triángulo Pequeño 1",
    points: [...trianguloData.find((p) => p.id === "p4").points],
    x: 800,
    y: 370,
    fill: "lime",
    rotation: 315,
    draggable: true,
  },
  {
    id: "up5",
    templateId: "p5",
    nombre: "Triángulo Pequeño 2",
    points: [...trianguloData.find((p) => p.id === "p5").points],
    x: 785,
    y: 375,
    fill: "yellow",
    rotation: 45,
    draggable: true,
  },
  {
    id: "up6",
    templateId: "p6",
    nombre: "Cuadrado",
    points: [...trianguloData.find((p) => p.id === "p6").points],
    x: 700,
    y: 500,
    fill: "blue",
    rotation: 45,
    draggable: true,
  },
  {
    id: "up7",
    templateId: "p7",
    nombre: "Paralelogramo",
    points: [...trianguloData.find((p) => p.id === "p7").points],
    x: 800,
    y: 520,
    fill: "teal",
    rotation: 180,
    draggable: true,
  },
]);
const correctPieces = ref({});
const correctPiecesCount = ref(0);

const templateColor = "gray";
const templateOpacity = 0.5;
const positionTolerance = 2;
const rotationTolerance = 5;

const getTemplateShapeConfig = (piece) => ({
  x: piece.x,
  y: piece.y,
  fill: templateColor,
  fillOpacity: templateOpacity,
  stroke: "black",
  strokeWidth: 1,
  id: `template-${piece.id}`,
  rotation: piece.rotation,
  sceneFunc: (context, shape) => {
    context.beginPath();
    context.moveTo(piece.points[0], piece.points[1]);
    for (let i = 2; i < piece.points.length; i += 2) {
      context.lineTo(piece.points[i], piece.points[i + 1]);
    }
    context.closePath();
    context.fillStrokeShape(shape);
  },
});

const getTemplateTextConfig = (piece, index) => ({
  x: piece.x + 10,
  y: piece.y + 10,
  text: index.toString(),
  fontSize: 20,
  fill: "black",
  opacity: 0.8,
});

const handleDragEnd = (e) => {
  console.log("Drag end triggered");

  const id = e.target.id();
  const draggedPiece = userPieces.value.find((p) => p.id === id);
  if (!draggedPiece) {
    console.log("No piece found for id:", id);
    return;
  }

  // ✅ ACTUALIZAR posición y rotación
  draggedPiece.x = e.target.x();
  draggedPiece.y = e.target.y();
  draggedPiece.rotation = e.target.rotation();

  console.log("Dragged piece (updated):", draggedPiece);

  const guidePiece = triangulo.value.find(
    (p) => p.id === draggedPiece.templateId
  );

  if (guidePiece) {
    const isCorrectPosition =
      Math.abs(draggedPiece.x - guidePiece.x) < positionTolerance &&
      Math.abs(draggedPiece.y - guidePiece.y) < positionTolerance;
    const isCorrectRotation =
      Math.abs(draggedPiece.rotation - guidePiece.rotation) % 360 <
      rotationTolerance;

    console.log("Position check:", isCorrectPosition);
    console.log("Rotation check:", isCorrectRotation);

    if (
      isCorrectPosition &&
      isCorrectRotation &&
      !correctPieces.value[draggedPiece.id]
    ) {
      correctPieces.value[draggedPiece.id] = true;
      correctPiecesCount.value++;
      draggedPiece.draggable = false;

      console.log(`${draggedPiece.nombre} colocada correctamente.`);
    } else {
      console.log(`${draggedPiece.nombre} colocada incorrectamente.`);
      // 👇 Aquí restamos un intento si fue incorrecta
      estadoJuego.value?.perderIntento();
    }
  }
};

const getUserShapeConfig = (piece) => ({
  x: piece.x,
  y: piece.y,
  fill: piece.fill,
  stroke: "black",
  strokeWidth: 2,
  draggable: piece.draggable,
  id: piece.id,
  rotation: piece.rotation,
  sceneFunc: (context, shape) => {
    context.beginPath();
    context.moveTo(piece.points[0], piece.points[1]);
    for (let i = 2; i < piece.points.length; i += 2) {
      context.lineTo(piece.points[i], piece.points[i + 1]);
    }
    context.closePath();
    context.fillStrokeShape(shape);
  },
  // on: {
  //   dragend: handleDragEnd,
  // },
});

const selectedShapeId = ref("");

const handleStageMouseDown = (e) => {
  if (e.target === e.target.getStage()) {
    selectedShapeId.value = "";
    updateTransformer();
    return;
  }

  const id = e.target.id();
  // Asegúrate de que solo los userPieces sean seleccionables
  const piece = userPieces.value.find((p) => p.id === id);
  if (piece) {
    selectedShapeId.value = id;
  } else {
    selectedShapeId.value = "";
  }
};

//Función de la rotación
function rotarSeleccion(grados) {
  const pieza = userPieces.value.find((p) => p.id === selectedShapeId.value);
  if (pieza && pieza.draggable) {
    pieza.rotation = (pieza.rotation + grados) % 360;
  }
}

//Funcion para limitar que se salga de la pantalla
</script>

<style scoped>
.v-stage {
  outline: 2px solid black;
  border: 1px solid #ccc;
}
p {
  padding-top: 10px;
}
.acciones-rotacion {
  outline: 2px solid black;
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 5px;
}
.acciones-rotacion button {
  margin: 5px;
}
.titulo-acciones-rotacion {
  color: var(--black-36-a-7680, #6a7680);
  text-align: center;

  /* Button - Consola DEV */
  font-family: Montserrat;
  font-size: 16px;
  font-style: normal;
  font-weight: 700;
  line-height: normal;
}
</style>
