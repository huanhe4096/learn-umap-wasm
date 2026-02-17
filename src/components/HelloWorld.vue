<script setup>
import { ref, onMounted } from 'vue'
import { createUMAP } from "embedding-atlas";

const canvasRef = ref(null)

let input_dim = 128;
let n = 5000;
let data = new Float32Array(n * input_dim)

// Fill the data with random values, range [-100, 100]
for (let i = 0; i < n * input_dim; i++) {
  data[i] = Math.random() * 200 - 100;
}

let options = {
  metric: "cosine",
}

let embedding = null;

async function computeEmbedding() {
  console.time('UMAP')
  // create UMAP and compute embedding, then draw
  const umap = await createUMAP(n, input_dim, 2, data, options)
  console.log('UMAP created', umap)
  umap.run()
  console.timeEnd('UMAP')
  embedding = umap.embedding()
}

defineProps({
  msg: String,
})

const count = ref(0)

function drawEmbedding() {
  const canvas = canvasRef.value
  if (!canvas || !embedding) return
  const ctx = canvas.getContext('2d')
  const w = canvas.width
  const h = canvas.height
  ctx.clearRect(0, 0, w, h)

  // Find bounds
  let minX = Infinity, maxX = -Infinity, minY = Infinity, maxY = -Infinity
  for (let i = 0; i < n; i++) {
    const x = embedding[2 * i]
    const y = embedding[2 * i + 1]
    if (x < minX) minX = x
    if (x > maxX) maxX = x
    if (y < minY) minY = y
    if (y > maxY) maxY = y
  }

  const pad = 20
  const rangeX = maxX - minX || 1
  const rangeY = maxY - minY || 1

  ctx.fillStyle = 'rgba(10,120,200,0.8)'
  const radius = 1.5

  for (let i = 0; i < n; i++) {
    const x = embedding[2 * i]
    const y = embedding[2 * i + 1]
    const sx = pad + ((x - minX) / rangeX) * (w - pad * 2)
    const sy = pad + ((y - minY) / rangeY) * (h - pad * 2)
    ctx.beginPath()
    ctx.arc(sx, h - sy, radius, 0, Math.PI * 2)
    ctx.fill()
  }
}

onMounted(() => {

  // set canvas pixel size for crisp rendering
  const canvas = canvasRef.value
  if (canvas) {
    // desired display size
    const displayW = 800
    const displayH = 600
    canvas.style.width = displayW + 'px'
    canvas.style.height = displayH + 'px'
    const ratio = window.devicePixelRatio || 1
    canvas.width = displayW * ratio
    canvas.height = displayH * ratio
    // scale context so drawing uses CSS pixels
    const ctx = canvas.getContext('2d')
    ctx.setTransform(ratio, 0, 0, ratio, 0, 0)
  }

  computeEmbedding().then(() => {
    drawEmbedding()
  })
})
</script>

<template>
  <h1>{{ msg }}</h1>


</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
