<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

const target = new Date('2026-11-04T06:00:00+05:30').getTime()
const values = ref({ days: '00', hours: '00', minutes: '00', seconds: '00' })
const finished = ref(false)
const previewMode = ref(false)
const isFullscreen = ref(false)

let timerInterval = null

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
  if (remaining === 0) {
    finished.value = true
    if (timerInterval) {
      clearInterval(timerInterval)
      timerInterval = null
    }
  }
}

function toggleFullscreen() {
  if (!document.fullscreenElement) {
    document.documentElement.requestFullscreen().catch(() => {})
    isFullscreen.value = true
  } else {
    document.exitFullscreen().catch(() => {})
    isFullscreen.value = false
  }
}

onMounted(() => {
  updateCountdown()
  timerInterval = setInterval(updateCountdown, 1000)
  
  const handleFsChange = () => {
    isFullscreen.value = !!document.fullscreenElement
  }
  document.addEventListener('fullscreenchange', handleFsChange)
})

onUnmounted(() => {
  if (timerInterval) {
    clearInterval(timerInterval)
    timerInterval = null
  }
})
</script>

<template>
  <main class="munnar">
    <!-- Static optimized background (Zero memory leaks on TV browsers) -->
    <div class="backdrop" aria-hidden="true"></div>

    <!-- Pure CSS gradient lighting (No filter:blur overhead) -->
    <div class="sunrise-glow" aria-hidden="true"></div>
    <div class="mist-gradient" aria-hidden="true"></div>

    <!-- Deep dark green overlay -->
    <div class="overlay" aria-hidden="true"></div>

    <!-- TV Controls -->
    <aside class="tv-controls">
      <button 
        type="button" 
        class="ctrl-btn" 
        @click="toggleFullscreen"
        title="Toggle TV Fullscreen"
      >
        {{ isFullscreen ? 'Exit Fullscreen' : '⛶ Fullscreen' }}
      </button>
    </aside>

    <section class="content">
      <!-- Location Badge with gentle CSS radar wave -->
      <div class="location-badge">
        <span class="pulse-beacon" aria-hidden="true">
          <span class="beacon-core"></span>
          <span class="beacon-wave"></span>
        </span>
        <span class="location-title">KERALA, INDIA</span>
        <span class="location-dot">•</span>
        <span class="location-sub">MUNNAR HILLS</span>
      </div>

      <!-- Hero Heading -->
      <h1 class="hero-title">Munnar is waiting</h1>

      <p class="description">
        The hills are calling. Mist, tea gardens and cool mountain air — see you at sunrise.
      </p>

      <!-- TV-Optimized Countdown Cards (No backdrop-filter = Zero GPU crash) -->
      <div v-if="!isArrived" class="countdown" aria-label="Time remaining to November 4, 2026 at 6 AM IST">
        <div 
          v-for="(value, label) in values" 
          :key="label" 
          class="countdown-card"
          :class="`card-${label}`"
        >
          <div class="number-box">
            <Transition name="tv-tick" mode="out-in">
              <span :key="value" class="number">{{ value }}</span>
            </Transition>
          </div>
          
          <div class="unit">
            <span class="unit-text">{{ label }}</span>
          </div>
        </div>
      </div>

      <!-- Arrival Experience -->
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

      <!-- Captions and Preview Mode -->
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
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:wght@400;500;600;700&family=Playfair+Display:wght@400;500;600;700&display=swap');

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html, body, #app {
  width: 100%;
  height: 100%;
  overflow: hidden;
}

body {
  background: #02140d;
  font-family: 'DM Sans', -apple-system, BlinkMacSystemFont, sans-serif;
  color: #f8f5ed;
  -webkit-font-smoothing: antialiased;
}

/* Main Container */
.munnar {
  position: relative;
  width: 100%;
  height: 100vh;
  height: 100dvh;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 30px 24px;
  overflow: hidden;
  color: #f8f5ed;
}

/* 1. Optimized TV Background (Static, zero composite churn) */
.backdrop {
  position: absolute;
  inset: 0;
  z-index: -4;
  background: url('/munnar.jpg') center 42% / cover no-repeat;
  filter: brightness(0.85);
  transform: translateZ(0); /* Hardware accelerate once without repainting */
}

/* 2. Pure CSS Radial Glows (No expensive filter:blur()) */
.sunrise-glow {
  position: absolute;
  top: -10vh;
  left: 50%;
  transform: translateX(-50%);
  width: 90vw;
  max-width: 1200px;
  height: 60vh;
  z-index: -3;
  pointer-events: none;
  background: radial-gradient(
    ellipse at 50% 20%,
    rgba(255, 220, 130, 0.14) 0%,
    rgba(74, 180, 130, 0.08) 35%,
    transparent 70%
  );
}

.mist-gradient {
  position: absolute;
  inset: 0;
  z-index: -2;
  pointer-events: none;
  background: 
    radial-gradient(ellipse at 25% 65%, rgba(40, 115, 85, 0.16) 0%, transparent 60%),
    radial-gradient(ellipse at 75% 60%, rgba(35, 105, 78, 0.14) 0%, transparent 60%),
    linear-gradient(180deg, transparent 50%, rgba(2, 20, 13, 0.6) 100%);
}

/* 3. Deep Dark Green Atmospheric Overlay */
.overlay {
  position: absolute;
  inset: 0;
  z-index: -1;
  background: 
    radial-gradient(ellipse at 50% 50%, rgba(3, 26, 18, 0.68) 0%, rgba(2, 16, 11, 0.92) 100%),
    linear-gradient(180deg, rgba(3, 26, 18, 0.88) 0%, rgba(4, 30, 22, 0.74) 45%, rgba(2, 18, 12, 0.94) 100%);
}

/* TV Controls (Fullscreen toggle) */
.tv-controls {
  position: absolute;
  top: 20px;
  right: 24px;
  z-index: 10;
}

.ctrl-btn {
  background: rgba(3, 26, 18, 0.85);
  border: 1px solid rgba(166, 213, 189, 0.25);
  color: #c9e4d5;
  padding: 8px 16px;
  border-radius: 999px;
  font-family: inherit;
  font-size: 13px;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.2s ease, border-color 0.2s ease, transform 0.2s ease;
}

.ctrl-btn:hover {
  background: rgba(8, 44, 32, 0.95);
  border-color: rgba(166, 213, 189, 0.5);
  color: #ffffff;
  transform: scale(1.03);
}

/* Content Container */
.content {
  position: relative;
  width: 100%;
  max-width: 1080px;
  text-align: center;
  z-index: 2;
  animation: tvFadeIn 0.8s ease-out both;
}

@keyframes tvFadeIn {
  from {
    opacity: 0;
    transform: translateY(12px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* Location Badge */
.location-badge {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  padding: 6px 18px;
  background: rgba(4, 32, 22, 0.88);
  border: 1px solid rgba(166, 213, 189, 0.25);
  border-radius: 999px;
  margin-bottom: 24px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.35);
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
  box-shadow: 0 0 6px #4ade80;
}

.beacon-wave {
  position: absolute;
  inset: -3px;
  border-radius: 50%;
  border: 1.5px solid #4ade80;
  animation: radarWave 2.2s cubic-bezier(0.1, 0.8, 0.3, 1) infinite;
}

@keyframes radarWave {
  0% { transform: scale(0.6); opacity: 1; }
  100% { transform: scale(2.4); opacity: 0; }
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

/* Hero Typography */
.hero-title {
  font-family: 'Playfair Display', Georgia, serif;
  font-size: clamp(48px, 6.8vw, 86px);
  font-weight: 500;
  line-height: 1.15;
  letter-spacing: -1.2px;
  margin: 0 auto;
  color: #ffffff;
  text-shadow: 0 4px 20px rgba(0, 0, 0, 0.7);
}

.description {
  margin: 22px auto 0;
  font-size: clamp(15px, 1.6vw, 19px);
  line-height: 1.7;
  color: rgba(240, 246, 242, 0.94);
  max-width: 660px;
  font-weight: 400;
  text-shadow: 0 2px 12px rgba(0, 0, 0, 0.6);
}

/* TV-Optimized Countdown Cards (Zero backdrop-filter = 100% OOM Safe) */
.countdown {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 20px;
  max-width: 860px;
  margin: 52px auto 0;
}

.countdown-card {
  position: relative;
  background: rgba(3, 28, 19, 0.88);
  border: 1px solid rgba(166, 213, 189, 0.22);
  border-radius: 18px;
  min-height: 148px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  overflow: hidden;
  box-shadow: 0 14px 32px rgba(0, 0, 0, 0.65), inset 0 1px 1px rgba(255, 255, 255, 0.12);
  transition: transform 0.25s ease, background 0.25s ease, border-color 0.25s ease;
  cursor: default;
}

.countdown-card:hover {
  transform: translateY(-6px);
  background: rgba(5, 38, 26, 0.94);
  border-color: rgba(166, 213, 189, 0.5);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.8), 0 0 20px rgba(74, 222, 128, 0.18);
}

/* Number Container & TV-Smooth Tick */
.number-box {
  position: relative;
  height: clamp(52px, 5.8vw, 74px);
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  width: 100%;
}

.number {
  font-family: 'Playfair Display', Georgia, serif;
  font-size: clamp(44px, 5.2vw, 68px);
  line-height: 1;
  font-weight: 500;
  font-variant-numeric: tabular-nums;
  letter-spacing: -0.5px;
  color: #ffffff;
  text-shadow: 0 4px 16px rgba(0, 0, 0, 0.5);
  display: inline-block;
}

.card-seconds .number {
  color: #f2fff8;
  text-shadow: 0 0 18px rgba(166, 213, 189, 0.35), 0 4px 16px rgba(0, 0, 0, 0.5);
}

/* Lightweight Hardware-Accelerated Tick (Zero blur texture allocation) */
.tv-tick-enter-active {
  transition: transform 0.32s cubic-bezier(0.16, 1, 0.3, 1), opacity 0.28s ease;
}

.tv-tick-leave-active {
  transition: transform 0.24s ease-in, opacity 0.22s ease-in;
  position: absolute;
}

.tv-tick-enter-from {
  opacity: 0;
  transform: translateY(-16px);
}

.tv-tick-leave-to {
  opacity: 0;
  transform: translateY(16px);
}

/* Unit Label */
.unit {
  margin-top: 8px;
}

.unit-text {
  font-family: 'DM Sans', sans-serif;
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 2px;
  text-transform: uppercase;
  color: #a6d5bd;
}

/* Arrival Screen */
.arrived-container {
  margin: 50px auto 20px;
}

.celebration-badge {
  font-size: 12px;
  font-weight: 700;
  letter-spacing: 2.5px;
  color: #ffd875;
  margin-bottom: 14px;
}

.arrived-title {
  font-family: 'Playfair Display', Georgia, serif;
  font-size: clamp(38px, 6vw, 64px);
  font-weight: 500;
  color: #ffffff;
  margin-bottom: 12px;
  text-shadow: 0 4px 16px rgba(0, 0, 0, 0.5);
}

.arrived-sub {
  font-size: 17px;
  color: rgba(255, 255, 255, 0.9);
  max-width: 520px;
  margin: 0 auto 24px;
  line-height: 1.6;
}

.reset-preview-btn {
  background: rgba(255, 255, 255, 0.15);
  border: 1px solid rgba(255, 255, 255, 0.3);
  color: #ffffff;
  padding: 10px 22px;
  border-radius: 999px;
  font-family: inherit;
  font-size: 13px;
  cursor: pointer;
  transition: background 0.2s ease, transform 0.2s ease;
}

.reset-preview-btn:hover {
  background: rgba(255, 255, 255, 0.25);
  transform: scale(1.03);
}

/* Caption and Action Row */
.caption-wrapper {
  margin-top: 36px;
}

.countdown-caption {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-size: 14px;
  color: rgba(213, 230, 222, 0.9);
  letter-spacing: 0.4px;
}

.caption-icon {
  color: #ffd875;
  font-size: 14px;
}

.action-row {
  margin-top: 14px;
}

.preview-toggle {
  background: transparent;
  border: 1px solid rgba(166, 213, 189, 0.3);
  color: #a6d5bd;
  padding: 6px 16px;
  border-radius: 999px;
  font-family: inherit;
  font-size: 12px;
  cursor: pointer;
  letter-spacing: 0.3px;
  transition: background 0.2s ease, color 0.2s ease;
}

.preview-toggle:hover {
  background: rgba(166, 213, 189, 0.15);
  color: #ffffff;
}

/* TV & Large Screen Optimizations */
@media (min-width: 1440px) {
  .hero-title {
    font-size: 82px;
  }
  .countdown {
    max-width: 960px;
    gap: 24px;
    margin-top: 60px;
  }
  .countdown-card {
    min-height: 168px;
  }
  .number {
    font-size: 72px;
  }
  .unit-text {
    font-size: 12px;
  }
}

/* Mobile & Smaller Screens */
@media (max-width: 768px) {
  .munnar {
    padding: 50px 16px 24px;
  }
  .tv-controls {
    top: 14px;
    right: 14px;
  }
  .ctrl-btn {
    padding: 6px 12px;
    font-size: 11px;
  }
  .hero-title {
    font-size: clamp(38px, 9vw, 56px);
  }
  .description {
    font-size: 14px;
    margin-top: 16px;
  }
  .countdown {
    gap: 10px;
    margin-top: 36px;
  }
  .countdown-card {
    min-height: 104px;
    border-radius: 14px;
  }
  .number-box {
    height: 46px;
  }
  .number {
    font-size: clamp(30px, 7.5vw, 42px);
  }
  .unit-text {
    font-size: 9px;
  }
}

@media (max-width: 420px) {
  .countdown {
    gap: 8px;
  }
  .countdown-card {
    min-height: 94px;
  }
  .number {
    font-size: 26px;
  }
}

/* Accessibility: Reduced Motion */
@media (prefers-reduced-motion: reduce) {
  .beacon-wave,
  .tv-tick-enter-active,
  .tv-tick-leave-active,
  .content {
    animation: none !important;
    transition: none !important;
  }
}
</style>
