<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

const target = new Date('2026-11-04T06:00:00+05:30').getTime()
const values = ref({ days: '00', hours: '00', minutes: '00', seconds: '00' })
const finished = ref(false)
const previewMode = ref(false)
const isAudioActive = ref(false)
const canvasRef = ref(null)

let timerInterval = null
let animationFrameId = null
let audioCtx = null
let windNode = null
let gainNode = null

const isArrived = computed(() => finished.value || previewMode.value)

function updateCountdown() {
  const remaining = Math.max(0, target - Date.now())
  const pad = value => String(value).padStart(2, '0')
  values.value = {
    days: pad(Math.floor(remaining / 86400000)),
    hours: pad(Math.floor((remaining / 3600000) % 24)),
    minutes: pad(Math.floor((remaining / 60000) % 60)),
    seconds: pad(Math.floor((remaining / 1000) % 60))
  }
  finished.value = remaining === 0
  if (finished.value && timerInterval) {
    clearInterval(timerInterval)
  }
}

// Particle System (Munnar Mountain Dew & Fireflies)
let particles = []
function initParticles(canvas) {
  const ctx = canvas.getContext('2d')
  let width = (canvas.width = window.innerWidth)
  let height = (canvas.height = window.innerHeight)

  const handleResize = () => {
    if (!canvas) return
    width = canvas.width = window.innerWidth
    height = canvas.height = window.innerHeight
  }
  window.addEventListener('resize', handleResize)

  const particleCount = Math.min(45, Math.floor((width * height) / 28000))
  particles = Array.from({ length: particleCount }, () => ({
    x: Math.random() * width,
    y: Math.random() * height,
    radius: Math.random() * 2.2 + 0.8,
    speedY: - (Math.random() * 0.45 + 0.15),
    speedX: (Math.random() - 0.5) * 0.35,
    baseOpacity: Math.random() * 0.65 + 0.25,
    opacity: 0.3,
    pulseOffset: Math.random() * Math.PI * 2,
    pulseSpeed: Math.random() * 0.02 + 0.015,
    hue: Math.random() > 0.4 ? '154, 75%, 72%' : '44, 95%, 76%' // Emerald mist or warm morning sunrise gold
  }))

  function render(time) {
    ctx.clearRect(0, 0, width, height)

    for (const p of particles) {
      p.y += p.speedY
      p.x += p.speedX + Math.sin(time * 0.001 + p.pulseOffset) * 0.25
      p.opacity = p.baseOpacity * (0.6 + 0.4 * Math.sin(time * p.pulseSpeed + p.pulseOffset))

      if (p.y < -10) {
        p.y = height + 10
        p.x = Math.random() * width
      }
      if (p.x < -10) p.x = width + 10
      if (p.x > width + 10) p.x = -10

      ctx.beginPath()
      ctx.arc(p.x, p.y, p.radius, 0, Math.PI * 2)
      ctx.fillStyle = `hsla(${p.hue}, ${p.opacity})`
      ctx.shadowBlur = 10
      ctx.shadowColor = `hsla(${p.hue}, 0.8)`
      ctx.fill()
      ctx.shadowBlur = 0
    }

    animationFrameId = requestAnimationFrame(render)
  }

  animationFrameId = requestAnimationFrame(render)

  return () => {
    window.removeEventListener('resize', handleResize)
    if (animationFrameId) cancelAnimationFrame(animationFrameId)
  }
}

// Ambient Mountain Breeze Sound Generator (Pure Web Audio API)
function toggleAmbientSound() {
  if (isAudioActive.value) {
    if (audioCtx) {
      audioCtx.close()
      audioCtx = null
    }
    isAudioActive.value = false
    return
  }

  try {
    const AudioContext = window.AudioContext || window.webkitAudioContext
    audioCtx = new AudioContext()
    
    // Wind noise buffer
    const bufferSize = audioCtx.sampleRate * 2
    const noiseBuffer = audioCtx.createBuffer(1, bufferSize, audioCtx.sampleRate)
    const output = noiseBuffer.getChannelData(0)
    for (let i = 0; i < bufferSize; i++) {
      output[i] = Math.random() * 2 - 1
    }

    const whiteNoise = audioCtx.createBufferSource()
    whiteNoise.buffer = noiseBuffer
    whiteNoise.loop = true

    // Lowpass filter for mountain wind
    const filter = audioCtx.createBiquadFilter()
    filter.type = 'lowpass'
    filter.frequency.setValueAtTime(260, audioCtx.currentTime)

    // Gentle LFO for gust effect
    const lfo = audioCtx.createOscillator()
    lfo.frequency.setValueAtTime(0.18, audioCtx.currentTime)
    const lfoGain = audioCtx.createGain()
    lfoGain.gain.setValueAtTime(140, audioCtx.currentTime)
    lfo.connect(lfoGain)
    lfoGain.connect(filter.frequency)
    lfo.start()

    gainNode = audioCtx.createGain()
    gainNode.gain.setValueAtTime(0.001, audioCtx.currentTime)
    gainNode.gain.exponentialRampToValueAtTime(0.08, audioCtx.currentTime + 2.5)

    whiteNoise.connect(filter)
    filter.connect(gainNode)
    gainNode.connect(audioCtx.destination)
    whiteNoise.start()

    // Occasional subtle mountain chime (pentatonic scale)
    const chimes = [587.33, 659.25, 880, 987.77, 1174.66]
    const chimeInterval = setInterval(() => {
      if (!audioCtx || audioCtx.state === 'closed') {
        clearInterval(chimeInterval)
        return
      }
      if (Math.random() > 0.45) return
      
      const osc = audioCtx.createOscillator()
      const cGain = audioCtx.createGain()
      const freq = chimes[Math.floor(Math.random() * chimes.length)]
      osc.frequency.setValueAtTime(freq, audioCtx.currentTime)
      cGain.gain.setValueAtTime(0.001, audioCtx.currentTime)
      cGain.gain.exponentialRampToValueAtTime(0.025, audioCtx.currentTime + 0.1)
      cGain.gain.exponentialRampToValueAtTime(0.0001, audioCtx.currentTime + 3.5)
      
      osc.connect(cGain)
      cGain.connect(audioCtx.destination)
      osc.start()
      osc.stop(audioCtx.currentTime + 3.6)
    }, 4000)

    isAudioActive.value = true
  } catch (err) {
    console.error('Audio could not be initialized:', err)
  }
}

let cleanupParticles = null
onMounted(() => {
  updateCountdown()
  timerInterval = setInterval(updateCountdown, 1000)
  if (canvasRef.value) {
    cleanupParticles = initParticles(canvasRef.value)
  }
})

onUnmounted(() => {
  if (timerInterval) clearInterval(timerInterval)
  if (cleanupParticles) cleanupParticles()
  if (audioCtx) audioCtx.close()
})
</script>

<template>
  <main class="munnar">
    <!-- Dynamic Animated Layers -->
    <div class="backdrop" aria-hidden="true"></div>
    <div class="sunrise-glow" aria-hidden="true"></div>
    
    <!-- Organic Drifting Mountain Mist Layers -->
    <div class="mist-layer mist-1" aria-hidden="true"></div>
    <div class="mist-layer mist-2" aria-hidden="true"></div>
    <div class="mist-layer mist-3" aria-hidden="true"></div>

    <!-- Ambient Floating Mountain Motes / Fireflies -->
    <canvas ref="canvasRef" class="ambient-canvas" aria-hidden="true"></canvas>

    <div class="overlay" aria-hidden="true"></div>

    <!-- Top Floating Ambient Audio Controller -->
    <aside class="ambience-control">
      <button 
        type="button"
        class="sound-btn" 
        :class="{ 'is-active': isAudioActive }"
        @click="toggleAmbientSound"
        :aria-label="isAudioActive ? 'Mute mountain breeze' : 'Play ambient mountain breeze sound'"
      >
        <span class="sound-waves" aria-hidden="true">
          <span class="bar bar-1"></span>
          <span class="bar bar-2"></span>
          <span class="bar bar-3"></span>
        </span>
        <span class="sound-text">{{ isAudioActive ? 'Mountain Breeze On' : 'Play Mountain Breeze' }}</span>
      </button>
    </aside>

    <section class="content">
      <!-- Animated Location Badge -->
      <div class="location-badge">
        <span class="pulse-beacon" aria-hidden="true">
          <span class="beacon-core"></span>
          <span class="beacon-wave"></span>
        </span>
        <span class="location-title">KERALA, INDIA</span>
        <span class="location-dot">•</span>
        <span class="location-sub">1,600M ELEVATION</span>
      </div>

      <!-- Hero Heading with Shimmer -->
      <h1 class="hero-title">
        <span class="shimmer-text">Munnar is waiting</span>
      </h1>

      <p class="description">
        The hills are calling. Mist, tea gardens and cool mountain air — see you at sunrise.
      </p>

      <!-- Animated Countdown Cards -->
      <div v-if="!isArrived" class="countdown" aria-label="Time remaining to November 4, 2026 at 6 AM IST">
        <div 
          v-for="(value, label, index) in values" 
          :key="label" 
          class="countdown-card"
          :class="`card-${label}`"
          :style="{ '--card-index': index }"
        >
          <div class="card-glass-glow" aria-hidden="true"></div>
          <div class="card-shimmer-highlight" aria-hidden="true"></div>
          
          <div class="number-box">
            <Transition name="tick" mode="out-in">
              <span :key="value" class="number">{{ value }}</span>
            </Transition>
          </div>
          
          <div class="unit">
            <span class="unit-text">{{ label }}</span>
            <span class="unit-line" aria-hidden="true"></span>
          </div>
        </div>
      </div>

      <!-- Celebratory Arrival Screen -->
      <div v-else class="arrived-container">
        <div class="celebration-badge">✦ THE MOMENT HAS ARRIVED ✦</div>
        <h2 class="arrived-title">Welcome to Munnar</h2>
        <p class="arrived-sub">Breathe in the mist. The tea hills greet you with morning light.</p>
        <button 
          v-if="previewMode" 
          type="button" 
          class="reset-preview-btn" 
          @click="previewMode = false"
        >
          Return to Countdown
        </button>
      </div>

      <!-- Interactive Caption & Preview Trigger -->
      <div class="caption-wrapper">
        <p class="countdown-caption">
          <span class="caption-icon" aria-hidden="true">☼</span>
          Countdown to 4 November 2026, 6:00 AM IST
        </p>
        <div class="action-row">
          <button 
            type="button" 
            class="preview-toggle" 
            @click="previewMode = !previewMode"
          >
            {{ previewMode ? '← Back to Live Countdown' : 'Preview Arrival Experience ↗' }}
          </button>
        </div>
      </div>
    </section>
  </main>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:ital,opsz,wght@0,9..40,400;0,9..40,500;0,9..40,600;0,9..40,700;1,9..40,400&family=Playfair+Display:ital,wght@0,400;0,500;0,600;0,700;1,400&display=swap');

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html, body, #app {
  width: 100%;
  min-height: 100%;
}

body {
  background: #08241c;
  font-family: 'DM Sans', -apple-system, BlinkMacSystemFont, sans-serif;
  overflow-x: hidden;
  color: #f8f5ed;
}

/* Main Container with Perspective */
.munnar {
  position: relative;
  isolation: isolate;
  min-height: 100vh;
  min-height: 100dvh;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 60px 24px 40px;
  overflow: hidden;
  color: #f8f5ed;
}

/* 1. Cinematic Ken Burns Background */
.backdrop {
  position: absolute;
  inset: -5%;
  width: 110%;
  height: 110%;
  z-index: -4;
  background: url('/munnar.jpg') center 42% / cover no-repeat;
  filter: saturate(1.15) brightness(0.88);
  animation: kenBurns 38s cubic-bezier(0.4, 0, 0.2, 1) infinite alternate;
  will-change: transform;
}

@keyframes kenBurns {
  0% {
    transform: scale(1) translate3d(0, 0, 0);
  }
  50% {
    transform: scale(1.08) translate3d(-1.5%, -1%, 0);
  }
  100% {
    transform: scale(1.04) translate3d(1%, 0.8%, 0);
  }
}

/* 2. Morning Golden Sunrise Shimmer */
.sunrise-glow {
  position: absolute;
  top: -15vh;
  left: 50%;
  transform: translateX(-50%);
  width: 90vw;
  max-width: 1100px;
  height: 65vh;
  z-index: -3;
  pointer-events: none;
  background: radial-gradient(
    ellipse at 50% 22%,
    rgba(255, 220, 130, 0.16) 0%,
    rgba(74, 180, 130, 0.14) 38%,
    rgba(2, 22, 15, 0) 72%
  );
  filter: blur(52px);
  animation: sunriseBreathe 10s ease-in-out infinite alternate;
}

@keyframes sunriseBreathe {
  0% {
    opacity: 0.55;
    transform: translateX(-50%) scale(0.95);
  }
  100% {
    opacity: 0.95;
    transform: translateX(-50%) scale(1.08);
  }
}

/* 3. Mountain Mist Layers */
.mist-layer {
  position: absolute;
  inset: 0;
  pointer-events: none;
  z-index: -2;
  mix-blend-mode: screen;
}

.mist-1 {
  background: radial-gradient(circle at 20% 65%, rgba(60, 140, 108, 0.18) 0%, transparent 60%),
              radial-gradient(circle at 80% 45%, rgba(45, 120, 92, 0.15) 0%, transparent 55%);
  filter: blur(35px);
  animation: driftMist1 32s ease-in-out infinite alternate;
}

.mist-2 {
  background: radial-gradient(circle at 75% 70%, rgba(70, 155, 120, 0.14) 0%, transparent 50%),
              radial-gradient(circle at 30% 35%, rgba(50, 130, 100, 0.13) 0%, transparent 55%);
  filter: blur(45px);
  animation: driftMist2 42s ease-in-out infinite alternate;
}

.mist-3 {
  background: linear-gradient(180deg, transparent 40%, rgba(4, 30, 21, 0.45) 75%, rgba(2, 20, 14, 0.85) 100%);
  z-index: -2;
}

@keyframes driftMist1 {
  0% {
    transform: translate3d(-6%, -2%, 0) scale(1);
  }
  100% {
    transform: translate3d(7%, 2%, 0) scale(1.05);
  }
}

@keyframes driftMist2 {
  0% {
    transform: translate3d(6%, 3%, 0) scale(1.03);
  }
  100% {
    transform: translate3d(-7%, -3%, 0) scale(0.97);
  }
}

/* 4. Canvas Floating Particles */
.ambient-canvas {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  z-index: -2;
  pointer-events: none;
}

/* 5. Deep Dark Green Atmospheric Overlay */
.overlay {
  position: absolute;
  inset: 0;
  z-index: -1;
  background: 
    radial-gradient(ellipse at 50% 50%, rgba(3, 26, 18, 0.62) 0%, rgba(2, 16, 11, 0.88) 100%),
    linear-gradient(180deg, rgba(3, 26, 18, 0.85) 0%, rgba(4, 30, 22, 0.72) 45%, rgba(2, 18, 12, 0.9) 100%);
  backdrop-filter: blur(1px);
}

/* Floating Audio Control */
.ambience-control {
  position: absolute;
  top: 24px;
  right: 28px;
  z-index: 10;
  animation: fadeDown 0.9s cubic-bezier(0.16, 1, 0.3, 1) 0.8s both;
}

.sound-btn {
  display: inline-flex;
  align-items: center;
  gap: 9px;
  padding: 8px 16px;
  background: rgba(14, 46, 36, 0.65);
  border: 1px solid rgba(166, 213, 189, 0.22);
  border-radius: 999px;
  color: #c9e4d5;
  font-family: inherit;
  font-size: 12px;
  font-weight: 500;
  letter-spacing: 0.3px;
  cursor: pointer;
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  transition: all 0.3s cubic-bezier(0.16, 1, 0.3, 1);
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
}

.sound-btn:hover {
  background: rgba(22, 64, 50, 0.85);
  color: #ffffff;
  border-color: rgba(166, 213, 189, 0.5);
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(4, 34, 24, 0.4), 0 0 15px rgba(166, 213, 189, 0.2);
}

.sound-waves {
  display: inline-flex;
  align-items: center;
  gap: 2.5px;
  height: 14px;
}

.sound-waves .bar {
  width: 2.5px;
  height: 5px;
  background: #a6d5bd;
  border-radius: 2px;
  transition: height 0.2s ease;
}

.sound-btn.is-active .bar-1 {
  animation: soundBar 0.9s ease-in-out infinite alternate;
}
.sound-btn.is-active .bar-2 {
  animation: soundBar 0.7s ease-in-out 0.2s infinite alternate;
}
.sound-btn.is-active .bar-3 {
  animation: soundBar 1.1s ease-in-out 0.4s infinite alternate;
}

@keyframes soundBar {
  0% { height: 4px; }
  100% { height: 14px; background: #ffe18d; }
}

/* Staggered Content Animation */
.content {
  position: relative;
  width: 100%;
  max-width: 1080px;
  text-align: center;
  z-index: 2;
}

/* Location Badge with Live Radar Pulse */
.location-badge {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  padding: 6px 18px;
  background: rgba(12, 44, 34, 0.55);
  border: 1px solid rgba(166, 213, 189, 0.25);
  border-radius: 999px;
  margin-bottom: 28px;
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  animation: fadeDown 0.8s cubic-bezier(0.16, 1, 0.3, 1) 0.15s both;
  box-shadow: 0 4px 18px rgba(0, 0, 0, 0.15);
}

.pulse-beacon {
  position: relative;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 10px;
  height: 10px;
}

.beacon-core {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: #4ade80;
  box-shadow: 0 0 8px #4ade80;
}

.beacon-wave {
  position: absolute;
  inset: -3px;
  border-radius: 50%;
  border: 1.5px solid #4ade80;
  animation: radarWave 2s cubic-bezier(0.1, 0.8, 0.3, 1) infinite;
}

@keyframes radarWave {
  0% {
    transform: scale(0.6);
    opacity: 1;
  }
  100% {
    transform: scale(2.6);
    opacity: 0;
  }
}

.location-title {
  color: #a6d5bd;
  font-size: 12px;
  font-weight: 700;
  letter-spacing: 1.6px;
}

.location-dot {
  color: rgba(166, 213, 189, 0.5);
  font-size: 11px;
}

.location-sub {
  color: rgba(213, 230, 222, 0.8);
  font-size: 11px;
  letter-spacing: 1px;
  font-weight: 500;
}

/* Elegant Shimmering Typography */
.hero-title {
  font-family: 'Playfair Display', Georgia, serif;
  font-size: clamp(48px, 7.5vw, 92px);
  font-weight: 400;
  line-height: 1.15;
  letter-spacing: -1.8px;
  margin: 0 auto;
  animation: fadeUp 0.9s cubic-bezier(0.16, 1, 0.3, 1) 0.3s both;
}

.shimmer-text {
  background: linear-gradient(
    135deg,
    #ffffff 0%,
    #e3f6ec 30%,
    #fff3d1 50%,
    #ffffff 75%,
    #aee2cc 100%
  );
  background-size: 250% auto;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  animation: textShimmer 9s linear infinite;
  display: inline-block;
}

@keyframes textShimmer {
  0% {
    background-position: 0% center;
  }
  100% {
    background-position: 250% center;
  }
}

.description {
  margin: 28px auto 0;
  font-size: clamp(15px, 1.6vw, 19px);
  line-height: 1.8;
  color: rgba(240, 246, 242, 0.92);
  max-width: 660px;
  font-weight: 400;
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
  animation: fadeUp 0.9s cubic-bezier(0.16, 1, 0.3, 1) 0.45s both;
}

/* Interactive Glass Countdown Cards */
.countdown {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 20px;
  max-width: 860px;
  margin: 68px auto 0;
  perspective: 1000px;
}

.countdown-card {
  position: relative;
  background: rgba(3, 26, 18, 0.62);
  border: 1px solid rgba(166, 213, 189, 0.16);
  border-radius: 20px;
  min-height: 152px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  backdrop-filter: blur(14px);
  -webkit-backdrop-filter: blur(14px);
  overflow: hidden;
  box-shadow: 0 16px 35px -8px rgba(1, 14, 9, 0.7), inset 0 1px 1px rgba(255, 255, 255, 0.14);
  transition: transform 0.4s cubic-bezier(0.16, 1, 0.3, 1),
              box-shadow 0.4s cubic-bezier(0.16, 1, 0.3, 1),
              border-color 0.4s ease,
              background-color 0.4s ease;
  animation: cardEntrance 0.85s cubic-bezier(0.16, 1, 0.3, 1) both;
  animation-delay: calc(0.55s + (var(--card-index, 0) * 0.1s));
  cursor: default;
}

/* Floating animation on cards */
.countdown-card:hover {
  transform: translateY(-8px) scale(1.025);
  background: rgba(5, 36, 25, 0.78);
  border-color: rgba(166, 213, 189, 0.45);
  box-shadow: 0 24px 45px -10px rgba(1, 14, 9, 0.88),
              0 0 25px rgba(74, 222, 128, 0.22),
              inset 0 1px 2px rgba(255, 255, 255, 0.25);
}

/* Subtle gloss light beam sweep across cards on hover */
.card-shimmer-highlight {
  position: absolute;
  inset: 0;
  background: linear-gradient(
    115deg,
    transparent 0%,
    transparent 40%,
    rgba(255, 255, 255, 0.08) 50%,
    transparent 60%,
    transparent 100%
  );
  transform: translateX(-100%);
  transition: transform 0.8s ease;
  pointer-events: none;
}

.countdown-card:hover .card-shimmer-highlight {
  transform: translateX(100%);
}

/* Number Container & Tick Transition */
.number-box {
  position: relative;
  height: clamp(52px, 6vw, 76px);
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  width: 100%;
}

.number {
  font-family: 'Playfair Display', Georgia, serif;
  font-size: clamp(44px, 5.5vw, 68px);
  line-height: 1;
  font-weight: 500;
  font-variant-numeric: tabular-nums;
  letter-spacing: -0.5px;
  color: #ffffff;
  text-shadow: 0 4px 18px rgba(0, 0, 0, 0.4);
  display: inline-block;
  will-change: transform, opacity;
}

/* Seconds card special heartbeat glow */
.card-seconds .number {
  color: #f6fffa;
  text-shadow: 0 0 20px rgba(166, 213, 189, 0.35), 0 4px 18px rgba(0, 0, 0, 0.4);
}

/* Smooth Tick Roll Animation */
.tick-enter-active {
  transition: transform 0.38s cubic-bezier(0.2, 0.85, 0.25, 1),
              opacity 0.32s ease,
              filter 0.32s ease;
}

.tick-leave-active {
  transition: transform 0.28s cubic-bezier(0.4, 0, 1, 1),
              opacity 0.25s ease,
              filter 0.25s ease;
  position: absolute;
}

.tick-enter-from {
  opacity: 0;
  transform: translateY(-45%) scale(0.92);
  filter: blur(4px);
}

.tick-leave-to {
  opacity: 0;
  transform: translateY(45%) scale(0.92);
  filter: blur(4px);
}

/* Unit Label */
.unit {
  position: relative;
  margin-top: 10px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
}

.unit-text {
  font-family: 'DM Sans', sans-serif;
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 2px;
  text-transform: uppercase;
  color: #a6d5bd;
}

.unit-line {
  width: 16px;
  height: 1.5px;
  background: rgba(166, 213, 189, 0.3);
  border-radius: 2px;
  transition: width 0.3s ease, background-color 0.3s ease;
}

.countdown-card:hover .unit-line {
  width: 28px;
  background: #a6d5bd;
}

/* Arrival Container */
.arrived-container {
  margin: 60px auto 20px;
  animation: arrivalBounce 1s cubic-bezier(0.16, 1, 0.3, 1) both;
}

.celebration-badge {
  font-size: 12px;
  font-weight: 700;
  letter-spacing: 2.5px;
  color: #ffd875;
  margin-bottom: 16px;
  animation: pulseGold 2s ease-in-out infinite alternate;
}

.arrived-title {
  font-family: 'Playfair Display', Georgia, serif;
  font-size: clamp(38px, 6vw, 64px);
  font-weight: 500;
  background: linear-gradient(135deg, #ffffff 0%, #ffe29a 60%, #a6d5bd 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  margin-bottom: 14px;
}

.arrived-sub {
  font-size: 17px;
  color: rgba(255, 255, 255, 0.88);
  max-width: 520px;
  margin: 0 auto 28px;
  line-height: 1.6;
}

.reset-preview-btn {
  background: rgba(255, 255, 255, 0.12);
  border: 1px solid rgba(255, 255, 255, 0.25);
  color: #ffffff;
  padding: 10px 22px;
  border-radius: 999px;
  font-family: inherit;
  font-size: 13px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.reset-preview-btn:hover {
  background: rgba(255, 255, 255, 0.22);
  transform: translateY(-2px);
}

/* Captions and Interactive Action Row */
.caption-wrapper {
  margin-top: 42px;
  animation: fadeUp 0.85s cubic-bezier(0.16, 1, 0.3, 1) 0.95s both;
}

.countdown-caption {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-size: 14px;
  color: rgba(213, 230, 222, 0.88);
  letter-spacing: 0.4px;
}

.caption-icon {
  color: #ffd875;
  font-size: 14px;
  animation: rotateSun 16s linear infinite;
}

@keyframes rotateSun {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.action-row {
  margin-top: 18px;
}

.preview-toggle {
  background: transparent;
  border: 1px solid rgba(166, 213, 189, 0.25);
  color: #a6d5bd;
  padding: 6px 16px;
  border-radius: 999px;
  font-family: inherit;
  font-size: 12px;
  cursor: pointer;
  letter-spacing: 0.3px;
  transition: all 0.25s ease;
}

.preview-toggle:hover {
  background: rgba(166, 213, 189, 0.12);
  border-color: rgba(166, 213, 189, 0.5);
  color: #ffffff;
  transform: translateY(-1px);
}

/* Entrance Keyframes */
@keyframes fadeDown {
  from {
    opacity: 0;
    transform: translateY(-20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes fadeUp {
  from {
    opacity: 0;
    transform: translateY(24px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes cardEntrance {
  0% {
    opacity: 0;
    transform: translateY(35px) scale(0.92);
  }
  100% {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

@keyframes arrivalBounce {
  0% {
    opacity: 0;
    transform: scale(0.85) translateY(20px);
  }
  100% {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}

@keyframes pulseGold {
  0% { opacity: 0.7; }
  100% { opacity: 1; text-shadow: 0 0 12px rgba(255, 216, 117, 0.6); }
}

/* Mobile Responsiveness */
@media (max-width: 768px) {
  .munnar {
    padding: 70px 16px 36px;
  }
  
  .ambience-control {
    top: 16px;
    right: 16px;
  }
  
  .sound-btn {
    padding: 6px 12px;
    font-size: 11px;
  }

  .location-badge {
    padding: 5px 14px;
    margin-bottom: 20px;
  }

  .location-title {
    font-size: 11px;
  }

  .location-sub {
    font-size: 10px;
  }

  .hero-title {
    font-size: clamp(40px, 9.5vw, 62px);
    letter-spacing: -1px;
  }

  .description {
    font-size: 14px;
    margin-top: 18px;
    line-height: 1.65;
  }

  .countdown {
    gap: 10px;
    margin-top: 42px;
  }

  .countdown-card {
    min-height: 108px;
    border-radius: 14px;
    padding: 8px 4px;
  }

  .number-box {
    height: 48px;
  }

  .number {
    font-size: clamp(32px, 8vw, 44px);
  }

  .unit-text {
    font-size: 9px;
    letter-spacing: 1.2px;
  }

  .unit-line {
    width: 10px;
    height: 1px;
  }

  .countdown-caption {
    font-size: 12px;
  }
}

@media (max-width: 420px) {
  .countdown {
    gap: 8px;
  }
  .countdown-card {
    min-height: 96px;
  }
  .number {
    font-size: 28px;
  }
}

/* Accessibility: Reduced Motion */
@media (prefers-reduced-motion: reduce) {
  .backdrop,
  .sunrise-glow,
  .mist-layer,
  .content,
  .location-badge,
  .hero-title,
  .shimmer-text,
  .description,
  .countdown-card,
  .caption-wrapper,
  .caption-icon,
  .beacon-wave,
  .tick-enter-active,
  .tick-leave-active {
    animation: none !important;
    transition: none !important;
  }
}
</style>
