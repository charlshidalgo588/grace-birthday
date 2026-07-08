<script setup lang="ts">
import { ref, reactive, computed, onMounted, onUnmounted } from 'vue'

interface Photo {
  src: string
  caption: string
}

interface Milestone {
  date: string
  title: string
  text: string
}

/* ============================================================
   ✏️  EDIT EVERYTHING IN THIS BLOCK TO PERSONALIZE THE SITE
   ============================================================ */

const herName = 'Sweetheart' // ✏️ her name
const yourName = 'Me' // ✏️ your name

// ✏️ her birthday — used for the live countdown. Update the year if you reuse this next year.
const herBirthday = '2026-08-20'

const message = [
  `My dearest ${herName},`,
  `Before you is a small ritual — a seal to break, a card to open, and, waiting on the other side, everything I've been wanting to tell you.`,
  `[✏️ Edit this paragraph — write about a specific memory. The night you met, a trip you took, an ordinary Tuesday that turned out to matter.]`,
  `[✏️ Edit this paragraph — write about what you love about her, in your own words. Be specific: a habit, a laugh, a way she sees the world.]`,
  `Today is about celebrating you — all of you. I hope this year holds everything you deserve, and I hope I get to watch it happen right beside you.`,
  `Happy Birthday. I love you more than this little letter can hold.`,
  `Forever yours, ${yourName}`,
]

// ✏️ Be specific here — real, small details beat grand statements.
const reasons = [
  'The way you laugh at your own jokes before you finish telling them.',
  'How you remember the small things I mention once and forget I said.',
  'The way you make any place feel like home within five minutes.',
  '[✏️ another true, specific reason]',
  '[✏️ another true, specific reason]',
  '[✏️ another true, specific reason]',
]

// ✏️ src paths: whatever the file's location is inside `public/`, use that
//    same path here with a leading slash, e.g. public/photos/1.jpg -> "/photos/1.jpg"
const photos = reactive<Photo[]>([
  { src: '/a-dark-knight-5c.jpg', caption: '✏️ add photo 1' },
  { src: '', caption: '✏️ add photo 2' },
  { src: '', caption: '✏️ add photo 3' },
  { src: '', caption: '✏️ add photo 4' },
  { src: '', caption: '✏️ add photo 5' },
  { src: '', caption: '✏️ add photo 6' },
])

// ✏️ one main video — e.g. src: "/videos/main.mp4"
//    poster is an optional thumbnail image shown before she presses play
const mainVideo = reactive({
  src: 'public/vids/moonlight-dark-knight-batman-moewalls-com.mp4',
  poster: '',
  caption: '✏️ edit this caption — e.g. "a little something I put together"',
})

const musicUrl = '' // ✏️ optional, e.g. "/music/song.mp3" — leave blank to hide the button

// ✏️ the wish message revealed after she blows out the candle
const wishText = `Whatever you wished for, I hope it finds you exactly when you need it. And if nothing crossed your mind — I'm already wishing it for you.`

// ✏️ your love story, in a few chapters — as many or as few as you like
const milestones = reactive<Milestone[]>([
  { date: '[✏️ month, year]', title: '[✏️ "How We Met"]', text: '[✏️ a line or two about it]' },
  {
    date: '[✏️ month, year]',
    title: '[✏️ "Our First Trip"]',
    text: '[✏️ a line or two about it]',
  },
  {
    date: '[✏️ month, year]',
    title: '[✏️ "A Chapter That Mattered"]',
    text: '[✏️ a line or two about it]',
  },
  { date: 'Today', title: 'Right Now', text: `And here we are, celebrating you, ${herName}.` },
])

/* ============================================================
   🌹 Rose bouquet — a hand-built SVG bouquet of real roses.
   Each rose is 3 rings of curved petals (dark outer → light inner)
   around a tiny center bud, on a stem with leaves, wrapped in paper
   with a ribbon bow. Tweak positions/colors below if you like.
   ============================================================ */

// single-petal shapes, reused (rotated) around each rose's center
const outerPetalPath = 'M0,0 C-13,-8 -15,-22 -5,-31 C-2,-33 2,-33 5,-31 C15,-22 13,-8 0,0 Z'
const midPetalPath = 'M0,0 C-9,-6 -10,-16 -3,-23 C-1,-25 1,-25 3,-23 C10,-16 9,-6 0,0 Z'
const innerPetalPath = 'M0,0 C-6,-4 -6,-10 -2,-14 C-1,-15 1,-15 2,-14 C6,-10 6,-4 0,0 Z'
const centerBudPath =
  'M0,-2 C-3,-4 -3,-7 0,-9 C2,-9.5 3,-8.5 2.5,-7 C2,-5.5 0,-4.5 -1,-3.5 C-1,-2.5 0,-2 0,-2 Z'
const leafPath = 'M0,0 C10,-11 27,-11 35,0 C27,11 10,11 0,0 Z'

const roses = [
  {
    x: 150,
    y: 92,
    scale: 1.18,
    rotate: -3,
    outer: 'var(--wine)',
    mid: 'var(--wine-soft)',
    inner: 'var(--gold-light)',
    stem: 'M150,300 C150,250 150,178 150,118',
  },
  {
    x: 97,
    y: 124,
    scale: 0.92,
    rotate: -16,
    outer: 'var(--wine-soft)',
    mid: 'var(--pink-mid)',
    inner: 'var(--blush)',
    stem: 'M150,300 C128,260 108,198 97,150',
  },
  {
    x: 203,
    y: 122,
    scale: 0.92,
    rotate: 16,
    outer: 'var(--wine-soft)',
    mid: 'var(--pink-mid)',
    inner: 'var(--blush)',
    stem: 'M150,300 C172,260 192,196 203,148',
  },
  {
    x: 64,
    y: 160,
    scale: 0.7,
    rotate: -30,
    outer: 'var(--pink-mid)',
    mid: 'var(--blush)',
    inner: '#fff2f6',
    stem: 'M150,300 C108,270 80,222 64,184',
  },
  {
    x: 236,
    y: 158,
    scale: 0.7,
    rotate: 28,
    outer: 'var(--pink-mid)',
    mid: 'var(--blush)',
    inner: '#fff2f6',
    stem: 'M150,300 C192,270 220,220 236,182',
  },
]

const leaves = [
  { x: 150, y: 232, rotate: -22, scale: 1.15, flip: false },
  { x: 150, y: 266, rotate: 26, scale: 1.05, flip: true },
  { x: 118, y: 202, rotate: -55, scale: 0.85, flip: false },
  { x: 186, y: 200, rotate: 55, scale: 0.85, flip: true },
]

/* ============================================================
   State + logic — no need to touch anything below this line
   ============================================================ */

const sealBroken = ref(false)
const envelopeOpen = ref(false)
const bookOpen = ref(false)
const showBook = ref(false)
const lightboxPhoto = ref<Photo | null>(null)
const videoPlaying = ref(false)
const musicPlaying = ref(false)
const audio = ref<HTMLAudioElement | null>(null)

// ---- bouquet reveal (shown after she reads the letter) ----
const bouquetRevealed = ref(false)
function revealBouquet() {
  if (bouquetRevealed.value) return
  bouquetRevealed.value = true
  burstPetals(46)
}

const tilts = [-6, 4, -3, 7, -5, 3, 6, -4]
const hearts = Array.from({ length: 16 }, (_, i) => ({
  id: i,
  left: (i * 6.3) % 100,
  dur: 10 + (i % 5) * 3,
  delay: (i % 8) * 1.4,
  size: 0.8 + (i % 4) * 0.25,
  glyph: i % 5 === 0 ? '🌹' : '♥',
}))

const firstLetter = computed(() => message[0]?.charAt(0) ?? '')
const firstRest = computed(() => message[0]?.slice(1) ?? '')

// ---- shared confetti/petal burst (used by seal + candle + bouquet reveal) ----
const petals = ref<
  { id: string; x: number; rot: number; delay: string; hue: number; top: string }[]
>([])
function burstPetals(topVh = 42) {
  const arr = []
  for (let i = 0; i < 14; i++) {
    arr.push({
      id: `${Date.now()}-${i}-${Math.random()}`,
      x: Math.round(Math.random() * 220 - 110),
      rot: Math.round(Math.random() * 360),
      delay: (Math.random() * 0.2).toFixed(2),
      hue: i % 2,
      top: topVh + 'vh',
    })
  }
  petals.value = arr
  setTimeout(() => {
    petals.value = []
  }, 1500)
}

function breakSeal() {
  if (sealBroken.value) return
  sealBroken.value = true
  burstPetals(42)
  setTimeout(() => {
    envelopeOpen.value = true
  }, 350)
}

function openBook() {
  showBook.value = true
}

function closeBook() {
  showBook.value = false
  bookOpen.value = false
  bouquetRevealed.value = false
}

function toggleMusic() {
  if (!audio.value) return
  if (musicPlaying.value) {
    audio.value.pause()
  } else {
    audio.value.play()
  }
  musicPlaying.value = !musicPlaying.value
}

function playMainVideo() {
  videoPlaying.value = true
}

// ---- reasons flip cards ----
const flipped = reactive(new Set<number>())
function toggleReason(i: number) {
  if (flipped.has(i)) flipped.delete(i)
  else flipped.add(i)
}

// ---- countdown to her birthday ----
const birthday = reactive({ days: 0, hours: 0, minutes: 0, seconds: 0, isToday: false })

let timer: ReturnType<typeof setInterval> | undefined

function updateBirthdayCountdown() {
  const target = new Date(herBirthday + 'T00:00:00').getTime()
  let diff = target - Date.now()
  const day = 86400000,
    hour = 3600000,
    minute = 60000
  birthday.isToday = diff <= 0 && diff > -day
  if (diff < 0) diff = 0
  birthday.days = Math.floor(diff / day)
  diff -= birthday.days * day
  birthday.hours = Math.floor(diff / hour)
  diff -= birthday.hours * hour
  birthday.minutes = Math.floor(diff / minute)
  diff -= birthday.minutes * minute
  birthday.seconds = Math.floor(diff / 1000)
}

// ---- candle wish ----
const candleLit = ref(true)
const wishGranted = ref(false)
function blowCandle() {
  if (!candleLit.value) return
  candleLit.value = false
  burstPetals(48)
  setTimeout(() => {
    wishGranted.value = true
  }, 700)
}

// ---- cursor sparkle trail (desktop only) ----
const sparkles = ref<{ id: number; x: number; y: number }[]>([])
let lastMove = 0
function handleMouseMove(e: MouseEvent) {
  const now = performance.now()
  if (now - lastMove < 60) return
  lastMove = now
  const id = now + Math.random()
  sparkles.value.push({ id, x: e.clientX, y: e.clientY })
  setTimeout(() => {
    sparkles.value = sparkles.value.filter((s) => s.id !== id)
  }, 700)
}

onMounted(() => {
  updateBirthdayCountdown()
  timer = setInterval(() => {
    updateBirthdayCountdown()
  }, 1000)
  if (window.matchMedia('(pointer: fine)').matches) {
    window.addEventListener('mousemove', handleMouseMove)
  }
})
onUnmounted(() => {
  if (timer) clearInterval(timer)
  window.removeEventListener('mousemove', handleMouseMove)
})

// ---- scroll reveal directive ----
const vReveal = {
  mounted(el: HTMLElement) {
    el.classList.add('reveal')
    const io = new IntersectionObserver(
      (entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            entry.target.classList.add('is-visible')
            io.unobserve(entry.target)
          }
        })
      },
      { threshold: 0.15 },
    )
    io.observe(el)
  },
}
</script>

<template>
  <div class="grain-overlay"></div>

  <div class="sparkle-layer">
    <span
      class="sparkle"
      v-for="s in sparkles"
      :key="s.id"
      :style="{ left: s.x + 'px', top: s.y + 'px' }"
      >✦</span
    >
  </div>

  <div class="petals-layer">
    <span
      class="confetti-petal"
      v-for="p in petals"
      :key="p.id"
      :style="{
        '--x': p.x + 'px',
        '--r': p.rot + 'deg',
        top: p.top,
        animationDelay: p.delay + 's',
        background: p.hue ? 'var(--gold-light)' : 'var(--blush)',
      }"
    ></span>
  </div>

  <div class="hearts-layer">
    <div
      class="heart"
      v-for="h in hearts"
      :key="h.id"
      :style="{
        left: h.left + '%',
        animationDuration: h.dur + 's',
        animationDelay: h.delay + 's',
        fontSize: h.size + 'rem',
      }"
    >
      {{ h.glyph }}
    </div>
  </div>

  <section class="hero">
    <p class="eyebrow label">a little something for</p>
    <h1 class="hero-title">Happy Birthday</h1>
    <p class="hero-sub script" style="font-size: 1.6rem">{{ herName }}</p>

    <div class="envelope-wrap">
      <div class="envelope" :class="{ 'is-open': envelopeOpen, 'is-lifted': envelopeOpen }">
        <div class="postage-stamp"><span>✦</span></div>
        <div class="card-out" v-if="envelopeOpen" @click="openBook">
          <div class="tap-hint">tap to open</div>
        </div>
        <div class="envelope-body"></div>
        <div class="envelope-flap"></div>
        <button
          class="wax-seal"
          :class="{ 'is-broken': sealBroken }"
          @click="breakSeal"
          aria-label="Break the seal"
        >
          <span class="seal-monogram script"
            >{{ yourName.charAt(0) }}<span class="amp">&amp;</span>{{ herName.charAt(0) }}</span
          >
        </button>
      </div>
    </div>

    <p class="hint" v-if="!sealBroken">break the seal to begin</p>
    <p class="hint" v-else-if="!envelopeOpen">opening…</p>
    <p class="hint" v-else>tap the card above ✦</p>
  </section>

  <div class="scene-backdrop" v-if="showBook" @click.self="closeBook">
    <div class="book-stage">
      <div class="book" :class="{ 'is-open': bookOpen }">
        <button class="close-scene" @click="closeBook" aria-label="Close">×</button>

        <div class="book-inside" v-if="bookOpen">
          <p
            class="letter-p"
            v-for="(p, i) in message"
            :key="i"
            :style="{ animationDelay: i * 0.35 + 's' }"
          >
            <span v-if="i === 0"
              ><span class="drop-cap">{{ firstLetter }}</span
              >{{ firstRest }}</span
            >
            <template v-else>{{ p }}</template>
          </p>
          <button class="bouquet-cue" @click="revealBouquet" v-if="!bouquetRevealed">
            🌹 a bouquet for you
          </button>
          <p class="bouquet-note script" v-else>for you, {{ herName }} — always</p>
        </div>

        <div class="book-cover" @click="bookOpen = true" v-show="!bookOpen">
          <div class="monogram script">{{ herName.charAt(0) }}</div>
          <div class="cover-title script">For {{ herName }}</div>
          <div class="open-cta label">tap to open</div>
        </div>
      </div>

      <div class="bouquet-reveal" v-if="bouquetRevealed">
        <div class="bouquet-glow"></div>
        <svg class="bouquet-svg" viewBox="0 0 300 400" xmlns="http://www.w3.org/2000/svg">
          <!-- paper wrap -->
          <path class="wrap-paper" d="M150,296 L252,392 L48,392 Z" />
          <path class="wrap-paper wrap-paper-fold" d="M150,296 L206,392 L94,392 Z" />

          <!-- stems -->
          <path v-for="(r, i) in roses" :key="'stem' + i" class="stem" :d="r.stem" />

          <!-- leaves -->
          <g
            v-for="(l, i) in leaves"
            :key="'leaf' + i"
            :transform="
              'translate(' +
              l.x +
              ',' +
              l.y +
              ') rotate(' +
              l.rotate +
              ') scale(' +
              (l.flip ? -l.scale : l.scale) +
              ',' +
              l.scale +
              ')'
            "
          >
            <path class="leaf" :d="leafPath" />
            <line class="leaf-vein" x1="3" y1="0" x2="30" y2="0" />
          </g>

          <!-- ribbon -->
          <g class="ribbon" transform="translate(150,297)">
            <path class="ribbon-loop" d="M0,0 C-26,-7 -30,-23 -14,-27 C-4,-29 2,-19 0,0 Z" />
            <path class="ribbon-loop" d="M0,0 C26,-7 30,-23 14,-27 C4,-29 -2,-19 0,0 Z" />
            <rect
              class="ribbon-knot"
              x="-7"
              y="-7"
              width="14"
              height="14"
              rx="2"
              transform="rotate(45)"
            />
          </g>

          <!-- roses -->
          <g
            v-for="(r, i) in roses"
            :key="'rose' + i"
            :transform="
              'translate(' + r.x + ',' + r.y + ') rotate(' + r.rotate + ') scale(' + r.scale + ')'
            "
          >
            <path class="rose-calyx" d="M-7,3 L0,17 L7,3 Z" />
            <g v-for="n in 6" :key="'o' + n" :transform="'rotate(' + n * 60 + ')'">
              <path class="petal petal-outer" :d="outerPetalPath" :fill="r.outer" />
            </g>
            <g v-for="n in 6" :key="'m' + n" :transform="'rotate(' + (n * 60 + 30) + ')'">
              <path class="petal petal-mid" :d="midPetalPath" :fill="r.mid" />
            </g>
            <g v-for="n in 5" :key="'i' + n" :transform="'rotate(' + (n * 72 + 15) + ')'">
              <path class="petal petal-inner" :d="innerPetalPath" :fill="r.inner" />
            </g>
            <path class="rose-center" :d="centerBudPath" />
          </g>
        </svg>
      </div>
    </div>
  </div>

  <section id="countdown" v-reveal>
    <div class="section-head">
      <h2>Counting Down To You</h2>
      <p v-if="!birthday.isToday">every moment brings us closer to celebrating you</p>
      <p v-else class="script" style="font-size: 1.5rem; color: var(--gold-light)">
        Today is the day, {{ herName }} 🎂
      </p>
    </div>
    <div class="counter-grid">
      <div class="counter-item">
        <span class="counter-num">{{ birthday.days }}</span
        ><span class="counter-label">days</span>
      </div>
      <div class="counter-item">
        <span class="counter-num">{{ birthday.hours }}</span
        ><span class="counter-label">hours</span>
      </div>
      <div class="counter-item">
        <span class="counter-num">{{ birthday.minutes }}</span
        ><span class="counter-label">minutes</span>
      </div>
      <div class="counter-item">
        <span class="counter-num pulse">{{ birthday.seconds }}</span
        ><span class="counter-label">seconds</span>
      </div>
    </div>
  </section>

  <section id="story" v-reveal>
    <div class="section-head">
      <h2>Our Love Story</h2>
      <p>a few chapters, so far</p>
    </div>
    <div class="timeline">
      <div class="timeline-item" v-for="(m, i) in milestones" :key="i" v-reveal>
        <div class="timeline-dot"></div>
        <div class="timeline-content">
          <span class="timeline-date label">{{ m.date }}</span>
          <h3>{{ m.title }}</h3>
          <p>{{ m.text }}</p>
        </div>
      </div>
    </div>
  </section>

  <section id="reasons" v-reveal>
    <div class="section-head">
      <h2>Reasons, Among Many</h2>
      <p>tap a card to turn it over</p>
    </div>
    <div class="reasons-grid">
      <div
        class="reason-card"
        v-for="(reason, i) in reasons"
        :key="i"
        :class="{ 'is-flipped': flipped.has(i) }"
        @click="toggleReason(i)"
        :style="{ transitionDelay: i * 0.06 + 's' }"
        v-reveal
      >
        <div class="reason-inner">
          <div class="reason-face reason-front">
            <span class="reason-index label">Reason {{ i + 1 }}</span>
            <span class="reason-glyph">♡</span>
          </div>
          <div class="reason-face reason-back">
            <p>{{ reason }}</p>
          </div>
        </div>
      </div>
    </div>
  </section>

  <section id="gallery" v-reveal>
    <div class="section-head">
      <h2>Our Moments</h2>
      <p>a few of my favorites</p>
    </div>
    <div class="gallery-grid">
      <figure
        class="polaroid"
        v-for="(photo, i) in photos"
        :key="i"
        :style="{
          transform: 'rotate(' + tilts[i % tilts.length] + 'deg)',
          transitionDelay: i * 0.06 + 's',
        }"
        @click="lightboxPhoto = photo"
        v-reveal
      >
        <div class="frame">
          <img v-if="photo.src" :src="photo.src" :alt="photo.caption" />
          <svg v-else viewBox="0 0 24 24">
            <path
              d="M9 3 7.17 5H4a2 2 0 0 0-2 2v11a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-3.17L15 3H9zm3 15a5 5 0 1 1 0-10 5 5 0 0 1 0 10z"
            />
          </svg>
        </div>
        <figcaption>{{ photo.caption }}</figcaption>
      </figure>
    </div>
  </section>

  <section id="film" v-reveal>
    <div class="section-head">
      <h2>A Little Film for You</h2>
      <p>{{ mainVideo.caption }}</p>
    </div>
    <div class="film-frame">
      <video
        v-if="videoPlaying && mainVideo.src"
        :src="mainVideo.src"
        class="film-video"
        controls
        autoplay
      ></video>
      <div v-else class="film-thumb" @click="playMainVideo">
        <img v-if="mainVideo.poster" :src="mainVideo.poster" class="film-poster" alt="" />
        <div class="play-btn big">
          <svg viewBox="0 0 24 24"><path d="M8 5v14l11-7z" /></svg>
        </div>
        <div v-if="!mainVideo.src" class="video-placeholder">
          Add your video URL in the config to play it here.
        </div>
      </div>
    </div>
  </section>

  <section id="wish" v-reveal>
    <div class="section-head">
      <h2>Make a Wish</h2>
      <p v-if="candleLit">tap the candle to blow it out</p>
      <p v-else>&nbsp;</p>
    </div>
    <div class="cake-wrap">
      <svg
        class="cake"
        viewBox="0 0 200 160"
        @click="blowCandle"
        role="button"
        aria-label="Blow out the candle"
      >
        <defs>
          <linearGradient id="cakeGrad" x1="0" y1="0" x2="0" y2="1">
            <stop offset="0%" stop-color="var(--wine-soft)" />
            <stop offset="100%" stop-color="var(--wine-deep)" />
          </linearGradient>
        </defs>
        <rect x="94" y="30" width="12" height="42" rx="3" fill="var(--gold)" />
        <g v-if="candleLit" class="flame-group">
          <path
            class="flame"
            d="M100 6 C 93 20 88 30 100 36 C 112 30 107 20 100 6 Z"
            fill="#ffb84d"
          />
        </g>
        <g v-else class="smoke">
          <path
            d="M100 30 Q 106 20 100 10 Q 94 0 100 -10"
            stroke="rgba(255,243,247,0.5)"
            stroke-width="2.5"
            fill="none"
            stroke-linecap="round"
          />
        </g>
        <path d="M28 72 Q 64 48 100 72 Q 136 48 172 72 L172 72 L28 72 Z" fill="var(--paper)" />
        <rect x="28" y="72" width="144" height="66" rx="12" fill="url(#cakeGrad)" />
        <rect x="28" y="98" width="144" height="6" fill="rgba(246,217,196,.3)" />
      </svg>
      <p class="wish-hint" v-if="candleLit">🕯️ tap</p>
      <p class="wish-text script" v-else-if="wishGranted">{{ wishText }}</p>
    </div>
  </section>

  <footer v-reveal>
    <span class="script">Happy Birthday, {{ herName }}. I love you.</span>
    <p>— {{ yourName }}</p>
  </footer>

  <div class="modal" v-if="lightboxPhoto" @click.self="lightboxPhoto = null">
    <button class="modal-close" @click="lightboxPhoto = null">×</button>
    <figure>
      <img v-if="lightboxPhoto.src" :src="lightboxPhoto.src" :alt="lightboxPhoto.caption" />
      <figcaption>{{ lightboxPhoto.caption }}</figcaption>
    </figure>
  </div>

  <button
    class="music-toggle"
    v-if="musicUrl"
    @click="toggleMusic"
    :class="{ 'is-playing': musicPlaying }"
    aria-label="Toggle music"
  >
    <span class="eq-bar" v-for="n in 3" :key="n"></span>
  </button>
  <audio ref="audio" :src="musicUrl" loop></audio>
</template>

<style>
:root {
  --wine: #8c1c4d; /* deep romantic pink */
  --wine-deep: #4a0f30; /* deep plum-pink, base background */
  --wine-soft: #b23368; /* soft rose pink */
  --blush: #ffd6e4; /* light blush pink */
  --paper: #fff3f6; /* warm pink-white paper */
  --paper-shadow: #f3d9e2;
  --gold: #dba384; /* rose gold accent */
  --gold-light: #f6d9c4; /* champagne rose-gold */
  --ink: #5c1638;
  --pink-mid: #d46a8c; /* mid rose pink, used for bouquet variety */
  --leaf: #8fa377; /* soft sage green, stems & leaves */
  --leaf-dark: #5f7350; /* deeper green, veins & calyx */
}
* {
  box-sizing: border-box;
}
html,
body {
  margin: 0;
  padding: 0;
}
body {
  background: radial-gradient(
    ellipse at top,
    var(--wine-soft) 0%,
    var(--wine-deep) 65%,
    #2e0a1f 100%
  );
  font-family: 'Cormorant Garamond', serif;
  color: var(--paper);
  min-height: 100vh;
  overflow-x: hidden;
}
#app {
  font-weight: normal;
}
h1,
h2,
h3 {
  font-family: 'Playfair Display', serif;
  margin: 0;
}
.script {
  font-family: 'Dancing Script', cursive;
}
.label {
  font-family: 'Jost', sans-serif;
  letter-spacing: 0.18em;
  text-transform: uppercase;
  font-size: 0.72rem;
  opacity: 0.75;
}
button {
  font-family: inherit;
  cursor: pointer;
}

/* ---------- grain + ambient layers ---------- */
.grain-overlay {
  position: fixed;
  inset: 0;
  z-index: 2;
  pointer-events: none;
  opacity: 0.035;
  mix-blend-mode: overlay;
  background-image: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='120' height='120'><filter id='n'><feTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='2' stitchTiles='stitch'/></filter><rect width='100%25' height='100%25' filter='url(%23n)'/></svg>");
}
.sparkle-layer {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 3;
  overflow: hidden;
}
.sparkle {
  position: absolute;
  color: var(--gold-light);
  font-size: 0.8rem;
  opacity: 0.9;
  transform: translate(-50%, -50%);
  animation: sparkleFade 0.7s ease forwards;
}
@keyframes sparkleFade {
  0% {
    opacity: 0.9;
    transform: translate(-50%, -50%) scale(1);
  }
  100% {
    opacity: 0;
    transform: translate(-50%, -80%) scale(0.4);
  }
}
@media (pointer: coarse) {
  .sparkle-layer {
    display: none;
  }
}

.petals-layer {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 6;
  overflow: hidden;
}
.confetti-petal {
  position: absolute;
  left: 50vw;
  width: 10px;
  height: 14px;
  border-radius: 0 100% 100% 100%;
  transform: rotate(45deg);
  animation: petalFall 1.3s ease-in forwards;
}
@keyframes petalFall {
  0% {
    transform: translate(0, 0) rotate(45deg);
    opacity: 1;
  }
  100% {
    transform: translate(var(--x), 260px) rotate(var(--r));
    opacity: 0;
  }
}

.hearts-layer {
  position: fixed;
  inset: 0;
  pointer-events: none;
  overflow: hidden;
  z-index: 0;
}
.heart {
  position: absolute;
  bottom: -40px;
  opacity: 0.35;
  color: var(--gold-light);
  animation: floatUp linear infinite;
  font-size: 1.2rem;
}
@keyframes floatUp {
  0% {
    transform: translateY(0) translateX(0) rotate(0deg);
    opacity: 0;
  }
  10% {
    opacity: 0.45;
  }
  90% {
    opacity: 0.35;
  }
  100% {
    transform: translateY(-110vh) translateX(20px) rotate(25deg);
    opacity: 0;
  }
}

/* ---------- scroll reveal ---------- */
.reveal {
  opacity: 0;
  transform: translateY(24px);
  transition:
    opacity 0.8s ease,
    transform 0.8s ease;
}
.reveal.is-visible {
  opacity: 1;
  transform: translateY(0);
}

/* ---------- hero / envelope ---------- */
.hero {
  position: relative;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  padding: 2rem 1.2rem;
  z-index: 1;
}
.eyebrow {
  margin-bottom: 0.6rem;
  color: var(--gold-light);
  opacity: 0.8;
}
.hero-title {
  font-size: clamp(2rem, 6vw, 3.2rem);
  margin-bottom: 0.3rem;
  background: linear-gradient(120deg, var(--gold-light) 30%, #fff8f2 50%, var(--gold-light) 70%);
  background-size: 200% auto;
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  animation: shimmer 5s linear infinite;
  text-shadow: 0 0 40px rgba(246, 217, 196, 0.25);
}
@keyframes shimmer {
  0% {
    background-position: 200% center;
  }
  100% {
    background-position: -200% center;
  }
}
.hero-sub {
  font-size: 1.1rem;
  opacity: 0.85;
  margin-bottom: 3rem;
  color: var(--blush);
}

.envelope-wrap {
  perspective: 1400px;
  margin-bottom: 1.6rem;
}
.envelope {
  width: 290px;
  max-width: 80vw;
  height: 190px;
  position: relative;
  filter: drop-shadow(0 25px 35px rgba(0, 0, 0, 0.45))
    drop-shadow(0 0 45px rgba(219, 163, 132, 0.28));
  transition: transform 0.6s ease;
}
.envelope.is-lifted {
  transform: translateY(-14px);
}
.envelope-body {
  position: absolute;
  inset: 0;
  background: linear-gradient(160deg, var(--wine-soft), var(--wine));
  border-radius: 6px;
  border: 1px solid rgba(246, 217, 196, 0.4);
}
.envelope-body::before {
  content: '';
  position: absolute;
  left: 0;
  right: 0;
  bottom: 0;
  height: 70%;
  background: linear-gradient(160deg, var(--wine), var(--wine-deep));
  clip-path: polygon(0 100%, 100% 100%, 100% 0, 50% 55%, 0 0);
  border-radius: 0 0 6px 6px;
}
.envelope-flap {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 52%;
  background: linear-gradient(200deg, var(--wine-soft), var(--wine));
  clip-path: polygon(0 0, 100% 0, 50% 100%);
  transform-origin: top center;
  transition: transform 1.1s cubic-bezier(0.6, -0.28, 0.35, 1.3);
  z-index: 3;
  border-top: 1px solid rgba(246, 217, 196, 0.35);
}
.envelope.is-open .envelope-flap {
  transform: rotateX(-178deg);
}

.postage-stamp {
  position: absolute;
  top: 10px;
  right: 12px;
  width: 38px;
  height: 46px;
  border: 2px dashed rgba(246, 217, 196, 0.6);
  border-radius: 2px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--gold-light);
  font-size: 1rem;
  transform: rotate(-6deg);
  z-index: 5;
  background: rgba(140, 28, 77, 0.35);
}

.wax-seal {
  position: absolute;
  top: 46%;
  left: 50%;
  width: 58px;
  height: 58px;
  border-radius: 50%;
  transform: translate(-50%, -50%);
  background: radial-gradient(circle at 35% 30%, var(--gold-light), var(--gold) 60%, #8a5560 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow:
    0 4px 10px rgba(0, 0, 0, 0.4),
    inset 0 0 8px rgba(255, 255, 255, 0.3);
  z-index: 4;
  transition:
    transform 0.5s ease,
    opacity 0.5s ease;
  border: none;
}
.seal-monogram {
  font-size: 1.05rem;
  color: var(--wine-deep);
  opacity: 0.9;
}
.seal-monogram .amp {
  font-size: 0.75rem;
  margin: 0 1px;
  opacity: 0.7;
}
.wax-seal.is-broken {
  transform: translate(-50%, -50%) scale(1.6) rotate(25deg);
  opacity: 0;
  pointer-events: none;
}

.card-out {
  position: absolute;
  top: -40%;
  left: 50%;
  width: 230px;
  max-width: 76vw;
  height: 150px;
  transform: translate(-50%, 0) scale(0.9);
  background: var(--paper);
  border-radius: 4px;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.4);
  z-index: 2;
  opacity: 0;
  transition:
    transform 0.9s cubic-bezier(0.2, 0.8, 0.3, 1.1) 0.5s,
    opacity 0.5s ease 0.5s;
  display: flex;
  align-items: center;
  justify-content: center;
}
.envelope.is-open .card-out {
  transform: translate(-50%, -92%) scale(1);
  opacity: 1;
}
.tap-hint {
  color: var(--wine);
  font-family: 'Jost', sans-serif;
  font-size: 0.75rem;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  opacity: 0.65;
}

.hint {
  margin-top: 1.4rem;
  font-size: 0.95rem;
  opacity: 0.75;
}

/* ---------- letter overlay ---------- */
.scene-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(30, 6, 20, 0.75);
  backdrop-filter: blur(3px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 20;
  padding: 1.5rem;
  animation: fadeIn 0.4s ease;
  overflow-y: auto;
}
@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

/* holds the letter book and, once revealed, the bouquet beside/below it */
.book-stage {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 2.4rem;
  flex-wrap: wrap;
  max-width: 100%;
  margin: auto;
}

.book {
  width: 380px;
  max-width: 92vw;
  height: 520px;
  max-height: 82vh;
  position: relative;
  perspective: 1800px;
}

.book-cover {
  position: absolute;
  inset: 0;
  border-radius: 6px;
  cursor: pointer;
  background: linear-gradient(155deg, var(--wine-soft), var(--wine) 60%, var(--wine-deep));
  border: 1px solid rgba(246, 217, 196, 0.45);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  transform-origin: left center;
  transition: transform 1.3s cubic-bezier(0.6, 0, 0.3, 1);
  backface-visibility: hidden;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5);
  padding: 2rem;
  text-align: center;
}
.book.is-open .book-cover {
  transform: rotateY(-150deg);
}
.book-cover .monogram {
  width: 70px;
  height: 70px;
  border-radius: 50%;
  border: 1.5px solid var(--gold);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.8rem;
  color: var(--gold-light);
}
.book-cover .cover-title {
  font-size: 1.6rem;
  color: var(--gold-light);
}
.book-cover .open-cta {
  font-size: 0.85rem;
  opacity: 0.8;
}

.book-inside {
  position: absolute;
  inset: 0;
  background: var(--paper);
  color: var(--ink);
  border-radius: 6px;
  padding: 2.6rem 2.2rem;
  overflow-y: auto;
  box-shadow: inset 0 0 40px rgba(140, 28, 77, 0.08);
  display: flex;
  flex-direction: column;
  gap: 1rem;
}
.book-inside::-webkit-scrollbar {
  width: 6px;
}
.book-inside::-webkit-scrollbar-thumb {
  background: var(--paper-shadow);
  border-radius: 6px;
}
.letter-p {
  font-size: 1.18rem;
  line-height: 1.55;
  opacity: 0;
  transform: translateY(8px);
  animation: revealP 0.7s ease forwards;
}
@keyframes revealP {
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
.drop-cap {
  float: left;
  font-family: 'Playfair Display', serif;
  font-size: 3.2rem;
  line-height: 0.8;
  padding-right: 0.15rem;
  padding-top: 0.1rem;
  color: var(--wine);
}

.close-scene {
  position: absolute;
  top: -3rem;
  right: 0;
  background: none;
  border: none;
  color: var(--gold-light);
  font-size: 1.6rem;
  opacity: 0.85;
}

/* button that reveals the bouquet, shown after the letter */
.bouquet-cue {
  align-self: center;
  margin-top: 0.6rem;
  background: linear-gradient(135deg, var(--wine-soft), var(--wine));
  border: 1px solid var(--gold);
  color: var(--gold-light);
  border-radius: 30px;
  padding: 0.6rem 1.4rem;
  font-family: 'Jost', sans-serif;
  font-size: 0.85rem;
  letter-spacing: 0.08em;
  box-shadow: 0 8px 18px rgba(0, 0, 0, 0.25);
  transition:
    transform 0.25s ease,
    box-shadow 0.25s ease;
  animation: bob 1.8s ease-in-out infinite;
}
.bouquet-cue:hover {
  transform: translateY(-2px);
  box-shadow: 0 12px 22px rgba(0, 0, 0, 0.3);
}
.bouquet-note {
  align-self: center;
  margin-top: 0.6rem;
  font-size: 1.3rem;
  color: var(--wine);
  opacity: 0.85;
  animation: revealP 0.6s ease forwards;
}
@keyframes bob {
  0%,
  100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(6px);
  }
}

/* ---------- shared section styles ---------- */
section {
  position: relative;
  z-index: 1;
  padding: 5rem 1.5rem;
  max-width: 1100px;
  margin: 0 auto;
}
.section-head {
  text-align: center;
  margin-bottom: 3rem;
}
.section-head h2 {
  font-size: clamp(1.8rem, 4vw, 2.6rem);
  color: var(--gold-light);
}
.section-head h2::after {
  content: '';
  display: block;
  width: 60px;
  height: 2px;
  margin: 0.8rem auto 0;
  background: linear-gradient(90deg, transparent, var(--gold), transparent);
}
.section-head p {
  opacity: 0.75;
  margin-top: 0.6rem;
}

/* ---------- together-since / birthday counters ---------- */
.counter-grid {
  display: flex;
  justify-content: center;
  gap: 2.4rem;
  flex-wrap: wrap;
}
.counter-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-width: 80px;
}
.counter-num {
  font-family: 'Playfair Display', serif;
  font-size: clamp(2rem, 5vw, 3rem);
  color: var(--gold-light);
}
.counter-num.pulse {
  animation: pulseNum 1s ease-in-out infinite;
}
@keyframes pulseNum {
  0%,
  100% {
    opacity: 1;
  }
  50% {
    opacity: 0.55;
  }
}
.counter-label {
  font-family: 'Jost', sans-serif;
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.14em;
  opacity: 0.75;
  margin-top: 0.3rem;
}
.counter-caption {
  text-align: center;
  margin-top: 2rem;
  font-size: 1.1rem;
  opacity: 0.8;
}

/* ---------- love story timeline ---------- */
.timeline {
  position: relative;
  max-width: 640px;
  margin: 0 auto;
  padding-left: 2.4rem;
}
.timeline::before {
  content: '';
  position: absolute;
  left: 7px;
  top: 6px;
  bottom: 6px;
  width: 2px;
  background: linear-gradient(var(--gold), transparent);
  opacity: 0.55;
}
.timeline-item {
  position: relative;
  margin-bottom: 2.6rem;
}
.timeline-item:last-child {
  margin-bottom: 0;
}
.timeline-dot {
  position: absolute;
  left: -2.4rem;
  top: 4px;
  width: 14px;
  height: 14px;
  border-radius: 50%;
  background: radial-gradient(circle at 35% 30%, var(--gold-light), var(--gold));
  box-shadow: 0 0 0 5px rgba(219, 163, 132, 0.18);
}
.timeline-content h3 {
  color: var(--gold-light);
  font-size: 1.3rem;
  margin: 0.3rem 0;
}
.timeline-content p {
  opacity: 0.85;
  line-height: 1.5;
}
.timeline-date {
  opacity: 0.7;
}

/* ---------- rose bouquet reveal (shown beside/below the letter card) ---------- */
.bouquet-reveal {
  position: relative;
  width: 300px;
  max-width: 84vw;
  animation: bouquetIn 0.9s cubic-bezier(0.2, 0.8, 0.3, 1.1);
}
@keyframes bouquetIn {
  0% {
    opacity: 0;
    transform: translateY(24px) scale(0.85);
  }
  100% {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}
.bouquet-glow {
  position: absolute;
  top: 12%;
  left: 50%;
  width: 340px;
  height: 340px;
  transform: translate(-50%, -50%);
  background: radial-gradient(circle, rgba(219, 163, 132, 0.3), transparent 70%);
  filter: blur(6px);
  z-index: 0;
  pointer-events: none;
}
.bouquet-svg {
  position: relative;
  z-index: 1;
  width: 100%;
  height: auto;
  display: block;
  overflow: visible;
  filter: drop-shadow(0 18px 26px rgba(0, 0, 0, 0.35));
}
.stem {
  fill: none;
  stroke: var(--leaf-dark);
  stroke-width: 4;
  stroke-linecap: round;
}
.leaf {
  fill: var(--leaf);
}
.leaf-vein {
  stroke: var(--leaf-dark);
  stroke-width: 1;
  opacity: 0.5;
}
.rose-calyx {
  fill: var(--leaf-dark);
}
.rose-center {
  fill: var(--gold-light);
}
.petal-outer {
  filter: drop-shadow(0 1px 1px rgba(0, 0, 0, 0.15));
}
.wrap-paper {
  fill: var(--paper);
}
.wrap-paper-fold {
  fill: var(--paper-shadow);
}
.ribbon-loop {
  fill: var(--gold-light);
}
.ribbon-knot {
  fill: var(--gold);
}

/* ---------- reasons flip cards ---------- */
.reasons-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
  gap: 1.6rem;
}
.reason-card {
  aspect-ratio: 3/4;
  perspective: 1200px;
  cursor: pointer;
}
.reason-inner {
  position: relative;
  width: 100%;
  height: 100%;
  transition: transform 0.7s cubic-bezier(0.4, 0.2, 0.2, 1);
  transform-style: preserve-3d;
}
.reason-card.is-flipped .reason-inner {
  transform: rotateY(180deg);
}
.reason-face {
  position: absolute;
  inset: 0;
  backface-visibility: hidden;
  border-radius: 10px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 1.2rem;
  text-align: center;
}
.reason-front {
  background: linear-gradient(155deg, var(--wine-soft), var(--wine-deep));
  border: 1px solid rgba(246, 217, 196, 0.35);
  gap: 0.8rem;
}
.reason-glyph {
  font-size: 1.8rem;
  color: var(--gold-light);
}
.reason-back {
  background: var(--paper);
  color: var(--ink);
  transform: rotateY(180deg);
  font-size: 1.05rem;
  line-height: 1.4;
}

/* ---------- gallery ---------- */
.gallery-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(190px, 1fr));
  gap: 2.2rem 1.6rem;
  justify-items: center;
}
.polaroid {
  background: var(--paper);
  padding: 0.7rem 0.7rem 2rem;
  width: 190px;
  border-radius: 2px;
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.35);
  cursor: pointer;
  transition:
    transform 0.3s ease,
    box-shadow 0.3s ease;
}
.polaroid:hover {
  transform: rotate(0deg) scale(1.05) !important;
  box-shadow: 0 18px 30px rgba(0, 0, 0, 0.45);
  z-index: 5;
}
.polaroid .frame {
  width: 100%;
  aspect-ratio: 1/1;
  background: linear-gradient(135deg, var(--blush), #f0c6d4);
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  color: var(--wine);
}
.polaroid .frame img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}
.polaroid .frame svg {
  width: 34px;
  height: 34px;
  opacity: 0.55;
}
.polaroid figcaption {
  display: block;
  text-align: center;
  font-family: 'Jost', sans-serif;
  font-size: 0.78rem;
  color: var(--ink);
  margin-top: 0.6rem;
  opacity: 0.75;
}

/* ---------- main video ---------- */
.film-frame {
  max-width: 780px;
  margin: 0 auto;
  border-radius: 14px;
  padding: 10px;
  background: linear-gradient(155deg, var(--wine-soft), var(--wine-deep));
  border: 1px solid var(--gold);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4);
}
.film-video {
  width: 100%;
  aspect-ratio: 16/9;
  border-radius: 8px;
  display: block;
  background: #000;
}
.film-thumb {
  position: relative;
  width: 100%;
  aspect-ratio: 16/9;
  border-radius: 8px;
  overflow: hidden;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, var(--wine-soft), var(--wine-deep));
}
.film-poster {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
}
.play-btn {
  width: 52px;
  height: 52px;
  border-radius: 50%;
  background: rgba(255, 243, 246, 0.94);
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  z-index: 2;
}
.play-btn.big {
  width: 76px;
  height: 76px;
}
.play-btn svg {
  width: 20px;
  height: 20px;
  fill: var(--wine);
  margin-left: 3px;
}
.play-btn.big svg {
  width: 30px;
  height: 30px;
}
.video-placeholder {
  position: absolute;
  bottom: 1rem;
  left: 1rem;
  right: 1rem;
  text-align: center;
  color: var(--paper);
  opacity: 0.7;
  font-family: 'Jost', sans-serif;
  font-size: 0.8rem;
}

/* ---------- make a wish / candle ---------- */
.cake-wrap {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.2rem;
}
.cake {
  width: 220px;
  max-width: 60vw;
  cursor: pointer;
  overflow: visible;
}
.flame {
  animation: flicker 1.4s ease-in-out infinite;
  transform-origin: 100px 34px;
}
@keyframes flicker {
  0%,
  100% {
    transform: scale(1) skewX(0deg);
  }
  35% {
    transform: scale(1.06, 0.94) skewX(2deg);
  }
  65% {
    transform: scale(0.94, 1.05) skewX(-2deg);
  }
}
.smoke path {
  animation: smokeRise 1.2s ease-out forwards;
}
@keyframes smokeRise {
  0% {
    opacity: 0.6;
    transform: translateY(0);
  }
  100% {
    opacity: 0;
    transform: translateY(-14px);
  }
}
.wish-hint {
  font-family: 'Jost', sans-serif;
  font-size: 0.85rem;
  opacity: 0.75;
}
.wish-text {
  font-size: clamp(1.3rem, 3vw, 1.7rem);
  color: var(--gold-light);
  text-align: center;
  max-width: 480px;
  animation: revealP 0.8s ease forwards;
}

/* ---------- modals ---------- */
.modal {
  position: fixed;
  inset: 0;
  background: rgba(28, 6, 20, 0.9);
  z-index: 30;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 2rem;
  animation: fadeIn 0.3s ease;
}
.modal img,
.modal video {
  max-width: 92vw;
  max-height: 80vh;
  border-radius: 6px;
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.6);
}
.modal figcaption {
  text-align: center;
  margin-top: 1rem;
  font-size: 1.1rem;
  color: var(--paper);
  opacity: 0.85;
}
.modal-close {
  position: absolute;
  top: 1.4rem;
  right: 1.6rem;
  background: none;
  border: none;
  color: var(--paper);
  font-size: 2rem;
  opacity: 0.85;
}

/* ---------- footer ---------- */
footer {
  text-align: center;
  padding: 5rem 1.5rem 6rem;
  position: relative;
  z-index: 1;
}
footer .script {
  font-size: clamp(1.6rem, 4vw, 2.3rem);
  color: var(--gold-light);
  display: block;
  margin-bottom: 0.6rem;
}
footer p {
  opacity: 0.75;
}

/* ---------- music toggle / equalizer ---------- */
.music-toggle {
  position: fixed;
  bottom: 1.4rem;
  right: 1.4rem;
  z-index: 15;
  width: 52px;
  height: 52px;
  border-radius: 50%;
  background: var(--gold);
  border: none;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 2px;
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.4);
}
.eq-bar {
  width: 3px;
  border-radius: 2px;
  background: var(--wine-deep);
}
.eq-bar:nth-child(1) {
  height: 8px;
}
.eq-bar:nth-child(2) {
  height: 16px;
}
.eq-bar:nth-child(3) {
  height: 10px;
}
.music-toggle.is-playing .eq-bar {
  animation: eqBounce 0.9s ease-in-out infinite;
}
.music-toggle.is-playing .eq-bar:nth-child(1) {
  animation-delay: 0s;
}
.music-toggle.is-playing .eq-bar:nth-child(2) {
  animation-delay: 0.15s;
}
.music-toggle.is-playing .eq-bar:nth-child(3) {
  animation-delay: 0.3s;
}
@keyframes eqBounce {
  0%,
  100% {
    height: 8px;
  }
  50% {
    height: 20px;
  }
}

@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.001s !important;
    transition-duration: 0.2s !important;
  }
}

/* ---------- responsive: laptop / tablet ---------- */
@media (max-width: 1024px) {
  section {
    padding: 4rem 1.4rem;
  }
}

/* ---------- responsive: tablet ---------- */
@media (max-width: 768px) {
  section {
    padding: 3.5rem 1.2rem;
  }
  .section-head {
    margin-bottom: 2.2rem;
  }
  .counter-grid {
    gap: 1.6rem;
  }
  .reasons-grid {
    grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
    gap: 1.2rem;
  }
  .film-frame {
    padding: 7px;
    border-radius: 10px;
  }
}

/* ---------- responsive: phone (cp) ---------- */
@media (max-width: 520px) {
  .hero {
    padding: 1.5rem 1rem;
  }
  section {
    padding: 3rem 1rem;
  }
  .gallery-grid {
    grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
    gap: 1.6rem 1rem;
  }
  .polaroid {
    width: 100%;
  }
  .reasons-grid {
    grid-template-columns: repeat(2, 1fr);
  }
  .counter-grid {
    gap: 1.2rem;
  }
  .counter-item {
    min-width: 64px;
  }
  .book-inside {
    padding: 2rem 1.4rem;
  }
  .letter-p {
    font-size: 1.05rem;
  }
  .cake {
    max-width: 72vw;
  }
  .timeline {
    padding-left: 2rem;
  }
  .timeline-dot {
    left: -2rem;
  }
  .bouquet-reveal {
    width: 230px;
  }
}

@media (max-width: 360px) {
  .reasons-grid {
    grid-template-columns: 1fr;
  }
  .envelope {
    height: 170px;
  }
}
</style>
