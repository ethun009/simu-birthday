<template>
  <div class="lyrics-container" ref="lyricsContainer">
    <h3 class="lyrics-title">Lyrics</h3>
    <div class="lyrics-content">
      <transition name="fade-page" mode="out-in">
        <div 
          :key="`page-${currentPageIndex}`" 
          class="lyrics-page"
        >
          <p 
            v-for="(line, index) in currentLyricsPage" 
            :key="`line-${index}`" 
            class="lyrics-line"
            :class="{ 'active-line': index === activeLineIndex, 'paused': !isPlaying }"
          >
            {{ line }}
          </p>
        </div>
      </transition>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount, watch, nextTick } from 'vue';

// Timed lyrics for each song with timestamps in seconds
const timedLyrics = {
 1: [
  { time: 0.00, text: "....✨" },
  { time: 1.96, text: "যদি বলি, \"হ্যাঁ, হাসছি আমি শুধুই তুমি আমার তাই\" 😊💖" },
  { time: 7.42, text: "বলো, ছেড়ে তো দেবে না কখনও মনের ভুলে 👉👈🥺" },
  { time: 13.01, text: "গোধূলি আকাশ মুছে দিল সাজ 🌆✨" },
  { time: 15.50, text: "অযথা দূরে তবু তুমি আজ 😔💭" },
  { time: 18.44, text: "অভিমানী ভুল ধরবে আঙ্গুল 👉👈💗" },
  { time: 21.39, text: "মন করে বায়না 😌💕" },
  { time: 24.22, text: "তুমি কি আমায় করবে পাগল 😵💘" },
  { time: 27.12, text: "শাড়ির আঁচল, চোখের কাজল 👀💄✨" },
  { time: 29.93, text: "প্রেমে তুমিও পড়ে যাবে, হায় 💞😌" },
  { time: 32.86, text: "দেখো যদি আয়না 🪞💫" },
  { time: 35.85, text: "বাঁচি এই বিশ্বাসে 💖🙏" },
  { time: 38.14, text: "শেষ নিঃশ্বাসে তোমাকেই পাশে চাই 🤍🫂" },
  { time: 41.92, text: "তুমি না থাকলে আমি শূন্য এ মহাদেশে 🌍💔" },
  { time: 47.85, text: "যদি ঘুমিয়েও পড়ি 😴✨" },
  { time: 49.61, text: "শেষ ঘুমে আমি তবুও তোমাকে চাই 🤍🥺" },
  { time: 53.66, text: "তুমি স্বপ্নেই এসো রূপকথার ওই দেশে 🌙✨🏰" },
  { time: 82.78, text: "আমি বুকের মাঝে জাপ্টে জড়িয়ে 🤗❤️" },
  { time: 85.64, text: "যত কথা আছে সবই তোমাকেই বলি 💬💞" },
  { time: 91.21, text: "আমি কান পেতে সেই মনের গভীরে 🎧💗" },
  { time: 94.35, text: "লুকোনো যন্ত্রণা শুনে ফেলি 😔❤️‍🩹" },
  { time: 102.90, text: "তুমি অভিমানে খুব হয়ে গেলে চুপ 😶💔" },
  { time: 106.08, text: "ভুল মেনে নিয়ে কত কত \"sorry\" বলি 😅💗" },
  { time: 111.35, text: "ভাবি থাকবোই রেগে গম্ভীর মুখে 😑" },
  { time: 114.72, text: "তোমার কোথায় ধুত আমি হেসে ফেলি! 😂💞" },
  { time: 123.22, text: "মারপিট আর ঝগড়াঝাঁটি-রা শান্তি চাইবে শেষে 😤➡️🤝" },
  { time: 128.47, text: "তাই অভিমান ভুলে আদর মাখতে তোমার কাছে এসে 🥺🤗💗" },
  { time: 137.22, text: "যদি বলি, \"আমার প্রতিটা রাত তোমার কোলে চাই\" 😴🫂💖" },
  { time: 142.91, text: "বলো, ঠোঁটের ছোঁয়ায় আদর মাখাবে গালে 😘🤍" },
  { time: 148.79, text: "যদি বলি, \"হ্যাঁ, হাসছি আমি শুধুই তুমি আমার তাই\" 😊💖" },
  { time: 154.19, text: "বলো, ছেড়ে তো দেবে না কখনও মনের ভুলে 👉👈🥺" },
  { time: 160.21, text: "বাঁচি এই বিশ্বাসে 💖🙏" },
  { time: 161.99, text: "শেষ নিঃশ্বাসে তোমাকেই পাশে চাই 🤍🫂" },
  { time: 165.94, text: "তুমি না থাকলে আমি শূন্য এ মহাদেশে 🌍💔" },
  { time: 171.89, text: "যদি ঘুমিয়েও পড়ি 😴✨" },
  { time: 173.59, text: "শেষ ঘুমে আমি তবুও তোমাকে চাই 🤍🥺" },
  { time: 177.05, text: "তুমি স্বপ্নেই এসো রূপকথার ওই দেশে 🌙🏰✨" }
]
,
  2: [
  { time: 0.00, text: "...........✨" },
  { time: 23.65, text: "Tu jo rootha toh kaun hansega 😔💔" },
  { time: 29.14, text: "Tu jo chhoota toh kaun rahega 🥺💞" },
  { time: 34.84, text: "Tu chup hai toh yeh darr lagta hai 😶💭💗" },
  { time: 40.53, text: "Apna mujhko ab kaun kahega 🥹🤍" },
  { time: 46.20, text: "Tu hi wajah… Tere bina 💖✨" },
  { time: 51.91, text: "Bewajah bekar hoon main 😔💭" },
  { time: 58.56, text: "Tera yaar hoon main 🤝❤️" },
  { time: 64.24, text: "Tera yaar hoon main 🤗💖" },
  { time: 100.28, text: "Aaja ladein phir khilono ke liye 🧸😄" },
  { time: 105.17, text: "Tu jeete main haar jaaun 😌👉👈" },
  { time: 111.44, text: "Aaja karein phir wohi shararatein 😄🔥" },
  { time: 116.62, text: "Tu bhaage main maar khaaun 😅✨" },
  { time: 123.72, text: "Meethi si woh... Gaali teri 😄💗" },
  { time: 129.58, text: "Sun’ne ko taiyaar hoon main 😊❤️" },
  { time: 135.60, text: "Tera yaar hoon main 🤝💞" },
  { time: 141.88, text: "Tera yaar hoon main 🫂💖" }
]
,
  3: [
    { time: 0, text: "music 🎵🎶🎼" },
    { time: 9.94, text: "Paaya maine, paaya tumhe, rab ne milaya tumhe" },
    { time: 12.70, text: "Honthon pe sajaya tumhe, nagme sagaya tumhe" },
    { time: 15.05, text: "Paaya maine, paaya tumhe, sab se chupaya tumhe" },
    { time: 18.52, text: "Sapna banaya tumhe, neendon mein bulaya tumhe" },
    { time: 20.39, text: "Tum jo aaye zindagi mein baat ban gayi" },
    { time: 27.23, text: "Ishq mazhab, ishq meri zaat ban gayi" },
    { time: 32.85, text: "Paaya maine, paaya tumhe, rab ne milaya tumhe" },
    { time: 35.41, text: "Honthon pe sajaya tumhe, nagme sagaya tumhe" },
    { time: 37.90, text: "Paaya maine, paaya tumhe, sab se chupaya tumhe" },
    { time: 41.73, text: "Sapna banaya tumhe, neendon mein bulaya tumhe" },
    { time: 44.41, text: "Ho tum jo aaye zindagi mein baat ban gayi" },
    { time: 50.01, text: "Sapne teri chahaton ke, sapne teri chahaton ke" },
    { time: 55.75, text: "Dekhti hoon ab kayee" },
    { time: 58.66, text: "Din hai sona aur chaandi raat ban gayi" },
    { time: 63.41, text: "Hoo tum jo aaye zindagi mein baat ban gayi" },
    { time: 69.98, text: "Paaya maine, paaya tumhe, rab ne milaya tumhe" },
    { time: 72.46, text: "Honthon pe sajaya tumhe, nagme sagaya tumhe" },
    { time: 75.33, text: "Paaya maine, paaya tumhe, sab se chupaya tumhe" },
    { time: 78.34, text: "Sapna banaya tumhe, neendon mein bulaya tumhe" },
    { time: 110.63, text: "Chahaton ka mazaa, faasalon mein nahin" },
    { time: 116.31, text: "Aa chupa loon tumhe hauslon mein kahin" },
    { time: 122.02, text: "Sab se upar likha, hai tere naam ko" },
    { time: 127.63, text: "Khwaishon se jude silsilon mein kahin" },
    { time: 133.35, text: "Khwaishein milne ki tumse, khwaishein milne ki tumse" },
    { time: 138.72, text: "Roz hoti hai nayi" },
    { time: 141.51, text: "Mere dil ki jeet meri maat ban gayi" },
    { time: 146.35, text: "Hoo tum jo aaye zindagi mein baat ban gayi" },
    { time: 175.75, text: "Paaya maine, paaya tumhe, rab ne milaya tumhe" },
    { time: 178.60, text: "Honthon pe sajaya tumhe, nagme sagaya tumhe" },
    { time: 181.13, text: "Paaya maine, paaya tumhe, sab se chupaya tumhe" },
    { time: 183.98, text: "Sapna banaya tumhe, neendon mein bulaya tumhe" },
    { time: 187.67, text: "Zindagi bewaafa hai yeh maana magar" },
    { time: 193.14, text: "Chod kar raah mein jaoge tum agar" },
    { time: 199.13, text: "Cheen launga main aasman se tumhe" },
    { time: 204.68, text: "Soona hoga na yeh, do dilon ka nagar" },
    { time: 210.39, text: "Ronke hain dil ke dar pe, ronke hain dil ke dar pe" },
    { time: 215.93, text: "Dhadkane hain surmayi" },
    { time: 218.55, text: "Meri kismat bhi tumhari, saath ban gayi" },
    { time: 223.54, text: "Hoo tum jo aaye zindagi mein baat ban gayi" },
    { time: 230.05, text: "Ishq mazhab, ishq meri zaat ban gayi" },
    { time: 235.72, text: "Sapne teri chahaton ke, sapne teri chahaton ke" },
    { time: 241.50, text: "Dekhti hoon ab kayee" },
    { time: 244.29, text: "Din hai sona aur chaandi raat ban gayi" },
    { time: 249.15, text: "Hoo tum jo aaye zindagi mein baat ban gayi" },
    { time: 255.83, text: "Paaya maine, paaya tumhe, rab ne milaya tumhe" },
    { time: 258.52, text: "Honthon pe sajaya tumhe, nagme sagaya tumhe" },
    { time: 261.11, text: "Paaya maine, paaya tumhe, sab se chupaya tumhe" },
    { time: 264.02, text: "Sapna banaya tumhe, neendon mein bulaya tumhe" },
    { time: 266.47, text: "Paaya maine, paaya tumhe, rab ne milaya tumhe" },
    { time: 269.62, text: "Honthon pe sajaya tumhe, nagme sagaya tumhe" },
    { time: 272.53, text: "Paaya maine, paaya tumhe, sab se chupaya tumhe" },
    { time: 275.39, text: "Sapna banaya tumhe, neendon mein bulaya tumhe..." }
  ]
};

// Props to receive from parent (MusicPlayer)
const props = defineProps({
  currentSongId: {
    type: Number,
    default: 1
  },
  isPlaying: {
    type: Boolean,
    default: false
  },
  currentTime: {
    type: Number,
    default: 0
  }
});

// Emits to communicate with parent
const emit = defineEmits(['update:currentTime']);

const lyricsContainer = ref(null);
const linesPerPage = ref(4); // Fixed to display exactly 4 lines per page
const activeLineIndex = ref(0);
const currentPageIndex = ref(0);
let pageTransitionInProgress = ref(false);

// Get current lyrics based on song ID
const currentLyrics = computed(() => {
  return timedLyrics[props.currentSongId] || [];
});

// Extract just the text for display
const currentLyricsText = computed(() => {
  return currentLyrics.value.map(line => line.text);
});

// Get the current page of lyrics based on page index
const currentLyricsPage = computed(() => {
  const startIndex = currentPageIndex.value * linesPerPage.value;
  return currentLyricsText.value.slice(startIndex, startIndex + linesPerPage.value);
});

// Check if we're on the last page
const isLastPage = computed(() => {
  return (currentPageIndex.value + 1) * linesPerPage.value >= currentLyricsText.value.length;
});

// Find the active line based on current playback time
const findActiveLine = (time) => {
  let activeIndex = 0;
  
  // Find the current active line based on time
  for (let i = 0; i < currentLyrics.value.length; i++) {
    const currentLineTime = currentLyrics.value[i].time;
    
    // If this line's time is less than or equal to current playback time
    // AND is greater than previous active line's time, it becomes the active line
    if (time >= currentLineTime) {
      // If we're at the last line or the next line's time is greater than current time
      if (i === currentLyrics.value.length - 1 || 
          (i + 1 < currentLyrics.value.length && time < currentLyrics.value[i + 1].time)) {
        activeIndex = i;
        // We found our line, no need to continue
        break;
      }
    }
  }
  
  // Calculate which page this line is on
  const newPageIndex = Math.floor(activeIndex / linesPerPage.value);
  
  // Always update the active line index first
  activeLineIndex.value = activeIndex % linesPerPage.value;
  
  // If we need to change page and not in transition
  if (newPageIndex !== currentPageIndex.value && !pageTransitionInProgress.value) {
    pageTransitionInProgress.value = true;
    currentPageIndex.value = newPageIndex;
    
    // Reset transition flag after animation completes
    setTimeout(() => {
      pageTransitionInProgress.value = false;
    }, 600);
  }
};

// Using a fixed value of 4 lines per page

// Reset display when song changes
const resetLyricsDisplay = () => {
  currentPageIndex.value = 0;
  activeLineIndex.value = 0;
  pageTransitionInProgress.value = false;
  
  // Force immediate update of active line based on current time
  // This ensures the correct line is highlighted right after song change
  nextTick(() => {
    findActiveLine(props.currentTime);
  });
};

// Watch for song changes
watch(() => props.currentSongId, () => {
  resetLyricsDisplay();
});

// Watch for playback time changes
watch(() => props.currentTime, (newTime) => {
  // Always check for active line, even when paused
  // This ensures lyrics update properly when seeking or changing songs
  findActiveLine(newTime);
});

// Watch for play/pause state changes
watch(() => props.isPlaying, (isPlaying) => {
  if (!isPlaying) {
    // We don't need to do anything special on pause
    // The lyrics will stay at current position
  }
});

// Handle window resize
const handleResize = () => {
  // No need to recalculate lines per page as it's fixed to 4
  // Just recalculate active line after resize
  findActiveLine(props.currentTime);
};

onMounted(() => {
  // Initialize with first line active
  activeLineIndex.value = 0;
  
  window.addEventListener('resize', handleResize);
});

onBeforeUnmount(() => {
  window.removeEventListener('resize', handleResize);
});
</script>

<style scoped>
.lyrics-container {
  width: 100%;
  height: 100%;
  padding: 15px;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  border-radius: 15px;
  box-shadow: 0 8px 20px rgba(156, 107, 148, 0.2);
  border: 1px solid rgba(255, 255, 255, 0.1);
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.lyrics-title {
  font-size: 1.2rem;
  color: #ff6b95;
  margin-top: 0;
  margin-bottom: 15px;
  text-align: center;
  font-weight: 600;
  letter-spacing: 1px;
  text-shadow: 0 0 10px rgba(255, 107, 149, 0.3);
}

.lyrics-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  flex-grow: 1;
  position: relative;
  overflow: hidden;
}

.lyrics-page {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.lyrics-line {
  margin: 5px 0;
  color: #9c6b94;
  font-size: 1rem;
  line-height: 1.5;
  text-align: center;
  width: 100%;
  transition: all 0.5s ease;
  opacity: 0.7;
}

.lyrics-line.active-line {
  color: #ff6b95;
  font-weight: 500;
  opacity: 1;
  transform: scale(1.05);
  text-shadow: 0 0 15px rgba(255, 107, 149, 0.7);
  animation: glow 1.5s infinite alternate;
}

.lyrics-line.active-line.paused {
  animation-play-state: paused;
}

.lyrics-line:empty {
  height: 15px;
}

/* Page transition - smoother with mode="out-in" */
.fade-page-enter-active,
.fade-page-leave-active {
  transition: all 0.5s ease;
}

.fade-page-enter-from {
  opacity: 0;
  transform: translateY(20px);
}

.fade-page-leave-to {
  opacity: 0;
  transform: translateY(-20px);
}

@keyframes glow {
  from {
    text-shadow: 0 0 10px rgba(255, 107, 149, 0.5);
  }
  to {
    text-shadow: 0 0 20px rgba(255, 107, 149, 0.9), 0 0 30px rgba(255, 107, 149, 0.5);
  }
}

@media (max-width: 768px) {
  .lyrics-title {
    font-size: 1.1rem;
  }
  
  .lyrics-line {
    font-size: 0.9rem;
  }
}
</style>