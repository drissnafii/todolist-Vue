<script setup lang="ts">
import { ref, onMounted, onUnmounted, nextTick, type ComponentPublicInstance } from 'vue'

// Matter.js types - external library with dynamic types

declare global {
  interface Window {
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    Matter: any
  }
}

// Tags with soft pastel colors
const tagsData = [
  { text: 'BUBBLE.IO', color: '#d2f18a', textColor: '#4a8c00' },
  { text: 'NOCODE', color: '#f0ee8a', textColor: '#9c7b00' },
  { text: 'EXPRESSION', color: '#e9d6ff', textColor: '#7e3bff' },
  { text: 'OPTION SET', color: '#ffd1d2', textColor: '#d13438' },
  { text: 'INTERFACE', color: '#c7e2ff', textColor: '#0078d4' },
  { text: 'VERSION', color: '#ffe0c9', textColor: '#ca5010' },
  { text: 'MOBILE', color: '#c9f5e9', textColor: '#0b8a70' },
  { text: 'UX/UI', color: '#ffd6ef', textColor: '#c2185b' },
  { text: 'ELEMENT', color: '#d2f18a', textColor: '#4a8c00' },
  { text: 'REPEATING GROUP', color: '#f0ee8a', textColor: '#9c7b00' },
  { text: 'API CONNECTOR', color: '#e9d6ff', textColor: '#7e3bff' },
  { text: 'CUSTOM STATE', color: '#ffd1d2', textColor: '#d13438' },
  { text: 'CONDITIONAL', color: '#c7e2ff', textColor: '#0078d4' },
  { text: 'DYNAMIC DATA', color: '#ffe0c9', textColor: '#ca5010' },
  { text: 'PLUGINS', color: '#c9f5e9', textColor: '#0b8a70' },
  { text: 'BACKEND WORKFLOW', color: '#f0ee8a', textColor: '#9c7b00' },
  { text: 'PRIVACY RULES', color: '#c5f0f0', textColor: '#038387' },
  { text: 'EVENTS', color: '#e9d6ff', textColor: '#7e3bff' },
  { text: 'DESIGN', color: '#c5f0f0', textColor: '#038387' },
]

const containerRef = ref<HTMLDivElement | null>(null)
const canvasRef = ref<HTMLCanvasElement | null>(null)
const tagRefs = ref<HTMLDivElement[]>([])
const isLoaded = ref(false)

// eslint-disable-next-line @typescript-eslint/no-explicit-any
let engine: any = null
// eslint-disable-next-line @typescript-eslint/no-explicit-any
let render: any = null
// eslint-disable-next-line @typescript-eslint/no-explicit-any
let tagBodies: { body: any; element: HTMLDivElement }[] = []

const setTagRef = (el: Element | ComponentPublicInstance | null, index: number) => {
  if (el && el instanceof HTMLDivElement) {
    tagRefs.value[index] = el
  }
}

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
  engine.world.gravity.y = 0.8

  // Create renderer (invisible, just for mouse interaction)
  render = Render.create({
    element: container,
    canvas: canvasRef.value,
    engine: engine,
    options: {
      width: containerWidth,
      height: containerHeight,
      background: 'transparent',
      wireframes: false,
      showVelocity: false,
      showAngleIndicator: false,
      showDebug: false,
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

  World.add(engine.world, [ground, wallLeft, wallRight])

  // Create tag bodies - start them at the BOTTOM (already settled)
  const tagsPerRow = 4
  tagBodies = tagRefs.value
    .map((tagElement, idx) => {
      if (!tagElement) return null

      const rect = tagElement.getBoundingClientRect()
      const width = rect.width
      const height = rect.height

      // Spread tags across the bottom, with slight randomness
      const row = Math.floor(idx / tagsPerRow)
      const col = idx % tagsPerRow

      // Calculate position - spread across bottom with some randomness
      const xSpacing = containerWidth / (tagsPerRow + 1)
      const x = xSpacing * (col + 1) + (Math.random() - 0.5) * 60
      const y = containerHeight - height / 2 - 20 - row * 55 - Math.random() * 30

      // Random initial rotation
      const angle = (Math.random() - 0.5) * 0.5

      const body = Bodies.rectangle(x, y, width, height, {
        chamfer: { radius: height / 2 },
        density: 0.001,
        friction: 0.3,
        frictionAir: 0.01,
        restitution: 0.6,
        angle: angle,
        render: { fillStyle: 'transparent' },
      })

      World.add(engine.world, body)
      return { body, element: tagElement }
    })
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    .filter(Boolean) as { body: any; element: HTMLDivElement }[]

  // Sync DOM positions with physics
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

  // Add mouse constraint for dragging
  const mouse = Mouse.create(render.canvas)

  // Fix mouse offset - important for proper drag behavior
  Mouse.setOffset(mouse, { x: 0, y: 0 })

  const mouseConstraint = MouseConstraint.create(engine, {
    mouse: mouse,
    constraint: {
      stiffness: 0.2,
      render: { visible: false },
    },
  })

  World.add(engine.world, mouseConstraint)

  // Keep mouse in sync with rendering
  render.mouse = mouse

  // Start the engine and renderer
  Render.run(render)
  Engine.run(engine)
}

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
  await loadMatterJS()
  isLoaded.value = true

  // Wait for DOM to update with tag elements
  await nextTick()

  // Small delay to ensure tags are rendered
  setTimeout(initPhysics, 100)
})

onUnmounted(() => {
  if (render) {
    const Matter = window.Matter
    Matter.Render.stop(render)
    Matter.Engine.clear(engine)
    Matter.World.clear(engine.world, false)
  }
})
</script>

<template>
  <div class="tag-canvas-wrapper">
    <div class="outer-frame">
      <div class="inner-frame">
        <div ref="containerRef" class="tag-canvas">
          <!-- Hidden canvas for Matter.js mouse interaction -->
          <canvas ref="canvasRef" class="physics-canvas"></canvas>

          <!-- Tag elements -->
          <div
            v-for="(tag, index) in tagsData"
            :key="index"
            :ref="(el) => setTagRef(el, index)"
            class="tag"
            :style="{
              backgroundColor: tag.color,
              color: tag.textColor,
            }"
          >
            {{ tag.text }}
          </div>

          <!-- Loading state -->
          <div v-if="!isLoaded" class="loading">Loading physics...</div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
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
}

/* Outer frame - Grey border */
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

/* Inner frame - White border */
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

  /* Glassmorphism with blue tint */
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

/* Main diagonal reflection */
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

/* Top edge glow */
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
    backdrop-filter: blur(8px);
    -webkit-backdrop-filter: blur(8px);
  }
}

/* Physics canvas - MUST be on top to capture mouse events for Matter.js */
.physics-canvas {
  position: absolute;
  inset: 0;
  pointer-events: auto;
  z-index: 30;
}

/* Tag pill styles */
.tag {
  position: absolute;
  display: inline-flex;
  justify-content: center;
  align-items: center;
  padding: 0 16px;
  height: 40px;
  font-size: 14px;
  text-transform: uppercase;
  font-weight: 600;
  border-radius: 20px;
  z-index: 20;
  cursor: grab;
  user-select: none;
  white-space: nowrap;
  pointer-events: none; /* Matter.js handles interaction via canvas */

  /* Inner shadows for depth/volume */
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.6),
    inset 0 -1px 0 rgba(0, 0, 0, 0.1),
    0 2px 4px rgba(0, 0, 0, 0.1);

  transition: box-shadow 0.15s ease;
}

.tag:hover {
  box-shadow:
    inset 0 1px 0 rgba(255, 255, 255, 0.8),
    inset 0 -1px 0 rgba(0, 0, 0, 0.15),
    0 4px 8px rgba(0, 0, 0, 0.15);
}

.tag:active {
  cursor: grabbing;
}

/* Loading state */
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

/* Attribution link */
.attribution {
  display: block;
  text-align: center;
  font-size: 14px;
  color: #9ca3af;
  margin-top: 16px;
  text-decoration: none;
  transition: color 0.3s ease;
}

.attribution:hover {
  color: #6b7280;
}
</style>
