<template>
    <div class="canvas-container">
        <canvas ref="canvas" @mousedown="startDrawing" @mousemove="draw" @mouseup="stopDrawing" @mouseleave="stopDrawing"></canvas>
    </div>
    <div class="input-containers">
        <div class="slider-group">
            <label for="brushSize">Brush Size: {{ brushSize }}</label>
            <input
                id="brushSize"
                type="range"
                min="1"
                max="6"
                v-model="brushSize"
            />
        </div>
        <div class="btn-group">
            <button 
                @click="mode = 'addSand'"
                :class="mode === 'addSand' ? 'active-button sand' : 'inactive-button'"
            >
                Sand
            </button>
            <button 
                @click="mode = 'addWater'"
                :class="mode === 'addWater' ? 'active-button water' : 'inactive-button'"
            >
                Water
            </button>
            <button 
                @click="mode = 'addRock'"
                :class="mode === 'addRock' ? 'active-button rock' : 'inactive-button'"
            >
                Rock
            </button>
            <button 
                @click="mode = 'delete'"
                :class="mode === 'delete' ? 'active-button del' : 'inactive-button'"
            >
                Delete
            </button>
        </div>
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
      const waterParticles = ref([]);
      const rockParticles = ref([]); // New rock particles array


      
      const canvasWidth = 800;
      const canvasHeight = 600;
      const cellSize = 2;
      const cols = canvasWidth / cellSize;
      const rows = canvasHeight / cellSize;
      const grid = Array.from({ length: cols }, () => Array(rows).fill(0));

      const brushSize = ref(1);
      const mode = ref('addSand');
      const sandColor = '#FFD700'; // Static sand color (golden)
      const waterColor = '#00BFFF'; // Water color (deepskyblue)
      const rockColor = "#808080";
      const rockClumps = ref([]);

      
     
      const cursorPosition = ref({ x: 0, y: 0 });
      const showCursor = ref(false);

  
      const startDrawing = (event) => {
        isDrawing.value = true;
        modifyParticles(event);
      };
  
      const draw = (event) => {
        const rect = canvas.value.getBoundingClientRect();
        cursorPosition.value = { x: event.clientX - rect.left, y: event.clientY - rect.top };
        showCursor.value = true;
        if (!isDrawing.value) return;
        modifyParticles(event);
      };
  
      const stopDrawing = () => {
        isDrawing.value = false;
        showCursor.value = false;

      };
  
      const modifyParticles = (event) => {
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
                if (mode.value === 'addSand' && grid[newCol][newRow] === 0) {
                    grid[newCol][newRow] = 1;
                    sandParticles.value.push({ col: newCol, row: newRow });
                } else if (mode.value === 'addWater' && grid[newCol][newRow] === 0) {
                    grid[newCol][newRow] = 2;
                    waterParticles.value.push({ col: newCol, row: newRow, settleTime: 0 });
                } else if (mode.value === 'delete' && grid[newCol][newRow] !== 0) {
                    grid[newCol][newRow] = 0;
                    sandParticles.value = sandParticles.value.filter(p => p.col !== newCol || p.row !== newRow);
                    waterParticles.value = waterParticles.value.filter(p => p.col !== newCol || p.row !== newRow);
                }
                else if (mode.value === 'addRock'){
                    const brushRadius = Math.floor(brushSize.value / 2);
                    const newClump = [];

                    for (let x = -brushRadius; x <= brushRadius; x++) {
                        for (let y = -brushRadius; y <= brushRadius; y++) {
                            const dist = Math.sqrt(x * x + y * y);
                            if (dist <= brushRadius) {
                                const newCol = col + x;
                                const newRow = row + y;
                            if (newCol >= 0 && newCol < cols && newRow >= 0 && newRow < rows && grid[newCol][newRow] === 0) {
                                grid[newCol][newRow] = 3; // Rock particle type
                                newClump.push({ col: newCol, row: newRow });
                            }
                            }
                        }
                    }
                    if (newClump.length > 0) {
                        rockClumps.value.push(newClump);
                    }
                }
              }
            }
          }
        }
      };
  
      const updateCanvas = () => {
  if (!context.value) return;

  context.value.clearRect(0, 0, canvasWidth, canvasHeight);

  // Draw and update sand particles
  context.value.fillStyle = sandColor;
  const newSandParticles = [];

  for (let i = 0; i < sandParticles.value.length; i++) {
    let particle = sandParticles.value[i];
    let { col, row } = particle;

    if (row < rows - 1) {
      if (grid[col][row + 1] === 0) {
        // Move down
        grid[col][row] = 0;
        grid[col][row + 1] = 1;
        row++;
      } else if (grid[col][row + 1] === 2) {
        // Swap with water
        grid[col][row] = 2;
        grid[col][row + 1] = 1;
        row++;
        // Update the water particle position
        for (let j = 0; j < waterParticles.value.length; j++) {
          if (waterParticles.value[j].col === col && waterParticles.value[j].row === row) {
            waterParticles.value[j].row--;
            break;
          }
        }
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
    }

    newSandParticles.push({ col, row });
    context.value.fillRect(col * cellSize, row * cellSize, cellSize, cellSize);
  }

  sandParticles.value = newSandParticles;

  // Update water particles
  context.value.fillStyle = waterColor;
  const newWaterParticles = [];

  for (let i = 0; i < waterParticles.value.length; i++) {
    let particle = waterParticles.value[i];
    let { col, row, settleTime } = particle;

    if (row < rows - 1 && grid[col][row + 1] === 0) {
      // Move down
      grid[col][row] = 0;
      grid[col][row + 1] = 2;
      row++;
      settleTime = 0; // Reset settle time when moved
    } else if (col > 0 && grid[col - 1][row] === 0) {
      // Move left
      grid[col][row] = 0;
      grid[col - 1][row] = 2;
      col--;
      settleTime = 0; // Reset settle time when moved
    } else if (col < cols - 1 && grid[col + 1][row] === 0) {
      // Move right
      grid[col][row] = 0;
      grid[col + 1][row] = 2;
      col++;
      settleTime = 0; // Reset settle time when moved
    } else if (settleTime < 60) { // Check if settle time is less than 1 second (assuming 60 frames per second)
      settleTime++; // Increment settle time if not moved
    }

    newWaterParticles.push({ col, row, settleTime });
    context.value.fillRect(col * cellSize, row * cellSize, cellSize, cellSize);
  }

  waterParticles.value = newWaterParticles;

  // Update rock clumps
  context.value.fillStyle = rockColor;
  const newRockClumps = [];

  for (let clump of rockClumps.value) {
    let canFall = true;

    for (let particle of clump) {
      let { col, row } = particle;
      if (row >= rows - 1 || grid[col][row + 1] !== 0) {
        canFall = false;
        break;
      }
    }

    // Move the clump down if possible
    if (canFall) {
      for (let particle of clump) {
        let { col, row } = particle;
        grid[col][row] = 0; // Clear the current position
      }
      for (let particle of clump) {
        particle.row++;
        let { col, row } = particle;
        grid[col][row] = 3; // Set the new position
      }
    }

    newRockClumps.push(clump);

    // Draw the clump
    for (let particle of clump) {
      let { col, row } = particle;
      context.value.fillRect(col * cellSize, row * cellSize, cellSize, cellSize);
    }
  }

  rockClumps.value = newRockClumps;

  // Draw the cursor indicator
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
  
  <style scoped>
  canvas {
    border: 1px solid black;
    background-color: rgb(65, 59, 59);
    cursor: none;
  }
  .canvas-container{
    border-radius: 1rem;
    padding: 2rem;
    background: rgb(103, 97, 97);
  }
  button{
    border-radius: .25rem;
    border: none;
    cursor: pointer;
    background-color: rgb(165, 85, 42);
    padding: .5rem;
    color: white;
  }
  button:hover{
    background-color: rgb(144, 206, 251);
    color: black;
  }
  .active-button{
    color: black;
    background: rgb(144, 206, 251);
    flex-grow: 2;
    transition: all .75s;
  }
  .active-button.sand{
    background-color: #FFD700;

  }
  .active-button.water{
    background-color: #00BFFF;
  }
  .active-button.rock{
    background-color: #808080;
    color: white
  }
  .active-button.del{
    background-color: rgb(234, 36, 33);
    color: white;
  }
  .input-containers{
    display: flex;
  }
  .btn-group{
    display: flex;
    justify-content: space-between;
    padding: 1rem;
    width: 20rem;
  }
  .slider-group{
    display: flex;
    color: white;
    width: 20rem;
    justify-content: space-between;
    align-items: center;
  }
  </style>
  