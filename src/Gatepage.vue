<script setup lang="ts">
import { ref, reactive, onMounted, onUnmounted } from 'vue'

const emit = defineEmits<{ (e: 'enter'): void }>()

/* ============================================================
   ✏️  EDIT THESE
   ============================================================ */

// Must match the birthday used on the main page's countdown.
const herBirthday = '2026-08-20'

// The image of him.
const manPhotoSrc = '/man-photo.png'

// Admin bypass password — typed into the tiny dot in the corner.
const ADMIN_PASSWORD = 'ttoapril30'

/* ============================================================
   State
   ============================================================ */

type Step = 'identity' | 'wrong-identity' | 'color' | 'wrong-color' | 'blooming' | 'locked'

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

function answerIdentity(ans: 'yes' | 'no') {
  if (ans === 'yes') step.value = 'color'
  else step.value = 'wrong-identity'
}

function answerColor(color: string) {
  if (color === 'pink') {
    step.value = 'blooming'
    setTimeout(() => {
      if (canEnter()) {
        flowersDone.value = true
        setTimeout(() => emit('enter'), 600)
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
    flowersDone.value = true
    setTimeout(() => emit('enter'), 600)
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
      flowersDone.value = true
      setTimeout(() => emit('enter'), 600)
    }
  } else {
    adminError.value = true
  }
}

onMounted(() => {
  updateCountdown()
  cdTimer = setInterval(updateCountdown, 1000)
})
onUnmounted(() => {
  if (cdTimer) clearInterval(cdTimer)
})
</script>

<template>
  <div class="gate" :class="{ 'gate--fading': flowersDone }">
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
        <p class="wrong-msg">Hmm… I don't believe you.<br />Only Gace can open this.</p>
        <button class="btn btn--retry" @click="step = 'identity'">Let me try again</button>
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
        <p class="wrong-msg">That doesn't sound right…<br />Think harder, Gace. 💭</p>
        <button class="btn btn--retry" @click="step = 'color'">Oops, let me pick again</button>
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
        <div class="blooming-text">Opening your gift, Gace… 🌹</div>
      </div>
    </transition>

    <!-- STEP 4: locked — correct answers, but not her birthday yet -->
    <transition name="fade-up">
      <div v-if="step === 'locked'" class="card card--locked">
        <div class="big-emoji">🎁</div>
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
      </div>
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
