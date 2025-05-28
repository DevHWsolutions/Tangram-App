<template>
  <div class="contenedor-principal">
    <!-- Canvas a la izquierda -->
    <v-stage
      ref="stage"
      :config="stageSize"
      @mousedown="handleStageMouseDown"
      @touchstart="handleStageMouseDown"
    >
      <v-layer ref="layer">
        <!-- Piezas del usuario -->
        <v-shape
          v-for="piece in userPieces"
          :key="piece.id"
          :config="getUserShapeConfig(piece)"
          @dragend="handleDragEnd"
        />

        <v-transformer ref="transformer" :resizeEnabled="false" />
      </v-layer>
    </v-stage>

    <!-- Info a la derecha -->
    <div class="panel-lateral">
      <h3>Posiciones actuales:</h3>
      <ul>
        <li v-for="piece in userPieces" :key="piece.id">
          {{ piece.nombre }} → x: {{ piece.x.toFixed(0) }}, y:
          {{ piece.y.toFixed(0) }}, rotación: {{ piece.rotation.toFixed(0) }}°
          color: {{ piece.fill }}
        </li>
      </ul>
    </div>
    <div v-if="selectedShapeId" class="acciones-rotacion">
      <span class="titulo-acciones-rotacion">Acciones:</span>
      <span>Pieza:</span>
      <span>{{ selectedPieceName }}</span>
      <button @click="rotarSeleccion(45)">Girar 45° Derecha</button>
      <button @click="rotarSeleccion(-45)">Girar 45° Izquierda</button>
    </div>
    <div>
      <input
        v-model="nombreArchivo"
        placeholder="Nombre del archivo"
        class="input"
      />

      <button @click="guardarJSON" class="btn">Guardar JSON</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";
import { triangulo as trianguloData } from "../trianguloABC.js";

const stageSize = ref({ width: 900, height: 700 });
const nombreArchivo = ref("");
const userPieces = ref([
  {
    id: "up1",
    templateId: "p1",
    nombre: "Triángulo Grande 1",
    points: [...trianguloData.find((p) => p.id === "p1").points],
    x: 750,
    y: 220,
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
    y: 90,
    fill: "orange",
    rotation: 45,
    draggable: true,
  },
  {
    id: "up3",
    templateId: "p3",
    nombre: "Triángulo Mediano",
    points: [...trianguloData.find((p) => p.id === "p3").points],
    x: 700,
    y: 340,
    fill: "magenta",
    rotation: 0,
    draggable: true,
  },
  {
    id: "up4",
    templateId: "p4",
    nombre: "Triángulo Pequeño 1",
    points: [...trianguloData.find((p) => p.id === "p4").points],
    x: 700,
    y: 440,
    fill: "lime",
    rotation: 315,
    draggable: true,
  },
  {
    id: "up5",
    templateId: "p5",
    nombre: "Triángulo Pequeño 2",
    points: [...trianguloData.find((p) => p.id === "p5").points],
    x: 700,
    y: 440,
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
    y: 520,
    fill: "blue",
    rotation: 45,
    draggable: true,
  },
  {
    id: "up7",
    templateId: "p7",
    nombre: "Paralelogramo",
    points: [...trianguloData.find((p) => p.id === "p7").points],
    x: 700,
    y: 520,
    fill: "teal",
    rotation: 180,
    draggable: true,
  },
]);
// const selectedShapeId = ref("");
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
});

const handleDragEnd = (e) => {
  const id = e.target.id();
  const piece = userPieces.value.find((p) => p.id === id);
  if (piece) {
    piece.x = e.target.x();
    piece.y = e.target.y();
    piece.rotation = e.target.rotation();
    console.log(
      `Pieza ${piece.nombre} movida → x: ${piece.x}, y: ${piece.y}, rotación: ${piece.rotation}`
    );
  }
};

// Control de selección y transformer
const selectedShapeId = ref("");
const transformer = ref(null);

const updateTransformer = () => {
  const transformerNode = transformer.value.getNode();
  const stage = transformerNode.getStage();

  const selectedNode = stage.findOne("#" + selectedShapeId.value);
  const isUserPiece = userPieces.value.some(
    (p) => p.id === selectedShapeId.value
  );

  if (selectedNode && isUserPiece) {
    transformerNode.nodes([selectedNode]);
  } else {
    transformerNode.nodes([]);
  }
};

const handleStageMouseDown = (e) => {
  if (e.target === e.target.getStage()) {
    selectedShapeId.value = "";
    updateTransformer();
    return;
  }

  const clickedOnTransformer =
    e.target.getParent()?.className === "Transformer";
  if (clickedOnTransformer) return;

  const id = e.target.id();
  const piece = userPieces.value.find((p) => p.id === id);

  selectedShapeId.value = piece ? id : "";
  updateTransformer();
};

function limpiarNumero(n) {
  return Number.isInteger(n) ? n : parseFloat(n.toFixed(2));
}

function guardarJSON() {
  if (!nombreArchivo.value.trim()) {
    alert("Por favor, escribe un nombre para el archivo.");
    return;
  }

  const limpiarNumero = (num) => Math.round(num); // redondea a entero

  const datos = userPieces.value.map((piece) => ({
    id: piece.id,
    nombre: piece.nombre,
    points: piece.points.map(limpiarNumero),
    x: limpiarNumero(piece.x),
    y: limpiarNumero(piece.y),
    fill: piece.fill,
    rotation: limpiarNumero(piece.rotation),
    draggable: true,
  }));

  // Generar el contenido manualmente como archivo .js sin comillas en las claves
  const contenido =
    `export const ${nombreArchivo.value} = [\n` +
    datos
      .map((piece) => {
        return `  {
    id: "${piece.id}",
    nombre: "${piece.nombre}",
    points: [${piece.points.join(", ")}],
    x: ${piece.x},
    y: ${piece.y},
    fill: "${piece.fill}",
    rotation: ${piece.rotation},
    draggable: ${piece.draggable}
  }`;
      })
      .join(",\n") +
    `\n];\n`;

  const blob = new Blob([contenido], { type: "text/javascript" });
  const url = URL.createObjectURL(blob);

  const enlace = document.createElement("a");
  enlace.href = url;
  enlace.download = `${nombreArchivo.value}.js`;
  enlace.click();

  URL.revokeObjectURL(url);
}
const selectedPieceName = computed(() => {
  const pieza = userPieces.value.find((p) => p.id === selectedShapeId.value);
  return pieza ? pieza.nombre : null;
});
function rotarSeleccion(grados) {
  const pieza = userPieces.value.find((p) => p.id === selectedShapeId.value);
  if (!pieza || !pieza.draggable) return;

  const nuevaRotacion = (pieza.rotation + grados) % 360;
  const angleRad = nuevaRotacion * (Math.PI / 180);

  const stageWidth = stageSize.value.width;
  const stageHeight = stageSize.value.height;

  let seSale = false;

  for (let i = 0; i < pieza.points.length; i += 2) {
    const px = pieza.points[i];
    const py = pieza.points[i + 1];

    // aplicar rotación a cada punto
    const rotatedX = px * Math.cos(angleRad) - py * Math.sin(angleRad);
    const rotatedY = px * Math.sin(angleRad) + py * Math.cos(angleRad);

    // trasladar al mundo real (posición actual de la pieza)
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

  if (!seSale) {
    pieza.rotation = nuevaRotacion;
  } else {
    // console.log("❌ No se puede rotar: la pieza se saldría del canvas.");
    alert(
      "⚠️ No se puede girar aquí. Intenta mover la pieza un poco antes de girarla., 👽Pista: Puedes apoyarte en el centro entre las figuras 6 y 3 ✅"
    );
  }
}
</script>

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

.v-stage {
  outline: 2px solid black;
  border: 1px solid #ccc;
}

p {
  padding-top: 5px;
}
.input {
  margin: 10px;
  padding: 8px;
  border: 1px solid #ccc;
}
.btn {
  padding: 8px 16px;
  background-color: #2c3e50;
  color: white;
  border: none;
  cursor: pointer;
}
.btn:hover {
  background-color: #1a242f;
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
