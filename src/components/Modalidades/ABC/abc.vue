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

      <EstadoJuego
        ref="estadoJuego"
        :totalPiezas="triangulo.length"
        :piezasCorrectas="correctPiecesCount"
        :intentosIniciales="6"
        :duracion="60"
        @finalizado="manejarFinJuego"
      />

      <div v-if="selectedShapeId" class="acciones-rotacion">
        <span class="titulo-acciones-rotacion">Acciones:</span>
        <span>Pieza:</span>
        <span>{{ selectedShapeId }}</span>
        <button @click="rotarSeleccion(45)">Girar 45° Derecha</button>
        <button @click="rotarSeleccion(-45)">Girar 45° Izquierda</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.contenedor-principal {
  border-radius: 32px;
  background: #eff3fb;
  display: flex;
  align-items: flex-start;
  margin: 5px;
}

.panel-lateral {
  width: 150x;
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
// ✅ Aquí guardamos posición y rotación original
userPieces.value = userPieces.value.map((p) => ({
  ...p,
  originalX: p.x,
  originalY: p.y,
  originalRotation: p.rotation,
}));
const correctPieces = ref({});
const correctPiecesCount = ref(0);

const templateColor = "gray";
const templateOpacity = 0.5;
const positionTolerance = 20;
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

// const handleDragEnd = (e) => {
//   console.log("Drag end triggered");

//   const id = e.target.id();
//   const draggedPiece = userPieces.value.find((p) => p.id === id);
//   if (!draggedPiece) {
//     console.log("No piece found for id:", id);
//     return;
//   }

//   // ✅ ACTUALIZAR posición y rotación
//   draggedPiece.x = e.target.x();
//   draggedPiece.y = e.target.y();
//   draggedPiece.rotation = e.target.rotation();

//   console.log("Dragged piece (updated):", draggedPiece);

//   const guidePiece = triangulo.value.find(
//     (p) => p.id === draggedPiece.templateId
//   );
//   // Lo guiamos como iman para que se coloque normal la pieza
//   if (guidePiece) {
//     const isCorrectPosition =
//       Math.abs(draggedPiece.x - guidePiece.x) < positionTolerance &&
//       Math.abs(draggedPiece.y - guidePiece.y) < positionTolerance;
//     const isCorrectRotation =
//       Math.abs(draggedPiece.rotation - guidePiece.rotation) % 360 <
//       rotationTolerance;

//     console.log("Position check:", isCorrectPosition);
//     console.log("Rotation check:", isCorrectRotation);

//     if (
//       isCorrectPosition &&
//       isCorrectRotation &&
//       !correctPieces.value[draggedPiece.id]
//     ) {
//       correctPieces.value[draggedPiece.id] = true;
//       correctPiecesCount.value++;

//       // 🔁 Forzar que se pegue EXACTAMENTE al contorno
//       draggedPiece.x = guidePiece.x;
//       draggedPiece.y = guidePiece.y;
//       draggedPiece.rotation = guidePiece.rotation;

//       draggedPiece.draggable = false;

//       console.log(`${draggedPiece.nombre} colocada correctamente.`);
//     } else {
//       console.log(`${draggedPiece.nombre} colocada incorrectamente.`);
//       // 👇 Aquí restamos un intento si fue incorrecta
//       estadoJuego.value?.perderIntento();
//     }
//   }

//   // 🧪 ¿Qué hace este código?
//   // 🔁 Recorre cada punto de tu figura.

//   // 🌀 Aplica la rotación y la posición actual.

//   // 📏 Verifica si se sale del canvas.

//   // 🎯 Ajusta x e y solo lo necesario para meter todos los puntos otra vez dentro.
//   const stageWidth = stageSize.value.width;
//   const stageHeight = stageSize.value.height;

//   let seSale = false;

//   for (let i = 0; i < draggedPiece.points.length; i += 2) {
//     const px = draggedPiece.points[i];
//     const py = draggedPiece.points[i + 1];

//     // Aplica rotación
//     const angle = draggedPiece.rotation * (Math.PI / 180);
//     const cos = Math.cos(angle);
//     const sin = Math.sin(angle);

//     const rotatedX = px * cos - py * sin;
//     const rotatedY = px * sin + py * cos;

//     // Traslada al mundo (posición en el canvas)
//     const worldX = rotatedX + draggedPiece.x;
//     const worldY = rotatedY + draggedPiece.y;

//     // Verifica si el punto está fuera del canvas
//     if (
//       worldX < 0 ||
//       worldX > stageWidth ||
//       worldY < 0 ||
//       worldY > stageHeight
//     ) {
//       seSale = true;
//       break;
//     }
//   }

//   if (seSale) {
//     console.log("Se salió del canvas. Regresando a su posición original.");

//     // ✨ ANIMACIÓN: regresa suavemente a su posición original
//     e.target.to({
//       x: draggedPiece.originalX,
//       y: draggedPiece.originalY,
//       rotation: draggedPiece.originalRotation,
//       duration: 0.5,
//       easing: Konva.Easings.EaseInOut,
//     });

//     // 🔁 También actualizamos los datos internos
//     draggedPiece.x = draggedPiece.originalX;
//     draggedPiece.y = draggedPiece.originalY;
//     draggedPiece.rotation = draggedPiece.originalRotation;
//   }
// };

// const getUserShapeConfig = (piece) => ({
//   x: piece.x,
//   y: piece.y,
//   fill: piece.fill,
//   stroke: "black",
//   strokeWidth: 2,
//   draggable: piece.draggable,
//   id: piece.id,
//   rotation: piece.rotation,
//   sceneFunc: (context, shape) => {
//     context.beginPath();
//     context.moveTo(piece.points[0], piece.points[1]);
//     for (let i = 2; i < piece.points.length; i += 2) {
//       context.lineTo(piece.points[i], piece.points[i + 1]);
//     }
//     context.closePath();
//     context.fillStrokeShape(shape);
//   },
//   // on: {
//   //   dragend: handleDragEnd,
//   // },
// });

const handleDragEnd = (e) => {
  //   ✅ Ahora cubre:
  // ✔️ Coloca pieza correctamente → se queda y suma punto.

  // ❌ Coloca en cualquier guía incorrecta → vuelve con animación + resta intento.

  // ⚠️ Coloca fuera del canvas → vuelve con animación (sin penalización).

  // 🕊️ Coloca lejos de cualquier guía → no pasa nada.
  const id = e.target.id();
  const draggedPiece = userPieces.value.find((p) => p.id === id);
  if (!draggedPiece) return;

  // Actualizamos posición y rotación
  draggedPiece.x = e.target.x();
  draggedPiece.y = e.target.y();
  draggedPiece.rotation = e.target.rotation();

  const guidePiece = triangulo.value.find(
    (p) => p.id === draggedPiece.templateId
  );

  // Verificamos colocación cerca de alguna guía
  const anyGuide = triangulo.value.find((p) => {
    const closeEnough =
      Math.abs(draggedPiece.x - p.x) < positionTolerance &&
      Math.abs(draggedPiece.y - p.y) < positionTolerance;
    return closeEnough;
  });

  const isCorrectPosition =
    guidePiece &&
    Math.abs(draggedPiece.x - guidePiece.x) < positionTolerance &&
    Math.abs(draggedPiece.y - guidePiece.y) < positionTolerance;

  const isCorrectRotation =
    guidePiece &&
    Math.abs(draggedPiece.rotation - guidePiece.rotation) % 360 <
      rotationTolerance;

  const esCorrecta = isCorrectPosition && isCorrectRotation;

  if (esCorrecta && !correctPieces.value[draggedPiece.id]) {
    // ✅ Pieza colocada correctamente
    correctPieces.value[draggedPiece.id] = true;
    correctPiecesCount.value++;

    draggedPiece.x = guidePiece.x;
    draggedPiece.y = guidePiece.y;
    draggedPiece.rotation = guidePiece.rotation;
    draggedPiece.draggable = false;

    e.target.to({
      x: guidePiece.x,
      y: guidePiece.y,
      rotation: guidePiece.rotation,
      duration: 0.4,
      easing: Konva.Easings.EaseInOut,
    });

    return;
  }

  // ❌ Si se soltó sobre una guía (cualquiera) pero fue incorrecta
  if (anyGuide && !esCorrecta) {
    console.log(
      `${draggedPiece.nombre} colocada incorrectamente sobre una guía.`
    );
    estadoJuego.value?.perderIntento(); // 🔻 Resta intento

    e.target.to({
      x: draggedPiece.originalX,
      y: draggedPiece.originalY,
      rotation: draggedPiece.originalRotation,
      duration: 0.4,
      easing: Konva.Easings.EaseInOut,
    });

    draggedPiece.x = draggedPiece.originalX;
    draggedPiece.y = draggedPiece.originalY;
    draggedPiece.rotation = draggedPiece.originalRotation;

    return;
  }

  // 🔁 Validar si se salió del canvas
  const stageWidth = stageSize.value.width;
  const stageHeight = stageSize.value.height;

  let seSale = false;

  for (let i = 0; i < draggedPiece.points.length; i += 2) {
    const px = draggedPiece.points[i];
    const py = draggedPiece.points[i + 1];

    const angle = draggedPiece.rotation * (Math.PI / 180);
    const cos = Math.cos(angle);
    const sin = Math.sin(angle);

    const rotatedX = px * cos - py * sin;
    const rotatedY = px * sin + py * cos;

    const worldX = rotatedX + draggedPiece.x;
    const worldY = rotatedY + draggedPiece.y;

    if (
      worldX < 0 ||
      worldX > stageWidth ||
      worldY < 0 ||
      worldY > stageHeight
    ) {
      seSale = true;
      break;
    }
  }

  if (seSale) {
    console.log("Se salió del canvas. Regresando a su posición original.");

    e.target.to({
      x: draggedPiece.originalX,
      y: draggedPiece.originalY,
      rotation: draggedPiece.originalRotation,
      duration: 0.4,
      easing: Konva.Easings.EaseInOut,
    });

    draggedPiece.x = draggedPiece.originalX;
    draggedPiece.y = draggedPiece.originalY;
    draggedPiece.rotation = draggedPiece.originalRotation;
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
  // dragBoundFunc: (pos) => { ... } <-- esto ya no va
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
const limitDragWithinBounds = (e) => {
  const shape = e.target;
  const stage = shape.getStage();
  const stageWidth = stage.width();
  const stageHeight = stage.height();

  const box = shape.getClientRect();

  let newX = shape.x();
  let newY = shape.y();

  if (box.x < 0) {
    newX -= box.x;
  }

  if (box.y < 0) {
    newY -= box.y;
  }

  if (box.x + box.width > stageWidth) {
    newX -= box.x + box.width - stageWidth;
  }

  if (box.y + box.height > stageHeight) {
    newY -= box.y + box.height - stageHeight;
  }

  shape.position({ x: newX, y: newY });
};

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
  border: 1px solid #ccc;
}
p {
  padding-top: 10px;
}
.acciones-rotacion {
  /* outline: 2px solid black; */
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 2rem;
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
