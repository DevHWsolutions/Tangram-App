<template>
  <div class="contenedor-principal">
    <!-- Canvas a la izquierda -->
    <div class="contenido-central">
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
            @dragmove="handleDragMove"
          />
        </v-layer>
      </v-stage>
    </div>
    <!-- Info a la derecha -->
    <div class="panel-lateral">
      <p>Piezas correctas: {{ correctPiecesCount }} / {{ triangulo.length }}</p>
      <!-- <OrdenPiezas :piezas="userPieces" /> -->
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
        <span>{{ selectedPieceName }}</span>
        <button @click="rotarSeleccion(45)">Girar 45° Derecha</button>
        <button @click="rotarSeleccion(-45)">Girar 45° Izquierda</button>
      </div>
      <AlertaMensaje v-if="mostrar" :mensaje="textoAlerta" :duracion="6000" />
    </div>
  </div>
</template>

<!-- <style scoped>
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
</style> -->
<style scoped>
.contenedor-principal {
  display: flex;
  flex-direction: row; /* Contenido a la izquierda, panel a la derecha */
  border-radius: 32px;
  background: #eff3fb;
  margin: 5px;
  /* width: 100%; */
  /* flex-wrap: wrap; */
}

/* Contenido principal a la izquierda */
.contenido-central {
  flex: 1;
  padding: 10px;
  box-sizing: border-box;
}

/* Panel lateral a la derecha */
.panel-lateral {
  width: 250px;
  padding: 5px;
  border-left: 2px solid #ccc;
  background-color: #fff;
  box-sizing: border-box;
}

/* Responsive para pantallas pequeñas */
@media (max-width: 768px) {
  .contenedor-principal {
    /* outline: 2px solid red; */
    flex-direction: column;
  }

  .panel-lateral {
    background-color: papayawhip;
    width: 100%;
    border-left: none;
    border-top: 2px solid #ccc;
  }
}
</style>
<script setup>
import { ref, computed } from "vue";
import { triangulo as trianguloData } from "../../../CamelloELITE.js"; // Asegúrate de que la ruta sea correcta
import OrdenPiezas from "../../common/OrdenPiezas.vue";
import EstadoJuego from "../../common/EstadoJuego.vue";
import AlertaMensaje from "../../common/AlertaMensaje.vue";
const estadoJuego = ref(null);
const bordeRojoCanvas = ref(false);
const juegoBloqueado = ref(false);
function manejarFinJuego(resultado) {
  juegoBloqueado.value = true;

  if (resultado.gano) {
    alert("¡Felicidades has alcanzado el 60% requerido, ganaste!");
    location.reload(); // 👈 Esto recarga toda la página
  } else {
    alert("Lo siento, no alcanzaste el 60% de piezas correctas.");
    location.reload(); // 👈 Esto recarga toda la página
  }
}
const stageSize = ref({
  width: 900,
  height: 700,
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
    fill: "#F8AE12",
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
    fill: "#F8AE12",
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
    fill: "#F8AE12",
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
    fill: "#F8AE12",
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
    fill: "#F8AE12",
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
    fill: "#F8AE12",
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
    fill: "#F8AE12",
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

const templateColor = "#F08784";
const templateOpacity = 0.5;
const positionTolerance = 75;
const rotationTolerance = 75;

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
//   //   ✅ Ahora cubre:
//   // ✔️ Coloca pieza correctamente → se queda y suma punto.

//   // ❌ Coloca en cualquier guía incorrecta → vuelve con animación + resta intento.

//   // ⚠️ Coloca fuera del canvas → vuelve con animación (sin penalización).

//   // 🕊️ Coloca lejos de cualquier guía → no pasa nada.
//   const id = e.target.id();
//   const draggedPiece = userPieces.value.find((p) => p.id === id);
//   if (!draggedPiece) return;

//   // Actualizamos posición y rotación
//   draggedPiece.x = e.target.x();
//   draggedPiece.y = e.target.y();
//   draggedPiece.rotation = e.target.rotation();

//   const guidePiece = triangulo.value.find(
//     (p) => p.id === draggedPiece.templateId
//   );

//   // Verificamos colocación cerca de alguna guía
//   const anyGuide = triangulo.value.find((p) => {
//     const closeEnough =
//       Math.abs(draggedPiece.x - p.x) < positionTolerance &&
//       Math.abs(draggedPiece.y - p.y) < positionTolerance;
//     return closeEnough;
//   });

//   const isCorrectPosition =
//     guidePiece &&
//     Math.abs(draggedPiece.x - guidePiece.x) < positionTolerance &&
//     Math.abs(draggedPiece.y - guidePiece.y) < positionTolerance;

//   const isCorrectRotation =
//     guidePiece &&
//     Math.abs(draggedPiece.rotation - guidePiece.rotation) % 360 <
//       rotationTolerance;

//   // const esCorrecta = isCorrectPosition && isCorrectRotation;
//   const esCorrecta = isCorrectPosition;

//   if (esCorrecta && !correctPieces.value[draggedPiece.id]) {
//     // ✅ Pieza colocada correctamente
//     correctPieces.value[draggedPiece.id] = true;
//     correctPiecesCount.value++;

//     draggedPiece.x = guidePiece.x;
//     draggedPiece.y = guidePiece.y;
//     draggedPiece.rotation = guidePiece.rotation;
//     draggedPiece.draggable = false;

//     e.target.to({
//       x: guidePiece.x,
//       y: guidePiece.y,
//       rotation: guidePiece.rotation,
//       duration: 0.4,
//       easing: Konva.Easings.EaseInOut,
//     });

//     return;
//   }

//   // ❌ Si se soltó sobre una guía (cualquiera) pero fue incorrecta
//   if (anyGuide && !esCorrecta) {
//     console.log(
//       `${draggedPiece.nombre} colocada incorrectamente sobre una guía.`
//     );
//     estadoJuego.value?.perderIntento(); // 🔻 Resta intento

//     e.target.to({
//       x: draggedPiece.originalX,
//       y: draggedPiece.originalY,
//       rotation: draggedPiece.originalRotation,
//       duration: 0.4,
//       easing: Konva.Easings.EaseInOut,
//     });

//     draggedPiece.x = draggedPiece.originalX;
//     draggedPiece.y = draggedPiece.originalY;
//     draggedPiece.rotation = draggedPiece.originalRotation;

//     return;
//   }

//   // 🔁 Validar si se salió del canvas
//   const stageWidth = stageSize.value.width;
//   const stageHeight = stageSize.value.height;

//   let seSale = false;

//   for (let i = 0; i < draggedPiece.points.length; i += 2) {
//     const px = draggedPiece.points[i];
//     const py = draggedPiece.points[i + 1];

//     const angle = draggedPiece.rotation * (Math.PI / 180);
//     const cos = Math.cos(angle);
//     const sin = Math.sin(angle);

//     const rotatedX = px * cos - py * sin;
//     const rotatedY = px * sin + py * cos;

//     const worldX = rotatedX + draggedPiece.x;
//     const worldY = rotatedY + draggedPiece.y;

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

//     e.target.to({
//       x: draggedPiece.originalX,
//       y: draggedPiece.originalY,
//       rotation: draggedPiece.originalRotation,
//       duration: 0.4,
//       easing: Konva.Easings.EaseInOut,
//     });

//     draggedPiece.x = draggedPiece.originalX;
//     draggedPiece.y = draggedPiece.originalY;
//     draggedPiece.rotation = draggedPiece.originalRotation;
//   }
// };
const handleDragEnd = (e) => {
  if (juegoBloqueado.value) return;

  const id = e.target.id();
  const draggedPiece = userPieces.value.find((p) => p.id === id);
  if (!draggedPiece) return;

  // Actualiza posición y rotación
  draggedPiece.x = e.target.x();
  draggedPiece.y = e.target.y();
  draggedPiece.rotation = e.target.rotation();

  const guidePiece = triangulo.value.find(
    (p) => p.id === draggedPiece.templateId
  );

  const anyGuide = triangulo.value.find((p) => {
    const closeEnough =
      Math.abs(draggedPiece.x - p.x) < positionTolerance &&
      Math.abs(draggedPiece.y - p.y) < positionTolerance;
    return closeEnough;
  });

  // Validación: puntos geométricos
  const puntosCoinciden = (pieza, guia, tolerancia = 25) => {
    if (!pieza || !guia || pieza.points.length !== guia.points.length)
      return false;

    const transformarPuntos = (obj) => {
      const puntosTransformados = [];
      const angle = obj.rotation * (Math.PI / 180);

      for (let i = 0; i < obj.points.length; i += 2) {
        const px = obj.points[i];
        const py = obj.points[i + 1];
        const rotadoX = px * Math.cos(angle) - py * Math.sin(angle);
        const rotadoY = px * Math.sin(angle) + py * Math.cos(angle);
        const worldX = rotadoX + obj.x;
        const worldY = rotadoY + obj.y;
        puntosTransformados.push({ x: worldX, y: worldY });
      }

      return puntosTransformados;
    };

    const puntosPieza = transformarPuntos(pieza);
    const puntosGuia = transformarPuntos(guia);

    // Para cada punto de la pieza, debe haber al menos un punto de la guía cercano
    return puntosPieza.every((puntoPieza) => {
      return puntosGuia.some((puntoGuia) => {
        const dx = puntoPieza.x - puntoGuia.x;
        const dy = puntoPieza.y - puntoGuia.y;
        const distancia = Math.sqrt(dx * dx + dy * dy);
        return distancia <= tolerancia;
      });
    });
  };

  const esCorrecta = puntosCoinciden(draggedPiece, guidePiece);

  // ✅ Pieza colocada correctamente
  if (esCorrecta && !correctPieces.value[draggedPiece.id]) {
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

  // ❌ Está sobre la guía pero mal colocada → Mostrar rojo y contar intento fallido
  if (anyGuide && !esCorrecta) {
    console.log(
      `${draggedPiece.nombre} colocada incorrectamente. Mostrando en rojo.`
    );
    estadoJuego.value?.perderIntento();

    // Color rojo temporal
    const originalFill = e.target.fill();
    e.target.fill("red");
    e.target.getLayer().batchDraw();

    setTimeout(() => {
      e.target.fill(originalFill);

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
    }, 300); // rojo visible 300ms

    return;
  }

  // 🔁 Validación de si se salió del canvas
  const stageWidth = stageSize.value.width;
  const stageHeight = stageSize.value.height;
  let seSale = false;

  for (let i = 0; i < draggedPiece.points.length; i += 2) {
    const px = draggedPiece.points[i];
    const py = draggedPiece.points[i + 1];

    const angle = draggedPiece.rotation * (Math.PI / 180);
    const rotatedX = px * Math.cos(angle) - py * Math.sin(angle);
    const rotatedY = px * Math.sin(angle) + py * Math.cos(angle);
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
    console.log("Se salió del canvas. Regresando a posición original.");

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

//ESta funcion bloquea la sliada de la pieza del canvas
const handleDragMove = (e) => {
  const shape = e.target;
  const id = shape.id();
  const draggedPiece = userPieces.value.find((p) => p.id === id);
  if (!draggedPiece) return;

  const stageWidth = stageSize.value.width;
  const stageHeight = stageSize.value.height;

  let fuera = false;

  for (let i = 0; i < draggedPiece.points.length; i += 2) {
    const px = draggedPiece.points[i];
    const py = draggedPiece.points[i + 1];

    const angle = draggedPiece.rotation * (Math.PI / 180);
    const rotatedX = px * Math.cos(angle) - py * Math.sin(angle);
    const rotatedY = px * Math.sin(angle) + py * Math.cos(angle);
    const worldX = rotatedX + shape.x();
    const worldY = rotatedY + shape.y();

    if (
      worldX < 0 ||
      worldX > stageWidth ||
      worldY < 0 ||
      worldY > stageHeight
    ) {
      fuera = true;
      break;
    }
  }

  if (fuera) {
    // 🔴 Mostrar borde rojo o feedback visual
    bordeRojoCanvas.value = true;

    // Cancelamos el movimiento
    shape.setX(draggedPiece.x); // Regresa a última posición válida
    shape.setY(draggedPiece.y);
  } else {
    bordeRojoCanvas.value = false;

    // Guardamos la posición válida
    draggedPiece.x = shape.x();
    draggedPiece.y = shape.y();
  }

  shape.getLayer().batchDraw();
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

const selectedPieceName = computed(() => {
  const pieza = userPieces.value.find((p) => p.id === selectedShapeId.value);
  return pieza ? pieza.nombre : null;
});

const handleStageMouseDown = (e) => {
  if (e.target === e.target.getStage()) {
    selectedShapeId.value = "";

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
// function rotarSeleccion(grados) {
//   const pieza = userPieces.value.find((p) => p.id === selectedShapeId.value);
//   if (pieza && pieza.draggable) {
//     pieza.rotation = (pieza.rotation + grados) % 360;
//   }
// }
const mostrar = ref(false);
const textoAlerta = ref("");
// function rotarSeleccion(grados) {
//   const pieza = userPieces.value.find((p) => p.id === selectedShapeId.value);
//   if (!pieza || !pieza.draggable) return;

//   const nuevaRotacion = (pieza.rotation + grados) % 360;
//   const angleRad = nuevaRotacion * (Math.PI / 180);

//   const stageWidth = stageSize.value.width;
//   const stageHeight = stageSize.value.height;

//   let seSale = false;

//   for (let i = 0; i < pieza.points.length; i += 2) {
//     const px = pieza.points[i];
//     const py = pieza.points[i + 1];

//     // aplicar rotación a cada punto
//     const rotatedX = px * Math.cos(angleRad) - py * Math.sin(angleRad);
//     const rotatedY = px * Math.sin(angleRad) + py * Math.cos(angleRad);

//     // trasladar al mundo real (posición actual de la pieza)
//     const worldX = rotatedX + pieza.x;
//     const worldY = rotatedY + pieza.y;

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

//   if (!seSale) {
//     pieza.rotation = nuevaRotacion;
//   } else {
//     // console.log("❌ No se puede rotar: la pieza se saldría del canvas.");
//     // alert(
//     //   "⚠️ No se puede girar aquí. Intenta mover la pieza un poco antes de girarla., 👽Pista: Puedes apoyarte en el centro entre las figuras 6 y 3 ✅"
//     // );
//     textoAlerta.value =
//       " No se puede girar aquí. Intenta mover la pieza un poco antes de girarla., 👽 Pista: Puedes apoyarte en el centro entre las figuras 6 y 3 ✅";
//     mostrar.value = true;
//     setTimeout(() => (mostrar.value = false), 3000);
//   }
// }
function rotarSeleccion(grados) {
  const pieza = userPieces.value.find((p) => p.id === selectedShapeId.value);
  if (!pieza || !pieza.draggable) return;

  const rotacionAnterior = pieza.rotation;
  const nuevaRotacion = (pieza.rotation + grados) % 360;
  pieza.rotation = nuevaRotacion; // Se aplica temporalmente para que se vea visualmente

  // Validar si la nueva rotación hace que se salga
  const angleRad = nuevaRotacion * (Math.PI / 180);
  const stageWidth = stageSize.value.width;
  const stageHeight = stageSize.value.height;

  let seSale = false;

  for (let i = 0; i < pieza.points.length; i += 2) {
    const px = pieza.points[i];
    const py = pieza.points[i + 1];

    const rotatedX = px * Math.cos(angleRad) - py * Math.sin(angleRad);
    const rotatedY = px * Math.sin(angleRad) + py * Math.cos(angleRad);

    const worldX = rotatedX + pieza.x;
    const worldY = rotatedY + pieza.y;

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
    // 🔁 Revertimos a la rotación anterior después de un pequeño retraso visual
    setTimeout(() => {
      pieza.rotation = rotacionAnterior;
    }, 200); // Pequeña pausa para que se note que rotó y se corrigió
  }
}
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
  max-width: 250px;
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 2rem;
}
.acciones-rotacion button {
  margin: 5px;
}
button {
  padding: 10px 16px;
  margin: 5px;
  font-size: 16px;
  font-weight: bold;
  color: #ffffff;
  background-color: #007bff; /* Azul fuerte para contraste */
  border: none;
  border-radius: 8px;
  cursor: pointer;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.3); /* Asegura legibilidad del texto */
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transition: background-color 0.3s ease;
}
.titulo-acciones-rotacion {
  color: var(--black-36-a-7680, #6a7680);
  text-align: center;

  font-size: 18px;
  font-weight: bold;
  color: #343a40;
  margin-right: 10px;
  padding: 4px 8px;
}

.titulo-acciones-rotacion,
.titulo-acciones-rotacion + span,
.titulo-acciones-rotacion + span + span {
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  vertical-align: middle;
  text-shadow: 1px 1px 1px rgba(255, 255, 255, 0.2);
}

.titulo-acciones-rotacion + span,
.titulo-acciones-rotacion + span + span {
  margin-right: 10px;
  font-size: 16px;
  color: #212529;
}
</style>
