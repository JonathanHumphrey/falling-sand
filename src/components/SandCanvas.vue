<template>
    <canvas ref="canvas" @mousedown="startDrawing" @mousemove="draw" @mouseup="stopDrawing" @mouseleave="stopDrawing"></canvas>
    <div>
      <label for="brushSize">Brush Size: {{ brushSize }}</label>
      <input
        id="brushSize"
        type="range"
        min="1"
        max="5"
        v-model="brushSize"
      />
    </div>
    <div>
      <button @click="mode = 'add'">Add Mode</button>
      <button @click="mode = 'delete'">Delete Mode</button>
    </div>
  </template>
  
  <script>
  import { ref, onMounted } from 'vue';
  
  export default {
    setup() {
      const canvas = ref(null);
      const context = ref(null);
      const isDrawing = ref(false);
      const sandParticles = ref([]);
      
      const canvasWidth = 800;
      const canvasHeight = 600;
      const cellSize = 2;
      const cols = canvasWidth / cellSize;
      const rows = canvasHeight / cellSize;
      const grid = Array.from({ length: cols }, () => Array(rows).fill(0));

      const brushSize = ref(1);
      const sandColor ='#FFD700'
      const mode = ref('add');
      const cursorPosition = ref({ x: 0, y: 0 });
      const showCursor = ref(false);

  
      const startDrawing = (event) => {
        isDrawing.value = true;
        modifySand(event);
      };
  
      const draw = (event) => {
        const rect = canvas.value.getBoundingClientRect();
        cursorPosition.value = { x: event.clientX - rect.left, y: event.clientY - rect.top };
        showCursor.value = true;
        if (!isDrawing.value) return;
        modifySand(event);
      };
  
      const stopDrawing = () => {
        isDrawing.value = false;
        showCursor.value = false;

      };
  
      const modifySand = (event) => {
        const rect = canvas.value.getBoundingClientRect();
        const x = event.clientX - rect.left;
        const y = event.clientY - rect.top;

        const col = Math.floor(x / cellSize);
        const row = Math.floor(y / cellSize);

        const radius = brushSize.value;

      for (let i = -radius; i <= radius; i++) {
        for (let j = -radius; j <= radius; j++) {
          const distance = Math.sqrt(i * i + j * j);
          if (distance <= radius) {
            const newCol = col + i;
            const newRow = row + j;
            if (newCol >= 0 && newCol < cols && newRow >= 0 && newRow < rows) {
              if (mode.value === 'add' && grid[newCol][newRow] === 0) {
                grid[newCol][newRow] = 1;
                sandParticles.value.push({ col: newCol, row: newRow });
              } else if (mode.value === 'delete' && grid[newCol][newRow] === 1) {
                grid[newCol][newRow] = 0;
                sandParticles.value = sandParticles.value.filter(p => p.col !== newCol || p.row !== newRow);
              }
            }
          }
        }
      }
    };
  
      const updateCanvas = () => {
        if (!context.value) return;

      context.value.clearRect(0, 0, canvasWidth, canvasHeight);
      context.value.fillStyle = sandColor; // Set the fill style to the static sand color

      const newSandParticles = [];

      for (let i = 0; i < sandParticles.value.length; i++) {
        let particle = sandParticles.value[i];
        let { col, row } = particle;

        if (row < rows - 1 && grid[col][row + 1] === 0) {
          // Move down
          grid[col][row] = 0;
          grid[col][row + 1] = 1;
          row++;
        } else if (col > 0 && row < rows - 1 && grid[col - 1][row + 1] === 0) {
          // Move diagonally left
          grid[col][row] = 0;
          grid[col - 1][row + 1] = 1;
          col--;
          row++;
        } else if (col < cols - 1 && row < rows - 1 && grid[col + 1][row + 1] === 0) {
          // Move diagonally right
          grid[col][row] = 0;
          grid[col + 1][row + 1] = 1;
          col++;
          row++;
        }

        newSandParticles.push({ col, row });
        context.value.fillRect(col * cellSize, row * cellSize, cellSize, cellSize);
      }

      sandParticles.value = newSandParticles;

      if (showCursor.value) {
        context.value.strokeStyle = 'red';
        context.value.beginPath();
        context.value.arc(cursorPosition.value.x, cursorPosition.value.y, brushSize.value * cellSize, 0, Math.PI * 2);
        context.value.stroke();
      }

      requestAnimationFrame(updateCanvas);
      };
  
      onMounted(() => {
        const canvasEl = canvas.value;
        canvasEl.width = canvasWidth;
        canvasEl.height = canvasHeight;
        context.value = canvasEl.getContext("2d");
        updateCanvas();
      });
  
      return {
        canvas,
        startDrawing,
        draw,
        stopDrawing,
        brushSize,
        mode,
        cursorPosition,
        showCursor
      };
    }
  };
  </script>
  
  <style>
  canvas {
    border: 1px solid black;
    background-color: rgb(65, 59, 59);
    cursor: none;
  }
  </style>
  