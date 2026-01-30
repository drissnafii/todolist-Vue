<script setup lang="ts">
import { ref, onMounted, onUnmounted, nextTick, watch, type ComponentPublicInstance } from 'vue'

// Matter.js types - external library with dynamic types
declare global {
  interface Window {
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    Matter: any
  }
}

// Task interface with optional position data
interface Task {
  id: string
  text: string
  color: string
  textColor: string
  // Position data (saved when navigating away)
  x?: number
  y?: number
  angle?: number
}

// Pastel color palette for random selection
const PASTEL_COLORS = [
  { bg: '#d2f18a', text: '#4a8c00' }, // green
  { bg: '#f0ee8a', text: '#9c7b00' }, // yellow
  { bg: '#e9d6ff', text: '#7e3bff' }, // purple
  { bg: '#ffd1d2', text: '#d13438' }, // red
  { bg: '#c7e2ff', text: '#0078d4' }, // blue
  { bg: '#ffe0c9', text: '#ca5010' }, // orange
  { bg: '#c9f5e9', text: '#0b8a70' }, // mint
  { bg: '#ffd6ef', text: '#c2185b' }, // pink
  { bg: '#c5f0f0', text: '#038387' }, // teal
]

const STORAGE_KEY = 'physics-tasks'

// Default tasks
const DEFAULT_TASKS: Task[] = [
  { id: '1', text: 'BUBBLE.IO', color: '#d2f18a', textColor: '#4a8c00' },
  { id: '2', text: 'NOCODE', color: '#f0ee8a', textColor: '#9c7b00' },
  { id: '3', text: 'EXPRESSION', color: '#e9d6ff', textColor: '#7e3bff' },
  { id: '4', text: 'MOBILE', color: '#c9f5e9', textColor: '#0b8a70' },
  { id: '5', text: 'UX/UI', color: '#ffd6ef', textColor: '#c2185b' },
  { id: '6', text: 'API CONNECTOR', color: '#e9d6ff', textColor: '#7e3bff' },
  { id: '7', text: 'DESIGN', color: '#c5f0f0', textColor: '#038387' },
]

// Reactive state
const tasks = ref<Task[]>([])
const newTaskText = ref('')
const containerRef = ref<HTMLDivElement | null>(null)
const canvasRef = ref<HTMLCanvasElement | null>(null)
const tagRefs = ref<HTMLDivElement[]>([])
const isLoaded = ref(false)
const initializedTaskIds = ref<Set<string>>(new Set()) // Track tasks with physics bodies

// eslint-disable-next-line @typescript-eslint/no-explicit-any
let engine: any = null
// eslint-disable-next-line @typescript-eslint/no-explicit-any
let render: any = null
// eslint-disable-next-line @typescript-eslint/no-explicit-any
let world: any = null
// eslint-disable-next-line @typescript-eslint/no-explicit-any
let tagBodies: { id: string; body: any; element: HTMLDivElement }[] = []

// localStorage functions
const loadTasks = (): Task[] => {
  try {
    const saved = localStorage.getItem(STORAGE_KEY)
    if (saved) {
      return JSON.parse(saved)
    }
  } catch (e) {
    console.error('Failed to load tasks:', e)
  }
  return DEFAULT_TASKS
}

const saveTasks = () => {
  try {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(tasks.value))
  } catch (e) {
    console.error('Failed to save tasks:', e)
  }
}

// Save current positions to task data
const savePositions = () => {
  tagBodies.forEach(({ id, body }) => {
    const task = tasks.value.find((t) => t.id === id)
    if (task && body) {
      task.x = body.position.x
      task.y = body.position.y
      task.angle = body.angle
    }
  })
  saveTasks()
}

// Generate random pastel color
const getRandomColor = () => {
  const index = Math.floor(Math.random() * PASTEL_COLORS.length)

  return PASTEL_COLORS[index]!
}

// Generate unique ID
const generateId = () => {
  return Date.now().toString(36) + Math.random().toString(36).substr(2)
}

const setTagRef = (el: Element | ComponentPublicInstance | null, index: number) => {
  if (el && el instanceof HTMLDivElement) {
    tagRefs.value[index] = el
  }
}

// Add new task
const addTask = async () => {
  const text = newTaskText.value.trim()
  if (!text) return

  const color = getRandomColor()
  const newTask: Task = {
    id: generateId(),
    text,
    color: color.bg,
    textColor: color.text,
  }

  tasks.value.push(newTask)
  newTaskText.value = ''
  saveTasks()

  // Wait for DOM to update, then add physics body
  await nextTick()
  setTimeout(() => addPhysicsBody(newTask, tasks.value.length - 1), 10)
}

// Add physics body for a single task
const addPhysicsBody = (task: Task, index: number) => {
  if (!engine || !containerRef.value) return

  const Matter = window.Matter
  const Bodies = Matter.Bodies
  const World = Matter.World

  const element = tagRefs.value[index]
  if (!element) return

  const containerWidth = containerRef.value.clientWidth
  const rect = element.getBoundingClientRect()
  const width = rect.width
  const height = rect.height

  // Drop from top with random x position
  const x = Math.random() * (containerWidth - width) + width / 2
  const y = -height - 50

  const body = Bodies.rectangle(x, y, width, height, {
    chamfer: { radius: height / 2 },
    density: 0.001,
    friction: 0.3,
    frictionAir: 0.01,
    restitution: 0.6,
    render: { fillStyle: 'transparent' },
  })

  World.add(world, body)
  tagBodies.push({ id: task.id, body, element })

  // Mark as initialized so it becomes visible
  initializedTaskIds.value.add(task.id)
}

// Organize tasks in a grid
const organizeTasks = () => {
  if (!engine || !containerRef.value || tagBodies.length === 0) return

  const Matter = window.Matter
  const Body = Matter.Body

  const containerWidth = containerRef.value.clientWidth
  const containerHeight = containerRef.value.clientHeight

  const tagsPerRow = 4
  const rowHeight = 55

  tagBodies.forEach((item, idx) => {
    const row = Math.floor(idx / tagsPerRow)
    const col = idx % tagsPerRow

    const xSpacing = containerWidth / (tagsPerRow + 1)
    const targetX = xSpacing * (col + 1)
    const targetY = containerHeight - 40 - row * rowHeight

    // Set position and reset velocity/angle
    Body.setPosition(item.body, { x: targetX, y: targetY })
    Body.setVelocity(item.body, { x: 0, y: 0 })
    Body.setAngle(item.body, 0)
    Body.setAngularVelocity(item.body, 0)
  })
}

// Initialize physics
const initPhysics = () => {
  if (!containerRef.value || !canvasRef.value) return

  const Matter = window.Matter
  const Engine = Matter.Engine
  const Render = Matter.Render
  const World = Matter.World
  const Bodies = Matter.Bodies
  const MouseConstraint = Matter.MouseConstraint
  const Mouse = Matter.Mouse
  const Events = Matter.Events

  const container = containerRef.value
  const containerWidth = container.clientWidth
  const containerHeight = container.clientHeight

  // Create engine
  engine = Engine.create()
  world = engine.world
  engine.world.gravity.y = 0.8

  // Create renderer
  render = Render.create({
    element: container,
    canvas: canvasRef.value,
    engine: engine,
    options: {
      width: containerWidth,
      height: containerHeight,
      background: 'transparent',
      wireframes: false,
    },
  })

  // Create boundaries
  const thickness = 50
  const ground = Bodies.rectangle(
    containerWidth / 2,
    containerHeight + thickness / 2,
    containerWidth + thickness * 2,
    thickness,
    { isStatic: true, render: { fillStyle: 'transparent' } },
  )

  const wallLeft = Bodies.rectangle(
    -thickness / 2,
    containerHeight / 2,
    thickness,
    containerHeight + thickness * 2,
    { isStatic: true, render: { fillStyle: 'transparent' } },
  )

  const wallRight = Bodies.rectangle(
    containerWidth + thickness / 2,
    containerHeight / 2,
    thickness,
    containerHeight + thickness * 2,
    { isStatic: true, render: { fillStyle: 'transparent' } },
  )

  World.add(world, [ground, wallLeft, wallRight])

  // Create bodies for existing tasks
  tagBodies = []
  tagRefs.value.forEach((element, idx) => {
    if (!element) return

    const task = tasks.value[idx]
    if (!task) return

    const rect = element.getBoundingClientRect()
    const width = rect.width
    const height = rect.height

    // Use saved position if available, otherwise calculate initial position
    let x: number, y: number, angle: number

    if (task.x !== undefined && task.y !== undefined) {
      // Restore saved position
      x = task.x
      y = task.y
      angle = task.angle ?? 0
    } else {
      // Calculate initial position at bottom
      const tagsPerRow = 4
      const row = Math.floor(idx / tagsPerRow)
      const col = idx % tagsPerRow
      const xSpacing = containerWidth / (tagsPerRow + 1)
      x = xSpacing * (col + 1) + (Math.random() - 0.5) * 40
      y = containerHeight - height / 2 - 20 - row * 55 - Math.random() * 20
      angle = (Math.random() - 0.5) * 0.4
    }

    const body = Bodies.rectangle(x, y, width, height, {
      chamfer: { radius: height / 2 },
      density: 0.001,
      friction: 0.3,
      frictionAir: 0.01,
      restitution: 0.6,
      angle,
      render: { fillStyle: 'transparent' },
    })

    World.add(world, body)
    tagBodies.push({ id: task.id, body, element })

    // Mark existing tasks as initialized
    initializedTaskIds.value.add(task.id)
  })

  // Sync positions
  const updatePositions = () => {
    tagBodies.forEach(({ body, element }) => {
      const { x, y } = body.position
      const angle = body.angle
      element.style.transform = `translate(-50%, -50%) translate(${x}px, ${y}px) rotate(${angle}rad)`
      element.style.left = '0px'
      element.style.top = '0px'
    })
  }

  Events.on(engine, 'afterUpdate', updatePositions)

  // Mouse constraint
  const mouse = Mouse.create(render.canvas)
  Mouse.setOffset(mouse, { x: 0, y: 0 })

  const mouseConstraint = MouseConstraint.create(engine, {
    mouse,
    constraint: { stiffness: 0.2, render: { visible: false } },
  })

  World.add(world, mouseConstraint)
  render.mouse = mouse

  Render.run(render)
  Engine.run(engine)
}

// Load Matter.js
const loadMatterJS = () => {
  return new Promise<void>((resolve) => {
    if (window.Matter) {
      resolve()
      return
    }
    const script = document.createElement('script')
    script.src = 'https://cdnjs.cloudflare.com/ajax/libs/matter-js/0.19.0/matter.min.js'
    script.onload = () => resolve()
    document.head.appendChild(script)
  })
}

onMounted(async () => {
  // Load tasks from localStorage
  tasks.value = loadTasks()

  await loadMatterJS()
  isLoaded.value = true
  await nextTick()
  // Small delay for DOM to be ready, then init physics immediately
  setTimeout(initPhysics, 10)
})

onUnmounted(() => {
  // Save positions before unmounting
  savePositions()

  if (render) {
    const Matter = window.Matter
    Matter.Render.stop(render)
    Matter.Engine.clear(engine)
    Matter.World.clear(world, false)
  }
})

// Watch for task changes
watch(tasks, saveTasks, { deep: true })
</script>

<template>
  <div class="tag-canvas-wrapper">
    <!-- Toolbar -->
    <div class="toolbar">
      <input
        v-model="newTaskText"
        type="text"
        placeholder="New task name..."
        class="task-input"
        @keyup.enter="addTask"
      />
      <button class="btn btn-add" @click="addTask">Add Task</button>
      <button class="btn btn-organize" @click="organizeTasks">Organize</button>
    </div>

    <div class="outer-frame">
      <div class="inner-frame">
        <div ref="containerRef" class="tag-canvas">
          <canvas ref="canvasRef" class="physics-canvas"></canvas>

          <div
            v-for="(task, index) in tasks"
            :key="task.id"
            :ref="(el) => setTagRef(el, index)"
            class="tag"
            :class="{ 'tag--hidden': !initializedTaskIds.has(task.id) }"
            :style="{
              backgroundColor: task.color,
              color: task.textColor,
            }"
          >
            {{ task.text }}
          </div>

          <div v-if="!isLoaded" class="loading">Loading physics...</div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Toolbar */
.toolbar {
  display: flex;
  gap: 12px;
  margin-bottom: 16px;
  flex-wrap: wrap;
}

.task-input {
  flex: 1;
  min-width: 180px;
  padding: 12px 16px;
  border: 2px solid #e0e0e0;
  border-radius: 12px;
  font-size: 14px;
  font-weight: 500;
  background: white;
  transition:
    border-color 0.2s,
    box-shadow 0.2s;
}

.task-input:focus {
  outline: none;
  border-color: #0078d4;
  box-shadow: 0 0 0 3px rgba(0, 120, 212, 0.15);
}

.btn {
  padding: 12px 24px;
  border: none;
  border-radius: 12px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition:
    transform 0.1s,
    box-shadow 0.2s;
}

.btn:hover {
  transform: translateY(-1px);
}

.btn:active {
  transform: translateY(0);
}

.btn-add {
  background: linear-gradient(135deg, #4a8c00, #5ca600);
  color: white;
  box-shadow: 0 2px 8px rgba(74, 140, 0, 0.3);
}

.btn-add:hover {
  box-shadow: 0 4px 12px rgba(74, 140, 0, 0.4);
}

.btn-organize {
  background: linear-gradient(135deg, #0078d4, #0086f0);
  color: white;
  box-shadow: 0 2px 8px rgba(0, 120, 212, 0.3);
}

.btn-organize:hover {
  box-shadow: 0 4px 12px rgba(0, 120, 212, 0.4);
}

/* Main wrapper */
.tag-canvas-wrapper {
  position: relative;
  padding: 20px;
  width: 100%;
  max-width: 700px;
  margin: 0 auto;
}

@media (max-width: 640px) {
  .tag-canvas-wrapper {
    padding: 10px;
  }
  .toolbar {
    gap: 8px;
  }
  .btn {
    padding: 10px 16px;
    font-size: 13px;
  }
}

/* Outer frame */
.outer-frame {
  position: relative;
  padding: 24px;
  border-radius: 32px;
  background: linear-gradient(145deg, #f0f0f0, #e6e6e6);
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.6),
    inset 0 -1px 0 rgba(0, 0, 0, 0.1),
    0 4px 8px rgba(0, 0, 0, 0.1),
    0 8px 16px rgba(0, 0, 0, 0.08),
    0 16px 32px rgba(0, 0, 0, 0.06);
}

@media (max-width: 640px) {
  .outer-frame {
    padding: 16px;
    border-radius: 24px;
  }
}

/* Inner frame */
.inner-frame {
  position: relative;
  padding: 16px;
  border-radius: 24px;
  background: linear-gradient(145deg, #fafafa, #f5f5f5);
  box-shadow:
    inset 0 2px 4px rgba(0, 0, 0, 0.06),
    inset 0 -1px 0 rgba(255, 255, 255, 0.8),
    0 2px 4px rgba(0, 0, 0, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.8);
}

@media (max-width: 640px) {
  .inner-frame {
    padding: 12px;
    border-radius: 20px;
  }
}

/* Glass canvas */
.tag-canvas {
  width: 100%;
  height: 60vh;
  min-height: 300px;
  border-radius: 20px;
  position: relative;
  overflow: hidden;
  background: linear-gradient(
    135deg,
    rgba(235, 245, 255, 0.25) 0%,
    rgba(220, 240, 255, 0.2) 50%,
    rgba(200, 230, 255, 0.15) 100%
  );
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  border: 1px solid rgba(180, 220, 255, 0.3);
}

.tag-canvas::before {
  content: '';
  position: absolute;
  top: -20%;
  left: -10%;
  width: 40%;
  height: 120%;
  background: linear-gradient(
    45deg,
    transparent 0%,
    rgba(235, 245, 255, 0.1) 20%,
    rgba(235, 245, 255, 0.3) 40%,
    rgba(240, 248, 255, 0.4) 50%,
    rgba(235, 245, 255, 0.3) 60%,
    rgba(235, 245, 255, 0.1) 80%,
    transparent 100%
  );
  transform: rotate(15deg);
  z-index: 1;
  pointer-events: none;
  border-radius: 20px;
}

.tag-canvas::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 30%;
  background: linear-gradient(
    180deg,
    rgba(240, 248, 255, 0.4) 0%,
    rgba(235, 245, 255, 0.2) 30%,
    rgba(230, 240, 255, 0.1) 60%,
    transparent 100%
  );
  z-index: 1;
  pointer-events: none;
  border-radius: 20px 20px 0 0;
}

@media (max-width: 640px) {
  .tag-canvas {
    height: 50vh;
    border-radius: 16px;
  }
}

/* Physics canvas */
.physics-canvas {
  position: absolute;
  inset: 0;
  pointer-events: auto;
  z-index: 30;
}

/* Tags */
.tag {
  position: absolute;
  display: inline-flex;
  justify-content: center;
  align-items: center;
  padding: 0 16px;
  height: 40px;
  font-size: 14px;
  font-weight: 600;
  border-radius: 20px;
  z-index: 20;
  cursor: grab;
  user-select: none;
  white-space: nowrap;
  pointer-events: none;
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.6),
    inset 0 -1px 0 rgba(0, 0, 0, 0.1),
    0 2px 4px rgba(0, 0, 0, 0.1);
  opacity: 1;
  transition: opacity 0.2s ease;
}

/* Hidden state for new tasks before physics positions them */
.tag--hidden {
  opacity: 0;
  transform: translate(-50%, -50%) translate(-9999px, -9999px);
}

/* Loading */
.loading {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #001d44;
  font-weight: 500;
  z-index: 5;
}
</style>
