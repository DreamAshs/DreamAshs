<script setup lang="ts">
import { onMounted, ref } from 'vue'


class Cell {
  x: number
  y: number
  size: number
  acc: [number, number]
  vel: [number, number]
  marker: [number, number]
  center: [number, number]
  width: number
  height: number
  minSize: number
  maxSize: number
  // hue: number

  constructor(center: [number, number], width: number, height: number, minSize: number, maxSize: number) {
    const a = Math.random() * 2 * Math.PI
    const x = Math.cos(a) * width
    const y = Math.sin(a) * height
    const s = Math.random()
    const markerX = 0 
    const markerY = - height * 3

    this.center = center
    this.width = width
    this.height = height
    this.minSize = minSize
    this.maxSize = maxSize
    this.x = x * s
    this.y = y * s
    this.size = minSize + (maxSize - minSize) * Math.random()
    this.vel = [1, 1]
    this.acc = [0, 0]
    this.marker = [markerX, markerY]
    // this.hue = 225 * Math.random();
  }

  reachedMarker(): boolean {
    return this.y <= this.marker[1] && this.marker[0] - 20 <= this.x
  }

  get normX(): number {
    return this.center[0] + this.x
  }

  get normY(): number {
    return this.center[1] + this.y
  }

  update() {
    if (this.reachedMarker()) {
      this.acc = [0, 0];
    } else {
      const dirX = this.marker[0] - this.x;
      const dirY = this.marker[1] - this.y;
      const len = Math.sqrt(Math.pow(dirX, 2) + Math.pow(dirY, 2))
      const dir: [number, number] = [dirX / len, dirY / len]

      this.acc = dir
    }

    const dt = 0.2

    this.vel[0] += this.acc[0] * dt
    this.vel[1] += this.acc[1] * dt

    this.x += this.vel[0] * dt
    this.y += this.vel[1] * dt
  }

  drawMarker(ctx: CanvasRenderingContext2D) {
    const x = this.center[0] + this.marker[0]
    const y = this.center[1] + this.marker[1]

    const cx = x - this.size / 2;
    const cy = y - this.size / 2;

    ctx.fillStyle = "red"
    ctx.fillRect(cx, cy, this.size, this.size);
  }

  draw(ctx: CanvasRenderingContext2D) {
    const x = this.center[0] + this.x
    const y = this.center[1] + this.y

    const cx = x - this.size / 2;
    const cy = y - this.size / 2;

    ctx.fillStyle = "white"
    ctx.fillRect(cx, cy, this.size, this.size);

    // this.drawMarker(ctx)
  }
}

const flame = ref<HTMLCanvasElement>()

class FlameRenderer {
  ctx: CanvasRenderingContext2D
  cells: Cell[]
  maxCellCount: number
  newCell: () => Cell

  constructor(ctx: CanvasRenderingContext2D, maxCellCount: number, newCell: () => Cell) {
    this.ctx = ctx
    this.cells = []
    this.maxCellCount = maxCellCount
    this.newCell = newCell
  }

  update() {
    if (this.cells.length < this.maxCellCount) {
      const cell = this.newCell()
      this.cells.push(cell)
    }

    for (let i = 0; i < this.cells.length; i++) {
      const cell = this.cells[i]

      cell.update()

      const x = cell.normX
      const y = cell.normY

      if (this.ctx.canvas.width <= x || this.ctx.canvas.height <= y || x < 0 || y < 0) {
        this.cells.splice(i, 1);

        const cell = this.newCell()
        this.cells.push(cell)
      }
    }
  }

  draw() {
    this.ctx.reset()

    for (const cell of this.cells) {
      cell.draw(this.ctx)
    }
  }
}

onMounted(() => {
  const w = 500;
  const h = 500;

  flame.value!.width = w;
  flame.value!.height = h;

  const ctx = flame.value!.getContext('2d')

  if (ctx == null) {
    throw new Error("Failed to get canvas ctx")
  }

  function flaming(renderer: FlameRenderer) {
    renderer.draw()
    renderer.update()

    requestAnimationFrame(() => flaming(renderer))
  }

  const newCell = () => new Cell([w / 2, h / 4 * 3], w / 8, h / 10, 5, 20)
  const renderer = new FlameRenderer(ctx, 100, newCell)
  flaming(renderer)
})
</script>

<template>
  <div id="container">
    <canvas id="flame" ref="flame"></canvas>

    <h1 id="heading">DREAM ASHS</h1>

    <div id="scroll-down-box">
      <svg width="30px" viewBox="0 0 120 120" id="scroll-down">
        <polyline fill="none" stroke="white" stroke-width="20" points="0,0 60,60 120,0" class="arrow" />
      </svg>
      <div>dream for more</div>
    </div>
  </div>
</template>

<style scoped>
#container {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: black;
  flex-direction: column;
}

/* p { */
/*   color: white; */
/*   font-family: "VictorMono NFM"; */
/* } */

#heading {
  font-family: "VictorMono NFM";
  font-size: 5rem;
  color: white;
}

#scroll-down-box {
  position: absolute;
  bottom: 5%;
  color: white;
  display: flex;
  flex-direction: column;
  place-items: center;
}

#scroll-down-box * {
  color: white;
  text-transform: uppercase;
  font-family: "VictorMono NFM";
}

@keyframes blink {
  0% {
    stroke: white;
  }
  80% {
    stroke: #222;
  }
  100% {
    stroke: white;
  }
}

.arrow {
  animation: blink 3s infinite;
}
</style>
