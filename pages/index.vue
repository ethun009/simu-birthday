<script setup>
import { ref, onMounted } from 'vue'

const isLoading = ref(false)
const progress = ref(0) // 0..100
const speedMbps = ref(0) // MB/s
const downloadedBytes = ref(0)
const totalBytes = ref(0)
const statusText = ref('Ready to reveal your gift')

// Love-themed hearts animation toggle
const heartsActive = ref(true)

// No manual install UI; rely on system prompt where available

// Asset list from /public/assets
const assetPaths = [
  '/assets/balloons.png',
  '/assets/cake.png',
  '/assets/dove.gif',
  '/assets/heartballoon.png',
  '/assets/letter-envelope.png',
  '/assets/table.png',
  '/assets/girl/1.jpg',
  '/assets/girl/2.jpg',
  '/assets/girl/3.jpg',
  '/assets/girl/4.jpg',
  '/assets/girl/5.jpg',
  '/assets/girl/6.jpg',
  '/assets/songs/jodiboli.mp3',
  '/assets/songs/hbd song.mp3',
  '/assets/songs/terayaarhu.mp3',
  '/assets/songs/Tum Jo Aaye.mp3',
  '/assets/voice/replyVoice.wav',
  '/assets/voice/bg song.wav',
  '/assets/voice/letter.wav',
]

onMounted(() => {
  if (process.client) {
    // No manual handling for install; allow default behavior
  }
})

function formatBytes(bytes) {
  const mb = bytes / (1024 * 1024)
  return `${mb.toFixed(2)} MB`
}

async function clearStorage() {
  try {
    localStorage.clear()
    if ('caches' in window) {
      const keys = await caches.keys()
      await Promise.all(keys.map(k => caches.delete(k)))
    }
    statusText.value = 'Storage cleared'
  } catch (e) {
    statusText.value = 'Failed to clear storage'
  }
}

// No install button/action per request

function encodedUrl(url) {
  return encodeURI(url)
}

async function startPreload() {
  if (isLoading.value) return
  isLoading.value = true
  heartsActive.value = false
  downloadedBytes.value = 0
  speedMbps.value = 0
  progress.value = 0
  statusText.value = 'Preparing download…'

  // Determine total size from Content-Length where available
  totalBytes.value = 0
  const lengths = await Promise.all(assetPaths.map(async (url) => {
    try {
      // Skip HEAD for audio where some dev servers may not support it
      const isAudio = url.endsWith('.wav') || url.endsWith('.mp3')
      if (isAudio) return 0
      const res = await fetch(encodedUrl(url), { method: 'HEAD' })
      const len = res.headers.get('content-length')
      return len ? parseInt(len, 10) : 0
    } catch {
      return 0
    }
  }))
  totalBytes.value = lengths.reduce((a, b) => a + b, 0)

  const cache = 'caches' in window ? await caches.open('assets-cache') : null
  const startTime = performance.now()

  let completedCount = 0
  for (const url of assetPaths) {
    statusText.value = `Downloading ${url.split('/').pop()}`
    try {
      const res = await fetch(encodedUrl(url))
      const contentLength = res.headers.get('content-length')
      const totalForThis = contentLength ? parseInt(contentLength, 10) : 0
      const reader = res.body ? res.body.getReader() : null
      if (!reader) {
        const blob = await res.blob()
        downloadedBytes.value += blob.size
        updateProgress(startTime)
        if (cache) await cache.put(encodedUrl(url), new Response(blob))
        completedCount++
        continue
      }
      const chunks = []
      let received = 0
      while (true) {
        const { done, value } = await reader.read()
        if (done) break
        chunks.push(value)
        received += value.byteLength
        downloadedBytes.value += value.byteLength
        updateProgress(startTime)
      }
      const blob = new Blob(chunks)
      // Store binary in Cache Storage (localStorage cannot hold large binaries reliably)
      if (cache) await cache.put(encodedUrl(url), new Response(blob))
      completedCount++
    } catch (e) {
      // Skip failed asset and continue
    }
  }

  statusText.value = 'All assets downloaded'
  progress.value = 100
  speedMbps.value = 0
  navigateTo('/gift')
}

function updateProgress(startTime) {
  const elapsedSec = (performance.now() - startTime) / 1000
  if (elapsedSec > 0) {
    speedMbps.value = downloadedBytes.value / (1024 * 1024) / elapsedSec
  }
  if (totalBytes.value > 0) {
    progress.value = Math.min(100, (downloadedBytes.value / totalBytes.value) * 100)
  } else {
    // Fallback when total size is unknown: approximate by completed assets (in-memory)
    // Note: This is updated in the preload loop using completedCount
  }
}
</script>

<template>
  <div class="love-page">
   
      <button class="top-btn clear" @click="clearStorage">Clear Storage</button>
     

    <div class="hero">
      <h1 class="title">A Little Gift, Just for You</h1>
      <p class="subtitle">Let me prepare your surprise. Tap when ready.</p>
      <button v-if="!isLoading" class="reveal-btn" @click="startPreload">
        Reveal Gift
      </button>

      <div v-else class="progress-card">
        <div class="progress-top">{{ progress.toFixed(0) }}% downloaded</div>
        <div class="progress-bar" :class="{ loading: isLoading }">
          <div class="progress-fill" :style="{ width: progress + '%' }"></div>
        </div>
        <div class="progress-bottom">{{ speedMbps.toFixed(2) }} MB/s • {{ formatBytes(downloadedBytes) }} / {{ formatBytes(totalBytes) }}</div>
        <div class="status">{{ statusText }}</div>
      </div>
    </div>

    <!-- No manual install prompt per request -->

   
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;500;700&family=Great+Vibes&display=swap');

.love-page {
  --pink-1: #ff9a9e;
  --pink-2: #fad0c4;
  --rose-1: #ffd1dc;
  --rose-2: #ffe5ec;
  --text-1: #5c2a3d;
  --white-glass: rgba(255,255,255,0.7);

  min-height: 100vh;
  width: 100vw;
  background: radial-gradient(1200px 600px at 10% 10%, var(--rose-1) 0%, var(--rose-2) 40%, #fff 100%),
              linear-gradient(135deg, var(--pink-1) 0%, var(--pink-2) 100%);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: var(--text-1);
  font-family: 'Poppins', sans-serif;
  position: relative;
  overflow: hidden;
  padding-top: var(--topbar-h);
}

.love-page::before {
  content: '';
  position: fixed;
  inset: -10% -10% -10% -10%;
  background: radial-gradient(800px 400px at 80% 20%, rgba(255, 182, 193, 0.5), transparent 60%),
              radial-gradient(600px 300px at 20% 80%, rgba(255, 215, 225, 0.45), transparent 60%),
              conic-gradient(from 180deg at 50% 50%, rgba(255, 255, 255, 0.4), rgba(255, 255, 255, 0) 25%, rgba(255, 255, 255, 0.4) 50%, rgba(255, 255, 255, 0) 75%, rgba(255, 255, 255, 0.4));
  filter: blur(32px);
  opacity: 0.5;
  animation: glowShift 18s linear infinite;
  pointer-events: none;
}



.top-btn {
  border: none;
  position: fixed;
  padding: 10px 16px;
  border-radius: 22px;
  font-weight: 600;
  cursor: pointer;
  top: 10px;
  right: 10%;
}
.top-btn.clear {
  background: rgba(255,255,255,0.18);
  color: #b0003a;
  border: 1px solid rgba(255, 209, 220, 0.9);
  backdrop-filter: blur(6px) saturate(120%);
  box-shadow: 0 4px 12px rgba(240, 155, 180, 0.22), inset 0 0 0 1px rgba(255,255,255,0.25);
  transition: transform 0.15s ease, box-shadow 0.2s ease, background 0.2s ease;
}
.top-btn.clear:hover {
  background: rgba(255,255,255,0.28);
  transform: translateY(-1px);
  box-shadow: 0 6px 16px rgba(240, 155, 180, 0.28), inset 0 0 0 1px rgba(255,255,255,0.35);
}
.top-btn.install { background: #ff69b4; color: #fff; }
.top-btn.install:disabled { opacity: 0.6; cursor: not-allowed; }

.spacer { flex: 1; }

.hero {
      margin-top: 0;
    text-align: center;
    padding: 0 20px;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    place-items: center;
}

.title {
  font-family: 'Great Vibes', cursive;
  font-size: clamp(2.6rem, 6vw, 4rem);
  margin-bottom: 6px;
  background: linear-gradient(90deg, #ff4d8c, #ff8fb3);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  text-shadow: 0 2px 10px rgba(255, 120, 160, 0.2);
  animation: titleFloat 6s ease-in-out infinite;
}

.subtitle {
  font-size: 1.05rem;
  opacity: 0.9;
  margin-bottom: 24px;
  letter-spacing: 0.2px;
  animation: fadeIn 800ms ease both;
}

.reveal-btn {
  background: linear-gradient(135deg, #ff4d8c, #ff7fb1, #ff4d8c);
  background-size: 200% 200%;
  color: #fff;
  border: none;
  padding: 14px 28px;
  border-radius: 28px;
  font-weight: 700;
  box-shadow: 0 8px 22px rgba(255, 91, 167, 0.35);
  cursor: pointer;
  transition: transform 0.1s ease, box-shadow 0.2s ease, filter 0.2s ease;
  position: relative;
  animation: gradientShift 6s ease infinite, pulse 2.8s ease-in-out infinite;
}
.reveal-btn:hover { transform: translateY(-1px); }
.reveal-btn:active { transform: translateY(0); box-shadow: 0 4px 12px rgba(255, 91, 167, 0.25); }
.reveal-btn::after {
  content: '';
  position: absolute;
  inset: 0;
  border-radius: inherit;
  background: radial-gradient(120% 120% at 10% 10%, rgba(255,255,255,0.35), transparent 40%);
  opacity: 0.7;
  pointer-events: none;
}

.progress-card {
  background: rgba(255, 255, 255, 0.85);
  border: 1px solid #ffd1dc;
  border-radius: 16px;
  padding: 18px;
  width: min(520px, 92vw);
  margin: 10px auto;
  box-shadow: 0 14px 34px rgba(240, 155, 180, 0.28);
  backdrop-filter: blur(6px);
}

.progress-top,
.progress-bottom {
  font-size: 0.95rem;
  font-weight: 600;
}

.progress-bar {
  height: 16px;
  background: #ffe5ec;
  border: 1px solid #ffd1dc;
  border-radius: 999px;
  overflow: hidden;
  margin: 10px 0;
  position: relative;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #ff69b4, #ffa3c7);
  width: 0%;
  transition: width 0.18s ease;
}

.progress-bar.loading::after {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(90deg, rgba(255,255,255,0), rgba(255,255,255,0.6), rgba(255,255,255,0));
  transform: translateX(-100%);
  animation: shine 1.4s ease-in-out infinite;
}

.status {
  margin-top: 6px;
  font-size: 0.9rem;
  opacity: 0.85;
}

.hearts {
  position: fixed;
  pointer-events: none;
  bottom: 8vh;
  display: grid;
  grid-template-columns: repeat(6, minmax(20px, 1fr));
  gap: 8px;
  width: 80vw;
}

.heart {
  color: #ff3e88;
  opacity: 0.7;
  animation: float 6s ease-in-out infinite;
  filter: drop-shadow(0 2px 4px rgba(255, 62, 136, 0.25));
}

@keyframes float {
  0% { transform: translateY(0) scale(1); opacity: 0.7; }
  50% { transform: translateY(-12px) scale(1.1); opacity: 1; }
  100% { transform: translateY(0) scale(1); opacity: 0.7; }
}

@keyframes glowShift {
  0% { transform: rotate(0deg); }
  50% { transform: rotate(180deg); }
  100% { transform: rotate(360deg); }
}

@keyframes titleFloat {
  0% { transform: translateY(0); }
  50% { transform: translateY(-6px); }
  100% { transform: translateY(0); }
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(4px); }
  to { opacity: 1; transform: translateY(0); }
}

@keyframes shine {
  from { transform: translateX(-100%); }
  to { transform: translateX(100%); }
}

@keyframes gradientShift {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

@keyframes pulse {
  0% { transform: translateY(0) scale(1); }
  50% { transform: translateY(-1px) scale(1.02); }
  100% { transform: translateY(0) scale(1); }
}
</style>