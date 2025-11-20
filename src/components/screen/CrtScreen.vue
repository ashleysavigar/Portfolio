<template>
  <div ref="screen" id="screen">
    <slot></slot>
  </div>
</template>

<script setup>
import { onMounted, onBeforeUnmount, ref } from 'vue'

const props = defineProps({
  /** Displays horizontal scanlines across the screen */
  showScanlines: {
    type: Boolean,
    default: true
  },
  /** Displays the CRT vignette overlay with curved edge darkening */
  showVignette: {
    type: Boolean,
    default: true
  },
  /** Enables subtle vertical wobble animation effect */
  showWobbley: {
    type: Boolean,
    default: true
  },
  /** Enables vertical scrolling roll effect */
  showRoll: {
    type: Boolean,
    default: false
  },
  /** Displays animated static/snow effect */
  showSnow: {
    type: Boolean,
    default: true
  },
  /** Opacity level of the snow effect (0-1) */
  snowOpacity: {
    type: Number,
    default: 0.2
  },
  /** Displays VCR tracking noise effect */
  showVcr: {
    type: Boolean,
    default: true
  },
  /** Opacity level of the VCR tracking noise (0-1) */
  vcrOpacity: {
    type: Number,
    default: 0.6
  },
  /** Blur amount in pixels for the VCR effect */
  vcrBlur: {
    type: Number,
    default: 1.5
  },
  /** Vertical position of VCR tracking noise bands */
  vcrTracking: {
    type: Number,
    default: 220
  },
  /** Number of tracking noise artifacts (simulates tape wear) */
  tapeAge: {
    type: Number,
    default: 70
  },
  /** Displays a background image behind the CRT effects */
  showImage: {
    type: Boolean,
    default: false
  },
  /** URL of the background image to display */
  imageSrc: {
    type: String,
    default: ''
  },
  /** Blur amount in pixels for the background image */
  imageBlur: {
    type: Number,
    default: 1.2
  },
})

const screen = ref(null)
let screenEffect

const getRandomInt = (min, max) => {
  min = Math.ceil(min)
  max = Math.floor(max)
  return Math.floor(Math.random() * (max - min + 1)) + min
}

class ScreenEffect {
  constructor(parent, options) {
    this.parent = typeof parent === 'string' ? document.querySelector(parent) : parent
    this.config = Object.assign({}, options)
    this.effects = {}
    this.events = {
      resize: this.onResize.bind(this),
    }
    window.addEventListener('resize', this.events.resize, false)
    this.render()
  }

  render() {
    const container = document.createElement('div')
    container.classList.add('screen-container')

    const w1 = document.createElement('div')
    w1.classList.add('screen-wrapper')

    const w2 = document.createElement('div')
    w2.classList.add('screen-wrapper')

    const w3 = document.createElement('div')
    w3.classList.add('screen-wrapper')

    w1.appendChild(w2)
    w2.appendChild(w3)
    container.appendChild(w1)

    this.parent.parentNode.insertBefore(container, this.parent)
    w3.appendChild(this.parent)

    this.nodes = { container, w1, w2, w3 }
    this.onResize()
  }

  onResize() {
    this.rect = this.parent.getBoundingClientRect()

    if (this.effects.vcr?.enabled) {
      this.generateVCRNoise()
    }
  }

  add(type, options = {}) {
    const config = Object.assign({ fps: 30, blur: 1 }, options)

    if (Array.isArray(type)) {
      for (const t of type) {
        this.add(t)
      }
      return this
    }

    if (type === 'snow') {
      return this.createSnow(config)
    }

    if (type === 'vcr') {
      return this.createVCR(config)
    }

    if (type === 'roll') {
      return this.enableRoll()
    }

    let node = null
    let wrapper = this.nodes.w2

    switch (type) {
      case 'wobblex':
      case 'wobbley':
        this.nodes.container.classList.add(type)
        this.effects[type] = { wrapper: this.nodes.container, node: null, enabled: true, config }
        return this

      case 'scanlines':
        node = document.createElement('div')
        node.classList.add(type)
        wrapper.appendChild(node)
        break

      case 'vignette':
        wrapper = this.nodes.container
        node = document.createElement('div')
        node.classList.add(type)
        wrapper.appendChild(node)
        break

      case 'image':
        wrapper = this.parent
        node = document.createElement('img')
        node.classList.add(type)
        node.src = config.src
        wrapper.appendChild(node)
        break

      case 'video':
        node = document.createElement('video')
        node.classList.add(type)
        node.src = config.src
        node.crossOrigin = 'anonymous'
        node.autoplay = true
        node.muted = true
        node.loop = true
        wrapper.appendChild(node)
        break
  }

    this.effects[type] = { wrapper, node, enabled: true, config }
    return this
  }

  remove(type) {
    const obj = this.effects[type]

    if (!obj) {
      return this
    }

    obj.enabled = false

    if (type === 'vcr') {
      clearInterval(this.vcrInterval)
    }

    if (type === 'snow') {
      cancelAnimationFrame(this.snowframe)
    }

    if (obj.node) {
      obj.wrapper.removeChild(obj.node)
    } else {
      obj.wrapper.classList.remove(type)
    }

    return this
  }

  createSnow(config) {
    const c = document.createElement('canvas')
    const ctx = c.getContext('2d')
    c.classList.add('snow')
    const width = Math.max(1, Math.floor(this.rect.width / 2))
    const height = Math.max(1, Math.floor(this.rect.height / 2))
    c.width = width
    c.height = height

    if (config.opacity !== undefined) {
      c.style.opacity = config.opacity
    }

    this.nodes.w2.appendChild(c)

    const animate = () => {
      this.generateSnow(ctx)
      this.snowframe = requestAnimationFrame(animate)
    }

    animate()
    this.effects.snow = { wrapper: this.nodes.w2, node: c, enabled: true, config }
    return this
  }

  generateSnow(ctx) {
    const w = ctx.canvas.width
    const h = ctx.canvas.height
    const d = ctx.createImageData(w, h)
    const b = new Uint32Array(d.data.buffer)

    for (let i = 0; i < b.length; i++) {
      b[i] = ((255 * Math.random()) | 0) << 24
    }

    ctx.putImageData(d, 0, 0)
  }

  createVCR(config) {
    const c = document.createElement('canvas')
    c.classList.add('vcr')
    this.nodes.w2.appendChild(c)
    c.width = Math.max(1, Math.floor(this.rect.width))
    c.height = Math.max(1, Math.floor(this.rect.height))
    this.effects.vcr = {
      wrapper: this.nodes.w2,
      node: c,
      ctx: c.getContext('2d'),
      enabled: true,
      config,
    }
    this.generateVCRNoise()
    return this
  }

  generateVCRNoise() {
    const { config } = this.effects.vcr

    if (config.fps >= 60) {
      cancelAnimationFrame(this.vcrInterval)
      const animate = () => {
        this.renderTrackingNoise()
        this.vcrInterval = requestAnimationFrame(animate)
      }
      animate()
    } else {
      clearInterval(this.vcrInterval)
      this.vcrInterval = setInterval(() => this.renderTrackingNoise(), 1000 / config.fps)
    }
  }

  renderTrackingNoise(radius = 2, xmax, ymax) {
    const canvas = this.effects.vcr.node
    const ctx = this.effects.vcr.ctx
    const config = this.effects.vcr.config
    let posy1 = config.miny || 0
    let posy2 = config.maxy || canvas.height
    let posy3 = config.miny2 || 0
    const num = config.num || 20

    xmax ??= canvas.width
    ymax ??= canvas.height
    canvas.style.filter = `blur(${config.blur}px)`
    ctx.clearRect(0, 0, canvas.width, canvas.height)
    ctx.fillStyle = '#fff'

    ctx.beginPath()

    for (let i = 0; i <= num; i++) {
      const x = Math.random(i) * xmax
      const y1 = getRandomInt((posy1 += 3), posy2)
      const y2 = getRandomInt(0, (posy3 -= 3))
      ctx.fillRect(x, y1, radius, radius)
      ctx.fillRect(x, y2, radius, radius)
      ctx.fill()
      this.renderTail(ctx, x, y1, radius)
      this.renderTail(ctx, x, y2, radius)
    }

    ctx.closePath()
  }

  renderTail(ctx, x, y, radius) {
    const n = getRandomInt(1, 50)
    const dirs = [1, -1]
    let rd = radius
    const dir = dirs[Math.floor(Math.random() * dirs.length)]

    for (let i = 0; i < n; i++) {
      const step = 0.01
      let r = getRandomInt((rd -= step), radius)
      let dx = getRandomInt(1, 4)
      radius -= 0.1
      dx *= dir
      ctx.fillRect((x += dx), y, r, r)
      ctx.fill()
    }
  }

  enableRoll() {
    const el = this.parent.firstElementChild

    if (!el) {
      return this
    }

    const div = document.createElement('div')
    div.classList.add('roller')
    this.parent.appendChild(div)
    div.appendChild(el)
    div.appendChild(el.cloneNode(true))
    this.effects.roll = { enabled: true, wrapper: this.parent, node: div, original: el }
  }
}

onMounted(() => {
  screenEffect = new ScreenEffect(screen.value, {})

  if (props.showImage && props.imageSrc) {
    screenEffect.add('image', { src: props.imageSrc, blur: props.imageBlur })
  }

  if (props.showScanlines) {
    screenEffect.add('scanlines')
  }

  if (props.showVignette) {
    screenEffect.add('vignette')
  }

  if (props.showSnow) {
    screenEffect.add('snow', { opacity: props.snowOpacity || 0.2 })
  }

  if (props.showVcr) {
    screenEffect.add('vcr', {
      blur: props.vcrBlur,
      num: props.tapeAge,
      fps: 60,
      miny: props.vcrTracking,
      miny2: 400 - props.vcrTracking,
    })
  }

  if (props.showWobbley) {
    screenEffect.add('wobbley')
  }

  if (props.showRoll) {
    screenEffect.add('roll')
  }

  if (props.showVcr && screenEffect.effects.vcr?.node) {
    screenEffect.effects.vcr.node.style.opacity = props.vcrOpacity
  }

  if (props.showImage && screenEffect.effects.image?.node) {
    screenEffect.effects.image.node.style.filter = `blur(${props.imageBlur}px)`
  }
})

onBeforeUnmount(() => {
  window.removeEventListener('resize', screenEffect.events.resize)
})
</script>

<style>
#screen {
  width: 100%;
  height: 100%;
  background: #171DA8;
  background-size: cover;
  animation: blue-flicker 5ms infinite;
  position: relative;
}

#screen::after {
  box-shadow: inset 0 0 10em rgba(0, 0, 0, 0.75);
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 9997;
  content: "";
  pointer-events: none;
}

@keyframes blue-flicker {
  28% { background: #171DA8; }
  30% { background: rgba(23, 29, 168, 0.95); }
  33% { background: #171DA8; }
  34% { background: rgba(23, 29, 168, 0.9); }
  35% { background: #171DA8; }
}

.screen-container {
  width: 100%;
  height: 100%;
  overflow: hidden;
  position: relative;
}

.screen-container .screen-wrapper {
  position: relative;
  width: 100%;
  height: 100%;
}

.screen-container canvas {
  position: absolute;
  left: 0;
  top: 0;
  z-index: 9998;
  width: 100%;
  height: 100%;
}

.screen-container canvas.snow {
  background-color: #aaa;
  opacity: 0.2;
}

.screen-container canvas.vcr {
  opacity: 0.6;
}

.screen-container .vignette {
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background-image: url(/crt.png);
  background-repeat: no-repeat;
  background-size: 100% 100%;
  z-index: 10000;
  pointer-events: none;
}

.screen-container .scanlines {
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  z-index: 9999;
  pointer-events: none;
  background: linear-gradient(
    rgba(18, 16, 16, 0) 50%,
    rgba(0, 0, 0, 0.25) 50%
  ),
  linear-gradient(
    90deg,
    rgba(255, 0, 0, 0.06),
    rgba(0, 255, 0, 0.02),
    rgba(0, 0, 255, 0.06)
  );
  background-size: 100% 2px, 3px 100%;
}

.screen-container .video {
  filter: blur(1px);
  width: 100%;
  height: 100%;
}

#screen .image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  filter: blur(1.2px);
}

.screen-container.wobblex {
  animation: wobblex 100ms infinite;
}

.screen-container.wobbley {
  animation: wobbley 100ms infinite;
}

.glitch {
  animation: glitch 5s ease 2000ms infinite;
}

.overlay .text {
  animation: glitch 5s ease 2000ms infinite;
}

.on > .screen-wrapper {
  animation: on 3s linear forwards;
}

.off > .screen-wrapper {
  animation: off 0.75s cubic-bezier(0.23, 1, 0.32, 1) forwards;
}

.roller {
  position: relative;
  animation: roll 2s linear infinite;
}

.dg.ac {
  z-index: 10000 !important;
}

@keyframes wobblex {
  50% {
    transform: translateX(1px);
  }

  51% {
    transform: translateX(0);
  }
}

@keyframes wobbley {
  0% {
    transform: translateY(0px);
  }

  50% {
    transform: translateY(2px);
  }

  100% {
    transform: translateY(0px);
  }
}

@keyframes glitch {
  40% {
    opacity: 1;
    transform: scale(1, 1) skew(0, 0);
  }

  41% {
    opacity: 0.8;
    transform: scale(1, 1.2) skew(80deg, 0);
  }

  42% {
    opacity: 0.8;
    transform: scale(1, 1.2) skew(-50deg, 0);
  }

  43% {
    opacity: 1;
    transform: scale(1, 1) skew(0, 0);
  }
}

@keyframes on {
  0% {
    transform: scale(1, 0.8);
    filter: brightness(4);
    opacity: 1;
  }

  3.5% {
    transform: scale(1, 0.8) translateY(100%);
  }

  3.6% {
    transform: scale(1, 0.8) translateY(-100%);
    opacity: 1;
  }

  9% {
    transform: scale(1.3, 0.6) translateY(100%);
    filter: brightness(4);
    opacity: 0;
  }

  11% {
    transform: scale(1, 1);
    filter: contrast(0) brightness(0);
    opacity: 0;
  }

  100% {
    transform: scale(1, 1);
    filter: contrast(1) brightness(1.2) saturate(1.3);
    opacity: 1;
  }
}

@keyframes off {
  0% {
    transform: scale(1, 1);
    filter: brightness(1);
  }

  40% {
    transform: scale(1, 0.005);
    filter: brightness(100);
  }

  70% {
    transform: scale(1, 0.005);
  }

  90% {
    transform: scale(0.005, 0.005);
  }

  100% {
    transform: scale(0, 0);
  }
}

@keyframes roll {
  from {
    transform: translateY(0);
  }

  to {
    transform: translateY(-50%);
  }
}
</style>
