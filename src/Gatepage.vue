<script setup lang="ts">
import { ref, reactive, onMounted, onUnmounted } from 'vue'

const emit = defineEmits<{ (e: 'enter'): void }>()

/* ============================================================
   ✏️  EDIT THESE
   ============================================================ */

// Must match the birthday used on the main page's countdown.
const herBirthday = '2026-08-20'

// The image of him.
const manPhotoSrc = `${import.meta.env.BASE_URL}pics/man-photo.png` // Admin bypass password — typed into the tiny dot in the corner.
const ADMIN_PASSWORD = 'ttoapril30'

/* ============================================================
   State
   ============================================================ */

type Step =
  | 'identity'
  | 'wrong-identity'
  | 'color'
  | 'wrong-color'
  | 'blooming'
  | 'locked'
  | 'celebration'

const step = ref<Step>('identity')
const flowersDone = ref(false)
const adminUnlocked = ref(false)

function isBirthdayReached() {
  const target = new Date(herBirthday + 'T00:00:00').getTime()
  return Date.now() >= target
}

function canEnter() {
  return adminUnlocked.value || isBirthdayReached()
}

/* ============================================================
   💌 Little rotating love notes — a quiet, genuine touch that
   sits beneath the questions while she's deciding what to tap.
   ✏️ Edit these to sound like you.
   ============================================================ */
const romanticLines = [
  'made with love, just for you 💕',
  'every little detail, just for you',
  'still my favorite person',
  'falling for you a little more, every day',
]
const romanticIndex = ref(0)
let romanticTimer: ReturnType<typeof setInterval> | undefined

const bloomingLines = [
  'Opening your gift, Grace… 🌹',
  'Almost there, my love…',
  'Just a little more magic…',
]
const bloomingIndex = ref(0)
let bloomingTimer: ReturnType<typeof setInterval> | undefined

/* ============================================================
   Celebration sound effects (synthesized — no audio files needed)
   A full birthday soundscape: an opening fanfare, festive party
   horns, a confetti popper, a music-box rendition of "Happy
   Birthday to You", a crowd cheer, booming fireworks, and a
   grand finale — scheduled together so it reads as an actual
   party, not noise.
   ============================================================ */

let audioCtx: AudioContext | null = null
function getAudioCtx(): AudioContext | null {
  try {
    if (!audioCtx) {
      const Ctx = window.AudioContext || (window as any).webkitAudioContext
      if (!Ctx) return null
      audioCtx = new Ctx()
    }
    if (audioCtx.state === 'suspended') {
      audioCtx.resume().catch(() => {})
    }
    return audioCtx
  } catch (err) {
    console.error('AudioContext unavailable:', err)
    return null
  }
}

/** A single bright bell/chime note — like a party-favor jingle. */
function playChimeNote(freq: number, delay = 0, volume = 0.16) {
  const ctx = getAudioCtx()
  if (!ctx) return
  const now = ctx.currentTime + delay

  const osc = ctx.createOscillator()
  const osc2 = ctx.createOscillator()
  const gain = ctx.createGain()

  osc.type = 'sine'
  osc2.type = 'triangle'
  osc.frequency.setValueAtTime(freq, now)
  osc2.frequency.setValueAtTime(freq * 2.005, now) // slightly detuned octave = shimmer

  gain.gain.setValueAtTime(0.0001, now)
  gain.gain.linearRampToValueAtTime(volume, now + 0.012)
  gain.gain.exponentialRampToValueAtTime(0.0001, now + 0.85)

  osc.connect(gain)
  osc2.connect(gain)
  gain.connect(ctx.destination)

  osc.start(now)
  osc2.start(now)
  osc.stop(now + 0.9)
  osc2.stop(now + 0.9)
}

/** A short cheerful arpeggio — bright "ta-da!" sparkle, used to open/close the party. */
function playChimeArpeggio(delay = 0, ascending = true) {
  const notes = [523.25, 659.25, 783.99, 1046.5] // C5 E5 G5 C6 — bright major chord
  const seq = ascending ? notes : [...notes].reverse()
  seq.forEach((freq, i) => {
    playChimeNote(freq, delay + i * 0.09, 0.15)
  })
}

/** Kazoo-ish party horn "toot" with a bit of buzzy vibrato, like blowing a paper horn. */
function playPartyHorn(delay = 0) {
  const ctx = getAudioCtx()
  if (!ctx) return
  const now = ctx.currentTime + delay

  const osc = ctx.createOscillator()
  const buzz = ctx.createOscillator()
  const vibrato = ctx.createOscillator()
  const vibratoGain = ctx.createGain()
  const gain = ctx.createGain()

  osc.type = 'sawtooth'
  buzz.type = 'square'
  vibrato.type = 'sine'

  osc.frequency.setValueAtTime(300, now)
  osc.frequency.linearRampToValueAtTime(520, now + 0.12)
  osc.frequency.linearRampToValueAtTime(430, now + 0.5)

  buzz.frequency.setValueAtTime(300, now)
  buzz.frequency.linearRampToValueAtTime(520, now + 0.12)
  buzz.frequency.linearRampToValueAtTime(430, now + 0.5)

  // fast little wobble, the "buzzy paper reed" feel
  vibrato.frequency.setValueAtTime(28, now)
  vibratoGain.gain.setValueAtTime(9, now)
  vibrato.connect(vibratoGain)
  vibratoGain.connect(osc.frequency)
  vibratoGain.connect(buzz.frequency)

  gain.gain.setValueAtTime(0.0001, now)
  gain.gain.linearRampToValueAtTime(0.22, now + 0.04)
  gain.gain.setValueAtTime(0.2, now + 0.35)
  gain.gain.exponentialRampToValueAtTime(0.0001, now + 0.62)

  const buzzGain = ctx.createGain()
  buzzGain.gain.value = 0.35 // quieter layer under the main tone

  osc.connect(gain)
  buzz.connect(buzzGain)
  buzzGain.connect(gain)
  gain.connect(ctx.destination)

  osc.start(now)
  buzz.start(now)
  vibrato.start(now)
  osc.stop(now + 0.65)
  buzz.stop(now + 0.65)
  vibrato.stop(now + 0.65)
}

/** A quick balloon "pop" — tight click plus a fast pitch-drop thud. */
function playBalloonPop(delay = 0) {
  const ctx = getAudioCtx()
  if (!ctx) return
  const now = ctx.currentTime + delay

  // the "crack" — very short filtered noise burst
  const bufferSize = Math.floor(ctx.sampleRate * 0.05)
  const buffer = ctx.createBuffer(1, bufferSize, ctx.sampleRate)
  const data = buffer.getChannelData(0)
  for (let i = 0; i < bufferSize; i++) {
    data[i] = (Math.random() * 2 - 1) * Math.pow(1 - i / bufferSize, 0.3)
  }
  const noise = ctx.createBufferSource()
  noise.buffer = buffer
  const noiseFilter = ctx.createBiquadFilter()
  noiseFilter.type = 'highpass'
  noiseFilter.frequency.value = 1200
  const noiseGain = ctx.createGain()
  noiseGain.gain.setValueAtTime(0.4, now)
  noiseGain.gain.exponentialRampToValueAtTime(0.001, now + 0.06)
  noise.connect(noiseFilter).connect(noiseGain).connect(ctx.destination)
  noise.start(now)

  // the "thud" — quick pitch drop, gives it body
  const osc = ctx.createOscillator()
  const oscGain = ctx.createGain()
  osc.type = 'triangle'
  osc.frequency.setValueAtTime(900, now)
  osc.frequency.exponentialRampToValueAtTime(90, now + 0.09)
  oscGain.gain.setValueAtTime(0.28, now)
  oscGain.gain.exponentialRampToValueAtTime(0.001, now + 0.1)
  osc.connect(oscGain).connect(ctx.destination)
  osc.start(now)
  osc.stop(now + 0.11)
}

/** A firework: low boom thump + crackling burst + a few high sparkle pings trailing off. */
function playFireworkBurst(delay = 0, big = false) {
  const ctx = getAudioCtx()
  if (!ctx) return
  const now = ctx.currentTime + delay

  // low "boom" thump
  const boom = ctx.createOscillator()
  const boomGain = ctx.createGain()
  boom.type = 'sine'
  boom.frequency.setValueAtTime(big ? 130 : 95, now)
  boom.frequency.exponentialRampToValueAtTime(28, now + (big ? 0.35 : 0.24))
  boomGain.gain.setValueAtTime(big ? 0.5 : 0.32, now)
  boomGain.gain.exponentialRampToValueAtTime(0.001, now + (big ? 0.4 : 0.28))
  boom.connect(boomGain).connect(ctx.destination)
  boom.start(now)
  boom.stop(now + 0.42)

  // crackling burst — the "sparkle explosion"
  const bufferSize = Math.floor(ctx.sampleRate * 0.5)
  const buffer = ctx.createBuffer(1, bufferSize, ctx.sampleRate)
  const data = buffer.getChannelData(0)
  for (let i = 0; i < bufferSize; i++) {
    data[i] = (Math.random() * 2 - 1) * Math.pow(1 - i / bufferSize, 1.6)
  }
  const noise = ctx.createBufferSource()
  noise.buffer = buffer
  const filter = ctx.createBiquadFilter()
  filter.type = 'bandpass'
  filter.frequency.value = 1400 + Math.random() * 1000
  filter.Q.value = 0.6
  const gain = ctx.createGain()
  gain.gain.setValueAtTime(big ? 0.4 : 0.26, now + 0.02)
  gain.gain.exponentialRampToValueAtTime(0.001, now + 0.5)
  noise.connect(filter).connect(gain).connect(ctx.destination)
  noise.start(now + 0.02)

  // a few bright random sparkle "pings" trailing after the boom
  const pingCount = big ? 5 : 3
  for (let p = 0; p < pingCount; p++) {
    const pingDelay = now + 0.08 + p * (0.05 + Math.random() * 0.05)
    const ping = ctx.createOscillator()
    const pingGain = ctx.createGain()
    ping.type = 'sine'
    ping.frequency.setValueAtTime(2200 + Math.random() * 1800, pingDelay)
    pingGain.gain.setValueAtTime(0.05, pingDelay)
    pingGain.gain.exponentialRampToValueAtTime(0.0001, pingDelay + 0.15)
    ping.connect(pingGain).connect(ctx.destination)
    ping.start(pingDelay)
    ping.stop(pingDelay + 0.16)
  }
}

/** A warm music-box/bell tone, used for the "Happy Birthday" melody. */
function playMelodyNote(freq: number, delay = 0, duration = 0.34, volume = 0.17) {
  const ctx = getAudioCtx()
  if (!ctx) return
  const now = ctx.currentTime + delay

  const osc = ctx.createOscillator()
  const osc2 = ctx.createOscillator()
  const gain = ctx.createGain()

  osc.type = 'triangle'
  osc2.type = 'sine'
  osc.frequency.setValueAtTime(freq, now)
  osc2.frequency.setValueAtTime(freq * 2, now) // octave shimmer, like a music box

  gain.gain.setValueAtTime(0.0001, now)
  gain.gain.linearRampToValueAtTime(volume, now + 0.02)
  gain.gain.exponentialRampToValueAtTime(0.0001, now + duration)

  osc.connect(gain)
  osc2.connect(gain)
  gain.connect(ctx.destination)

  osc.start(now)
  osc2.start(now)
  osc.stop(now + duration + 0.05)
  osc2.stop(now + duration + 0.05)
}

/**
 * Plays a short instrumental rendition of the classic "Happy Birthday to
 * You" tune (the melody itself has long been in the public domain) as a
 * music-box-style chime sequence. Returns the time (relative delay, in
 * seconds) at which the tune finishes, so other effects can be timed
 * to land right after it.
 */
function playHappyBirthdayTune(startDelay = 0): number {
  const C4 = 261.63,
    D4 = 293.66,
    E4 = 329.63,
    G4 = 392.0,
    A4 = 440.0,
    B4 = 493.88,
    C5 = 523.25,
    D5 = 587.33,
    E5 = 659.25,
    F5 = 698.46,
    G5 = 783.99

  const step = 0.26 // eighth-note spacing
  const long = 0.5 // held final note of each phrase

  const notes: { freq: number; dur: number }[] = [
    // "Hap-py Birth-day to you"
    { freq: G4, dur: step * 0.6 },
    { freq: G4, dur: step * 0.4 },
    { freq: A4, dur: step },
    { freq: G4, dur: step },
    { freq: C5, dur: step },
    { freq: B4, dur: long },
    // "Hap-py Birth-day to you"
    { freq: G4, dur: step * 0.6 },
    { freq: G4, dur: step * 0.4 },
    { freq: A4, dur: step },
    { freq: G4, dur: step },
    { freq: D5, dur: step },
    { freq: C5, dur: long },
    // "Hap-py Birth-day dear Grace"
    { freq: G4, dur: step * 0.6 },
    { freq: G4, dur: step * 0.4 },
    { freq: G5, dur: step },
    { freq: E5, dur: step },
    { freq: C5, dur: step },
    { freq: B4, dur: step },
    { freq: A4, dur: long },
    // "Hap-py Birth-day to you"
    { freq: F5, dur: step * 0.6 },
    { freq: F5, dur: step * 0.4 },
    { freq: E5, dur: step },
    { freq: C5, dur: step },
    { freq: D5, dur: step },
    { freq: C5, dur: long * 1.3 },
  ]

  let t = startDelay
  notes.forEach((n) => {
    playMelodyNote(n.freq, t, n.dur, 0.17)
    t += n.dur
  })
  return t
}

/** A swelling crowd "cheer" — filtered noise sweeping upward, like a room full of people going "yaaay!" */
function playCrowdCheer(delay = 0) {
  const ctx = getAudioCtx()
  if (!ctx) return
  const now = ctx.currentTime + delay

  const bufferSize = Math.floor(ctx.sampleRate * 1.1)
  const buffer = ctx.createBuffer(1, bufferSize, ctx.sampleRate)
  const data = buffer.getChannelData(0)
  for (let i = 0; i < bufferSize; i++) {
    data[i] = Math.random() * 2 - 1
  }
  const noise = ctx.createBufferSource()
  noise.buffer = buffer

  const filter = ctx.createBiquadFilter()
  filter.type = 'bandpass'
  filter.Q.value = 0.9
  filter.frequency.setValueAtTime(500, now)
  filter.frequency.linearRampToValueAtTime(2200, now + 0.5)
  filter.frequency.linearRampToValueAtTime(1400, now + 1.0)

  const gain = ctx.createGain()
  gain.gain.setValueAtTime(0.0001, now)
  gain.gain.linearRampToValueAtTime(0.14, now + 0.35)
  gain.gain.linearRampToValueAtTime(0.1, now + 0.7)
  gain.gain.exponentialRampToValueAtTime(0.0001, now + 1.1)

  noise.connect(filter).connect(gain).connect(ctx.destination)
  noise.start(now)
  noise.stop(now + 1.15)
}

/** A confetti-cannon "pop-fizz" — sharp burst plus a few twinkling high pings, like a party popper going off. */
function playConfettiPopper(delay = 0) {
  const ctx = getAudioCtx()
  if (!ctx) return
  const now = ctx.currentTime + delay

  const bufferSize = Math.floor(ctx.sampleRate * 0.12)
  const buffer = ctx.createBuffer(1, bufferSize, ctx.sampleRate)
  const data = buffer.getChannelData(0)
  for (let i = 0; i < bufferSize; i++) {
    data[i] = (Math.random() * 2 - 1) * Math.pow(1 - i / bufferSize, 0.5)
  }
  const noise = ctx.createBufferSource()
  noise.buffer = buffer
  const filter = ctx.createBiquadFilter()
  filter.type = 'highpass'
  filter.frequency.value = 2500
  const gain = ctx.createGain()
  gain.gain.setValueAtTime(0.3, now)
  gain.gain.exponentialRampToValueAtTime(0.001, now + 0.14)
  noise.connect(filter).connect(gain).connect(ctx.destination)
  noise.start(now)

  // trailing sparkle pings, like confetti twinkling down
  for (let p = 0; p < 4; p++) {
    const pingDelay = now + 0.05 + p * 0.07
    const ping = ctx.createOscillator()
    const pingGain = ctx.createGain()
    ping.type = 'triangle'
    ping.frequency.setValueAtTime(1800 + Math.random() * 1400, pingDelay)
    pingGain.gain.setValueAtTime(0.06, pingDelay)
    pingGain.gain.exponentialRampToValueAtTime(0.0001, pingDelay + 0.18)
    ping.connect(pingGain).connect(ctx.destination)
    ping.start(pingDelay)
    ping.stop(pingDelay + 0.2)
  }
}

function playCelebrationSounds() {
  // opening fanfare — bright sparkle + a festive double horn toot + a popper
  playChimeArpeggio(0, true)
  playPartyHorn(0.3)
  playPartyHorn(0.62)
  playConfettiPopper(0.85)

  // the "Happy Birthday" tune, punctuated with party sounds at each line
  const tuneEnd = playHappyBirthdayTune(1.15)
  playBalloonPop(1.15 + 1.55)
  playFireworkBurst(1.15 + 3.1)
  playPartyHorn(1.15 + 3.3)
  playCrowdCheer(1.15 + 4.7)
  playFireworkBurst(1.15 + 4.9, true)

  // grand finale — poppers, a pop, a big firework, a horn, and a closing sparkle
  playConfettiPopper(tuneEnd + 0.1)
  playBalloonPop(tuneEnd + 0.3)
  playFireworkBurst(tuneEnd + 0.5, true)
  playPartyHorn(tuneEnd + 0.75)
  playChimeArpeggio(tuneEnd + 0.95, false)
}

/* ---------- celebration → then actually enter ---------- */

let celebrationTimer: ReturnType<typeof setTimeout> | undefined
let enterTimer: ReturnType<typeof setTimeout> | undefined

function triggerCelebration() {
  step.value = 'celebration'
  playCelebrationSounds()
  celebrationTimer = setTimeout(() => {
    flowersDone.value = true
    enterTimer = setTimeout(() => emit('enter'), 700)
  }, 8600)
}

function answerIdentity(ans: 'yes' | 'no') {
  if (ans === 'yes') step.value = 'color'
  else step.value = 'wrong-identity'
}

function answerColor(color: string) {
  if (color === 'pink') {
    step.value = 'blooming'
    bloomingIndex.value = 0
    if (bloomingTimer) clearInterval(bloomingTimer)
    bloomingTimer = setInterval(() => {
      bloomingIndex.value = (bloomingIndex.value + 1) % bloomingLines.length
    }, 1050)
    setTimeout(() => {
      if (bloomingTimer) {
        clearInterval(bloomingTimer)
        bloomingTimer = undefined
      }
      if (canEnter()) {
        triggerCelebration()
      } else {
        // not yet her birthday — show the locked/countdown card instead
        step.value = 'locked'
      }
    }, 3200)
  } else {
    step.value = 'wrong-color'
  }
}

/* ---------- countdown (shown on the locked step) ---------- */

const cd = reactive({ days: 0, hours: 0, minutes: 0, seconds: 0 })
let cdTimer: ReturnType<typeof setInterval> | undefined

function updateCountdown() {
  const target = new Date(herBirthday + 'T00:00:00').getTime()
  let diff = target - Date.now()
  if (diff < 0) diff = 0
  const day = 86400000,
    hour = 3600000,
    minute = 60000
  cd.days = Math.floor(diff / day)
  diff -= cd.days * day
  cd.hours = Math.floor(diff / hour)
  diff -= cd.hours * hour
  cd.minutes = Math.floor(diff / minute)
  diff -= cd.minutes * minute
  cd.seconds = Math.floor(diff / 1000)

  // if time passes while she's sitting on the locked screen, let her in
  if (step.value === 'locked' && isBirthdayReached()) {
    triggerCelebration()
  }
}

/* ---------- hidden admin unlock ---------- */

const showAdminInput = ref(false)
const adminPasswordInput = ref('')
const adminError = ref(false)

function toggleAdminInput() {
  showAdminInput.value = !showAdminInput.value
  adminError.value = false
  adminPasswordInput.value = ''
}

function submitAdminPassword() {
  if (adminPasswordInput.value === ADMIN_PASSWORD) {
    adminUnlocked.value = true
    showAdminInput.value = false
    adminPasswordInput.value = ''
    adminError.value = false

    // if she's currently stuck on the locked screen, let her through now
    if (step.value === 'locked') {
      triggerCelebration()
    }
  } else {
    adminError.value = true
  }
}

onMounted(() => {
  updateCountdown()
  cdTimer = setInterval(updateCountdown, 1000)

  romanticTimer = setInterval(() => {
    romanticIndex.value = (romanticIndex.value + 1) % romanticLines.length
  }, 4200)
})
onUnmounted(() => {
  if (cdTimer) clearInterval(cdTimer)
  if (celebrationTimer) clearTimeout(celebrationTimer)
  if (enterTimer) clearTimeout(enterTimer)
  if (romanticTimer) clearInterval(romanticTimer)
  if (bloomingTimer) clearInterval(bloomingTimer)
  if (audioCtx) {
    audioCtx.close().catch(() => {})
  }
})
</script>

<template>
  <div class="gate" :class="{ 'gate--fading': flowersDone }">
    <!-- soft night-sky ambiance: twinkling stars + drifting bokeh light -->
    <div class="stars-layer" aria-hidden="true">
      <span
        v-for="n in 44"
        :key="'star' + n"
        class="star"
        :style="{
          left: ((n * 13.7 + 3) % 100) + '%',
          top: ((n * 7.9 + 4) % 82) + '%',
          animationDelay: (n % 22) * 0.28 + 's',
          animationDuration: 2.2 + (n % 4) * 0.7 + 's',
          width: 1 + (n % 3) + 'px',
          height: 1 + (n % 3) + 'px',
        }"
      ></span>
    </div>
    <div class="bokeh-layer" aria-hidden="true">
      <span
        v-for="n in 6"
        :key="'bokeh' + n"
        class="bokeh"
        :style="{
          left: ((n * 17 + 4) % 88) + '%',
          top: ((n * 23 + 8) % 68) + '%',
          animationDelay: n * 1.3 + 's',
          animationDuration: 7 + (n % 3) + 's',
          width: 60 + (n % 3) * 30 + 'px',
          height: 60 + (n % 3) * 30 + 'px',
        }"
      ></span>
    </div>

    <div class="ambient-petals" aria-hidden="true">
      <span
        v-for="n in 18"
        :key="n"
        class="ap"
        :style="{
          left: ((n * 5.5) % 100) + '%',
          animationDuration: 8 + (n % 5) + 's',
          animationDelay: n * 0.6 + 's',
          fontSize: 0.9 + (n % 3) * 0.4 + 'rem',
        }"
        >🌸</span
      >
    </div>

    <!-- STEP 1: identity -->
    <transition name="fade-up" appear>
      <div v-if="step === 'identity'" class="card">
        <img :src="manPhotoSrc" alt="A handsome man" class="man" />

        <h1 class="question">Are you my love, Grace?</h1>
        <div class="btn-row">
          <button class="btn btn--yes" @click="answerIdentity('yes')">Yes, it's me! 💕</button>
          <button class="btn btn--no" @click="answerIdentity('no')">No</button>
        </div>
      </div>
    </transition>

    <!-- STEP 1b: said no -->
    <transition name="fade-up">
      <div v-if="step === 'wrong-identity'" class="card card--wrong">
        <div class="big-emoji">🤨</div>
        <p class="wrong-msg">
          Hmm… I don't think that's my Grace.<br />Only she gets to open this one. 💕
        </p>
        <button class="btn btn--retry" @click="step = 'identity'">Let me try again, love</button>
      </div>
    </transition>

    <!-- STEP 2: color -->
    <transition name="fade-up">
      <div v-if="step === 'color'" class="card">
        <div class="big-emoji">🎉</div>
        <h1 class="question" style="font-size: clamp(1.05rem, 3vw, 1.35rem)">
          Okay then — prove it.<br />What's your favorite color?
        </h1>
        <div class="color-grid">
          <button
            class="color-btn"
            style="background: #3a86ff; color: #fff"
            @click="answerColor('blue')"
          >
            Blue
          </button>
          <button class="color-btn color-btn--pink" @click="answerColor('pink')">Pink 🌸</button>
          <button
            class="color-btn"
            style="background: #8338ec; color: #fff"
            @click="answerColor('purple')"
          >
            Purple
          </button>
          <button
            class="color-btn"
            style="background: #fb5607; color: #fff"
            @click="answerColor('orange')"
          >
            Orange
          </button>
        </div>
      </div>
    </transition>

    <!-- STEP 2b: wrong color -->
    <transition name="fade-up">
      <div v-if="step === 'wrong-color'" class="card card--wrong">
        <div class="big-emoji">😏</div>
        <p class="wrong-msg">
          That's not quite it, sweetheart…<br />think of what makes you smile. 💭
        </p>
        <button class="btn btn--retry" @click="step = 'color'">One more guess, my love</button>
      </div>
    </transition>

    <!-- STEP 3: blooming -->
    <transition name="fade-up">
      <div v-if="step === 'blooming'" class="blooming-layer" aria-hidden="true">
        <div
          v-for="n in 48"
          :key="n"
          class="bloom-flower"
          :style="{
            left: ((n * 7.3 + 3) % 98) + '%',
            animationDelay: n * 0.055 + 's',
            fontSize: 1.2 + (n % 5) * 0.45 + 'rem',
            animationDuration: 2.4 + (n % 4) * 0.3 + 's',
          }"
        >
          🌸
        </div>
        <transition name="fade-caption" mode="out-in">
          <div class="blooming-text" :key="bloomingIndex">{{ bloomingLines[bloomingIndex] }}</div>
        </transition>
      </div>
    </transition>

    <!-- STEP 4: locked — correct answers, but not her birthday yet -->
    <transition name="fade-up">
      <div v-if="step === 'locked'" class="card card--locked">
        <div class="big-emoji locked-heart">💗</div>
        <h1 class="question" style="font-size: clamp(1.05rem, 3vw, 1.35rem)">
          It's really you 💕<br />
          But this gift isn't ready to open yet.
        </h1>
        <p class="locked-sub">It unlocks itself on your birthday. Come back then.</p>
        <div class="locked-counter">
          <div class="locked-counter-item">
            <span class="locked-num">{{ cd.days }}</span
            ><span class="locked-label">days</span>
          </div>
          <div class="locked-counter-item">
            <span class="locked-num">{{ cd.hours }}</span
            ><span class="locked-label">hrs</span>
          </div>
          <div class="locked-counter-item">
            <span class="locked-num">{{ cd.minutes }}</span
            ><span class="locked-label">min</span>
          </div>
          <div class="locked-counter-item">
            <span class="locked-num">{{ cd.seconds }}</span
            ><span class="locked-label">sec</span>
          </div>
        </div>
        <p class="locked-footer script">worth every second, my love</p>
      </div>
    </transition>

    <!-- STEP 5: celebration — HAPPY BIRTHDAY burst, right before entering -->
    <transition name="fade-up">
      <div v-if="step === 'celebration'" class="celebration-layer" aria-hidden="true">
        <!-- balloons rising -->
        <div
          v-for="n in 14"
          :key="'b' + n"
          class="cel-balloon"
          :style="{
            left: ((n * 7.1 + 2) % 96) + '%',
            animationDelay: n * 0.22 + 's',
            animationDuration: 4.2 + (n % 4) * 0.5 + 's',
            fontSize: 1.8 + (n % 3) * 0.5 + 'rem',
          }"
        >
          {{ ['🎈', '🎈', '🎈', '💗'][n % 4] }}
        </div>

        <!-- hearts drifting up -->
        <div
          v-for="n in 20"
          :key="'h' + n"
          class="cel-heart"
          :style="{
            left: ((n * 5.3 + 4) % 98) + '%',
            animationDelay: n * 0.15 + 's',
            animationDuration: 3.4 + (n % 5) * 0.4 + 's',
            fontSize: 1 + (n % 3) * 0.4 + 'rem',
          }"
        >
          {{ ['💕', '💖', '💗', '❤️'][n % 4] }}
        </div>

        <!-- flowers + confetti falling -->
        <div
          v-for="n in 40"
          :key="'f' + n"
          class="cel-fall"
          :style="{
            left: ((n * 6.7 + 1) % 99) + '%',
            animationDelay: n * 0.07 + 's',
            fontSize: 1 + (n % 4) * 0.4 + 'rem',
            animationDuration: 3 + (n % 4) * 0.35 + 's',
          }"
        >
          {{ ['🌸', '🌹', '🎉', '✨', '💐'][n % 5] }}
        </div>

        <div class="celebration-text">
          <span class="cel-line">HAPPY BIRTHDAY MY LOOVVEEE</span>
          <span class="cel-name">GRACE!🎂</span>
          <span class="cel-signature script">— Charls</span>
        </div>
      </div>
    </transition>

    <!-- a quiet, rotating little love note while she's deciding -->
    <transition name="fade-caption" mode="out-in">
      <p
        v-if="step === 'identity' || step === 'color'"
        class="romantic-caption script"
        :key="romanticIndex"
      >
        {{ romanticLines[romanticIndex] }}
      </p>
    </transition>

    <!-- tiny, easy-to-miss admin unlock, bottom-right corner -->
    <div class="admin-corner">
      <button class="admin-dot" @click="toggleAdminInput" aria-label="" tabindex="-1"></button>
      <div v-if="showAdminInput" class="admin-box">
        <input
          v-model="adminPasswordInput"
          type="password"
          class="admin-input"
          placeholder="•••••"
          @keyup.enter="submitAdminPassword"
          autofocus
        />
        <button class="admin-go" @click="submitAdminPassword">↵</button>
        <p v-if="adminError" class="admin-error">nope</p>
      </div>
    </div>
  </div>
</template>

<style scoped>
.gate {
  position: fixed;
  inset: 0;
  background: radial-gradient(ellipse at top, #b23368 0%, #4a0f30 65%, #2e0a1f 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
  overflow: hidden;
  transition: opacity 0.6s ease;
}
.gate--fading {
  opacity: 0;
  pointer-events: none;
}

/* ---------- soft night-sky ambiance ---------- */
.stars-layer {
  position: absolute;
  inset: 0;
  pointer-events: none;
  z-index: 0;
}
.star {
  position: absolute;
  border-radius: 50%;
  background: #f6d9c4;
  box-shadow: 0 0 4px rgba(246, 217, 196, 0.85);
  animation: starTwinkle ease-in-out infinite;
}
@keyframes starTwinkle {
  0%,
  100% {
    opacity: 0.15;
    transform: scale(0.75);
  }
  50% {
    opacity: 0.95;
    transform: scale(1.3);
  }
}
.bokeh-layer {
  position: absolute;
  inset: 0;
  pointer-events: none;
  z-index: 0;
  overflow: hidden;
}
.bokeh {
  position: absolute;
  border-radius: 50%;
  background: radial-gradient(circle, rgba(246, 217, 196, 0.22), transparent 72%);
  filter: blur(3px);
  animation: bokehDrift ease-in-out infinite alternate;
}
@keyframes bokehDrift {
  0% {
    transform: translate(0, 0) scale(1);
    opacity: 0.6;
  }
  100% {
    transform: translate(22px, -32px) scale(1.18);
    opacity: 0.95;
  }
}

.ambient-petals {
  position: absolute;
  inset: 0;
  pointer-events: none;
}
.ap {
  position: absolute;
  bottom: -40px;
  animation: apFloat linear infinite;
  opacity: 0.55;
}
@keyframes apFloat {
  0% {
    transform: translateY(0) rotate(0deg);
    opacity: 0;
  }
  10% {
    opacity: 0.55;
  }
  90% {
    opacity: 0.4;
  }
  100% {
    transform: translateY(-110vh) rotate(30deg);
    opacity: 0;
  }
}

.card {
  position: absolute;
  background: rgba(255, 243, 246, 0.1);
  border: 1px solid rgba(246, 217, 196, 0.35);
  backdrop-filter: blur(14px);
  -webkit-backdrop-filter: blur(14px);
  border-radius: 22px;
  padding: 2.2rem 2rem;
  width: min(420px, 88vw);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.2rem;
  text-align: center;
  box-shadow: 0 28px 56px rgba(0, 0, 0, 0.45);
}
/* shimmering gold-foil edge — a slowly rotating conic highlight
   masked so only a thin ring around the card catches the light */
.card::before {
  content: '';
  position: absolute;
  inset: -1px;
  border-radius: 22px;
  padding: 1px;
  background: conic-gradient(
    from 0deg,
    transparent 0%,
    rgba(246, 217, 196, 0.9) 8%,
    transparent 20%,
    transparent 100%
  );
  -webkit-mask:
    linear-gradient(#fff 0 0) content-box,
    linear-gradient(#fff 0 0);
  -webkit-mask-composite: xor;
  mask-composite: exclude;
  animation: foilSpin 7s linear infinite;
  pointer-events: none;
}
@keyframes foilSpin {
  to {
    transform: rotate(360deg);
  }
}
.card--wrong {
  background: rgba(140, 28, 77, 0.22);
}
.card--locked {
  background: rgba(140, 28, 77, 0.22);
}

.man {
  width: 180px;
  height: 180px;
  object-fit: cover;
  object-position: top center;
  border-radius: 50%;
  border: 3px solid rgba(246, 217, 196, 0.6);
  filter: drop-shadow(0 12px 24px rgba(0, 0, 0, 0.5));
  background: rgba(255, 255, 255, 0.08);
}

.question {
  font-family: 'Playfair Display', serif;
  font-size: clamp(1.25rem, 4vw, 1.65rem);
  color: #f6d9c4;
  margin: 0;
  line-height: 1.4;
}
.big-emoji {
  font-size: 3.5rem;
  line-height: 1;
}
.locked-heart {
  animation: heartBeat 2.2s ease-in-out infinite;
}
@keyframes heartBeat {
  0%,
  100% {
    transform: scale(1);
  }
  15% {
    transform: scale(1.14);
  }
  30% {
    transform: scale(1);
  }
  45% {
    transform: scale(1.1);
  }
  60% {
    transform: scale(1);
  }
}
.wrong-msg {
  font-family: 'Cormorant Garamond', serif;
  font-size: 1.18rem;
  color: #ffd6e4;
  margin: 0;
  line-height: 1.6;
}

.locked-sub {
  font-family: 'Cormorant Garamond', serif;
  font-size: 1.05rem;
  color: #ffd6e4;
  margin: 0;
  opacity: 0.85;
}
.locked-counter {
  display: flex;
  gap: 1.2rem;
  justify-content: center;
  margin-top: 0.4rem;
}
.locked-counter-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-width: 46px;
}
.locked-num {
  font-family: 'Playfair Display', serif;
  font-size: 1.6rem;
  color: #f6d9c4;
}
.locked-label {
  font-family: 'Jost', sans-serif;
  font-size: 0.62rem;
  text-transform: uppercase;
  letter-spacing: 0.12em;
  opacity: 0.7;
  color: #ffd6e4;
  margin-top: 0.15rem;
}
.locked-footer {
  font-size: 1.15rem;
  color: #f6d9c4;
  opacity: 0.85;
  margin: 0.2rem 0 0;
}

.btn-row {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
  justify-content: center;
}
.btn {
  font-family: 'Jost', sans-serif;
  font-size: 0.9rem;
  letter-spacing: 0.06em;
  border-radius: 30px;
  padding: 0.7rem 1.7rem;
  cursor: pointer;
  border: 1.5px solid rgba(246, 217, 196, 0.6);
  transition:
    transform 0.2s ease,
    box-shadow 0.2s ease;
}
.btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.28);
}
.btn--yes {
  background: linear-gradient(135deg, #b23368, #8c1c4d);
  color: #f6d9c4;
}
.btn--no {
  background: rgba(255, 255, 255, 0.1);
  color: #f6d9c4;
}
.btn--retry {
  background: linear-gradient(135deg, #b23368, #8c1c4d);
  color: #f6d9c4;
}

.color-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 0.9rem;
  width: 100%;
}
.color-btn {
  font-family: 'Jost', sans-serif;
  font-size: 0.95rem;
  letter-spacing: 0.04em;
  padding: 0.8rem 1rem;
  border-radius: 14px;
  border: none;
  cursor: pointer;
  font-weight: 500;
  transition:
    transform 0.2s ease,
    box-shadow 0.2s ease;
}
.color-btn:hover {
  transform: translateY(-3px) scale(1.04);
  box-shadow: 0 10px 24px rgba(0, 0, 0, 0.32);
}
.color-btn--pink {
  background: linear-gradient(135deg, #ff85b3, #e91e8c);
  color: #fff;
  border: 1.5px solid rgba(255, 180, 210, 0.6);
  animation: pinkPulse 2s ease-in-out infinite;
}
@keyframes pinkPulse {
  0%,
  100% {
    box-shadow: 0 0 18px rgba(233, 30, 140, 0.45);
  }
  50% {
    box-shadow: 0 0 30px rgba(233, 30, 140, 0.75);
  }
}

.blooming-layer {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-end;
  padding-bottom: 12vh;
  pointer-events: none;
}
.bloom-flower {
  position: absolute;
  top: -70px;
  animation: bloomFall ease-in forwards;
}
@keyframes bloomFall {
  0% {
    transform: translateY(0) rotate(0deg) scale(0.2);
    opacity: 0;
  }
  12% {
    opacity: 1;
    transform: translateY(8vh) rotate(15deg) scale(1);
  }
  100% {
    transform: translateY(118vh) rotate(80deg) scale(0.75);
    opacity: 0.3;
  }
}
.blooming-text {
  font-family: 'Dancing Script', cursive;
  font-size: clamp(1.5rem, 5vw, 2.1rem);
  color: #f6d9c4;
  text-shadow: 0 0 22px rgba(246, 217, 196, 0.7);
  animation: textGlow 1.4s ease-in-out infinite;
  position: relative;
  z-index: 2;
}
@keyframes textGlow {
  0%,
  100% {
    opacity: 0.8;
    text-shadow: 0 0 14px rgba(246, 217, 196, 0.5);
  }
  50% {
    opacity: 1;
    text-shadow: 0 0 28px rgba(246, 217, 196, 0.9);
  }
}

/* ---------- STEP 5: celebration ---------- */
.celebration-layer {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  pointer-events: none;
}

.cel-balloon {
  position: absolute;
  bottom: -60px;
  animation: celBalloonUp ease-in forwards;
  filter: drop-shadow(0 6px 10px rgba(0, 0, 0, 0.35));
}
@keyframes celBalloonUp {
  0% {
    transform: translateY(0) translateX(0) rotate(0deg);
    opacity: 0;
  }
  10% {
    opacity: 1;
  }
  100% {
    transform: translateY(-125vh) translateX(18px) rotate(8deg);
    opacity: 0.9;
  }
}

.cel-heart {
  position: absolute;
  bottom: -30px;
  animation: celHeartUp ease-in-out forwards;
}
@keyframes celHeartUp {
  0% {
    transform: translateY(0) translateX(0) scale(0.6);
    opacity: 0;
  }
  15% {
    opacity: 1;
  }
  100% {
    transform: translateY(-110vh) translateX(-14px) scale(1.1);
    opacity: 0;
  }
}

.cel-fall {
  position: absolute;
  top: -60px;
  animation: celFallDown ease-in forwards;
}
@keyframes celFallDown {
  0% {
    transform: translateY(0) rotate(0deg) scale(0.3);
    opacity: 0;
  }
  12% {
    opacity: 1;
    transform: translateY(10vh) rotate(20deg) scale(1);
  }
  100% {
    transform: translateY(120vh) rotate(90deg) scale(0.8);
    opacity: 0.25;
  }
}

.celebration-text {
  position: relative;
  z-index: 3;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.3rem;
  text-align: center;
  padding: 0 1rem;
  animation: celPop 0.7s cubic-bezier(0.2, 1.4, 0.4, 1) both;
}
.cel-line {
  font-family: 'Jost', sans-serif;
  letter-spacing: 0.18em;
  font-size: clamp(1rem, 4vw, 1.5rem);
  color: #ffd6e4;
  text-transform: uppercase;
}
.cel-name {
  font-family: 'Playfair Display', serif;
  font-size: clamp(2.4rem, 9vw, 4.2rem);
  color: #f6d9c4;
  text-shadow:
    0 0 20px rgba(246, 217, 196, 0.7),
    0 0 46px rgba(233, 30, 140, 0.55);
  animation: celGlow 1.3s ease-in-out infinite;
}
.cel-signature {
  font-size: clamp(1.15rem, 3.4vw, 1.55rem);
  color: #ffd6e4;
  opacity: 0.9;
  margin-top: 0.3rem;
}
@keyframes celPop {
  0% {
    opacity: 0;
    transform: scale(0.6) translateY(20px);
  }
  100% {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}
@keyframes celGlow {
  0%,
  100% {
    text-shadow:
      0 0 16px rgba(246, 217, 196, 0.55),
      0 0 34px rgba(233, 30, 140, 0.4);
  }
  50% {
    text-shadow:
      0 0 26px rgba(246, 217, 196, 0.9),
      0 0 58px rgba(233, 30, 140, 0.75);
  }
}

/* ---------- rotating little love note ---------- */
.romantic-caption {
  position: fixed;
  bottom: 16px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 1.05rem;
  color: rgba(246, 217, 196, 0.6);
  z-index: 150;
  pointer-events: none;
  text-align: center;
  padding: 0 1.2rem;
  max-width: 90vw;
}

.fade-up-enter-active {
  transition:
    opacity 0.5s ease,
    transform 0.5s ease;
}
.fade-up-leave-active {
  transition:
    opacity 0.28s ease,
    transform 0.28s ease;
}
.fade-up-enter-from {
  opacity: 0;
  transform: translateY(22px);
}
.fade-up-leave-to {
  opacity: 0;
  transform: translateY(-12px);
}
.fade-caption-enter-active,
.fade-caption-leave-active {
  transition: opacity 0.6s ease;
}
.fade-caption-enter-from,
.fade-caption-leave-to {
  opacity: 0;
}

/* ---------- hidden admin unlock, bottom-right ---------- */
.admin-corner {
  position: fixed;
  bottom: 10px;
  right: 10px;
  z-index: 200;
  display: flex;
  align-items: flex-end;
  gap: 0.4rem;
}
.admin-dot {
  width: 9px;
  height: 9px;
  border-radius: 50%;
  background: rgba(246, 217, 196, 0.25);
  border: none;
  padding: 0;
  cursor: default;
  opacity: 0.4;
  transition: opacity 0.2s ease;
}
.admin-dot:hover {
  opacity: 0.85;
  cursor: pointer;
}
.admin-box {
  display: flex;
  align-items: center;
  gap: 0.3rem;
  background: rgba(30, 6, 20, 0.85);
  border: 1px solid rgba(246, 217, 196, 0.25);
  border-radius: 20px;
  padding: 0.25rem 0.4rem 0.25rem 0.7rem;
  backdrop-filter: blur(6px);
  position: relative;
}
.admin-input {
  background: transparent;
  border: none;
  outline: none;
  color: #f6d9c4;
  font-family: 'Jost', sans-serif;
  font-size: 0.72rem;
  width: 72px;
}
.admin-input::placeholder {
  color: rgba(246, 217, 196, 0.35);
}
.admin-go {
  background: none;
  border: none;
  color: rgba(246, 217, 196, 0.6);
  font-size: 0.85rem;
  padding: 0.1rem 0.3rem;
}
.admin-go:hover {
  color: #f6d9c4;
}
.admin-error {
  position: absolute;
  bottom: -1.2rem;
  right: 0;
  font-size: 0.62rem;
  color: #ff9db8;
  font-family: 'Jost', sans-serif;
  margin: 0;
  opacity: 0.8;
}

@media (max-width: 400px) {
  .man {
    width: 150px;
    height: 150px;
  }
  .card {
    padding: 1.8rem 1.4rem;
    gap: 1rem;
  }
}
</style>
