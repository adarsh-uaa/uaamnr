<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const target = new Date('2026-11-04T06:00:00+05:30').getTime()
const values = ref({ days: '00', hours: '00', minutes: '00', seconds: '00' })
const finished = ref(false)
let interval

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
  if (finished.value) clearInterval(interval)
}
onMounted(() => { updateCountdown(); interval = setInterval(updateCountdown, 1000) })
onUnmounted(() => clearInterval(interval))
</script>

<template>
  <main class="munnar">
    <div class="backdrop" aria-hidden="true"></div>
    <div class="overlay" aria-hidden="true"></div>
    <section class="content">
      <div class="location">KERALA, INDIA</div>
      <h1>Munnar is waiting</h1>
      <p class="description">The hills are calling. Mist, tea gardens and cool mountain air — see you at sunrise.</p>
      <div v-if="!finished" class="countdown" aria-label="Time remaining to November 4, 2026 at 6 AM IST">
        <div v-for="(value, label) in values" :key="label" class="countdown-card">
          <div class="number">{{ value }}</div>
          <div class="unit">{{ label }}</div>
        </div>
      </div>
      <div v-else class="arrived">The wait is over. Welcome to Munnar.</div>
      <p class="countdown-caption">Countdown to 4 November, 6:00 AM IST</p>
    </section>
  </main>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:wght@400;500;600;700&family=Playfair+Display:wght@400;500;600&display=swap');
* { margin: 0; padding: 0; box-sizing: border-box; }
html, body, #app { width: 100%; min-height: 100%; }
body { background: #0d3025; font-family: 'DM Sans', sans-serif; }
.munnar { position: relative; isolation: isolate; min-height: 100vh; min-height: 100dvh; display: flex; justify-content: center; align-items: center; padding: 65px 24px; overflow: hidden; color: #f8f5ed; }
.backdrop { position: absolute; inset: 0; z-index: -2; background: url('/munnar.jpg') center 55% / cover no-repeat; }
.overlay { position: absolute; inset: 0; z-index: -1; background: linear-gradient(180deg, rgba(9,43,36,.8) 0%, rgba(5,39,28,.78) 45%, rgba(5,34,23,.74) 100%); }
.content { width: 100%; max-width: 1050px; text-align: center; animation: fadeIn 1s ease-out both; }
.location { color: #a6d5bd; font-size: 13px; font-weight: 700; letter-spacing: 1.2px; margin-bottom: 30px; }
h1 { font-family: 'Playfair Display', Georgia, serif; font-size: clamp(48px, 7vw, 88px); font-weight: 400; line-height: 1.2; letter-spacing: -1.5px; }
.description { margin: 32px auto 0; font-size: clamp(14px, 1.55vw, 18px); line-height: 1.8; color: rgba(255,255,255,.9); }
.countdown { display: grid; grid-template-columns: repeat(4, minmax(0,1fr)); gap: 16px; max-width: 815px; margin: 80px auto 0; }
.countdown-card { background: rgba(4,36,25,.38); border: 1px solid rgba(255,255,255,.025); border-radius: 17px; min-height: 134px; display: flex; flex-direction: column; justify-content: center; align-items: center; backdrop-filter: blur(5px); -webkit-backdrop-filter: blur(5px); }
.number { font-family: 'Playfair Display', Georgia, serif; font-size: clamp(42px,5vw,61px); line-height: 1.15; font-weight: 400; font-variant-numeric: tabular-nums; }
.unit { margin-top: 8px; font-family: 'Playfair Display', Georgia, serif; font-size: 11px; text-transform: uppercase; color: #d5e6d8; }
.countdown-caption { margin-top: 34px; font-size: 14px; color: rgba(255,255,255,.84); }
.arrived { margin: 80px auto 0; font-family: 'Playfair Display', Georgia, serif; font-size: clamp(28px,4vw,48px); }
@keyframes fadeIn { from { opacity: 0; transform: translateY(16px); } to { opacity: 1; transform: translateY(0); } }
@media (max-width: 650px) { .munnar { padding: 55px 18px; } .location { font-size: 11px; margin-bottom: 23px; } h1 { font-size: clamp(44px,10vw,65px); letter-spacing: -1px; } .description { font-size: 14px; margin-top: 22px; max-width: 340px; } .countdown { gap: 9px; margin-top: 48px; } .countdown-card { min-height: 100px; border-radius: 12px; } .number { font-size: clamp(31px,8vw,46px); } .unit { font-size: 9px; margin-top: 6px; } .countdown-caption { margin-top: 30px; font-size: 12px; } }
@media (prefers-reduced-motion: reduce) { .content { animation: none; } }
</style>
