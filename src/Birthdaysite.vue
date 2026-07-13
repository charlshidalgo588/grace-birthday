<script setup lang="ts">
import { ref, reactive, computed, onMounted, onUnmounted, nextTick } from 'vue'

interface Photo {
  src: string
  caption: string
  liked?: boolean
}

interface Milestone {
  date: string
  title: string
  text: string
}

const herName = 'Graceyy'
const yourName = 'Charls'
const herAge = 22

const message = [
  `My dearest Love ${herName},`,
  ``,
  `I know letters are usually written with paper and pen, but even though I'm not writing this one by hand, I promise that every word comes straight from my heart.`,
  `We may be far apart, but I hope this letter finds you with a smile and makes your day a little more special.`,
  `It may not be the best gift you receive today, but I hope you'll cherish it because every word is sincere, genuine, and written just for you.`,
  `Just like you, these words are truly special.`,
  `I am forever grateful to have you in my life, love. I will never get tired of telling you that. I am always here to support and cheer for you, to love and care for you, to fight and get things fixed with you, as I want to live my life only with you.`,
  `Even though we're apart right now, my heart will always be close to you. Sooner or later, we'll be together again, and I promise we'll do all the things we used to enjoy—the food we love, the laughter we share, the warm hugs and sweet kisses, and my favorite part: listening to all your stories. (But I like more what you are thinking rn 🤪)`,
  `By the way, you're absolutely adorable whenever you tell me your stories. I could listen to you for hours. Hehe.`,
  `I hope you always stay the wonderful person you are, my love, because that's what makes you so special—simply being yourself.`,
  `Please never lose your kind heart, your generosity, your purity, and the love you so freely give to the people around you. Those beautiful qualities are what make you truly one of a kind, and they're some of the many reasons I love you so much.`,
  `Thank you for being here always for me, my love, for staying, especailly in hard times. I appreciate you so much, and I love you so much.`,
  `I promise to work hard for us and if someday I get things right, I will give you the life and happiness you deserve, love.`,
  `Today is your day, love, I hope you feel just how loved and appreciated you are. My wish for you is simple: that this year brings you closer to everything you've been praying for. And I hope I get to be right beside you through it all—celebrating your wins, holding your hand through the hard days, and making more beautiful memories with you.`,
  `Happy Birthday, my love. I love you more than this little letter could ever put into words. No matter how much I write, it will never be enough to show you just how much you mean to me.`,
  `Forever yours, ${yourName}`,
]

const wishes = [
  'I hope this year brings you endless happiness and opportunities.',
  'I hope you continue chasing your dreams because I know you can achieve them.',
  'Thank you for being such a kind, genuine, and beautiful soul.',
  'I hope you always remember how loved and appreciated you are.',
  'No matter how far apart we are, I hope you always feel my love for you.',
  'Happy Birthday, my love. I hope today is as amazing as you are. ❤️',
]

const photos = reactive<Photo[]>([
  { src: `${import.meta.env.BASE_URL}pics/2.jpg`, caption: '' },
  { src: `${import.meta.env.BASE_URL}pics/1.jpg`, caption: '' },
  { src: `${import.meta.env.BASE_URL}pics/3.jpg`, caption: '' },
  { src: `${import.meta.env.BASE_URL}pics/4.jpg`, caption: '' },
  { src: `${import.meta.env.BASE_URL}pics/5.jpeg`, caption: '' },
  { src: `${import.meta.env.BASE_URL}pics/6.jpeg`, caption: '' },
  { src: `${import.meta.env.BASE_URL}pics/7.jpeg`, caption: '' },
  { src: `${import.meta.env.BASE_URL}pics/8.jpeg`, caption: '' },
  { src: `${import.meta.env.BASE_URL}pics/9.jpeg`, caption: '' },
  { src: `${import.meta.env.BASE_URL}pics/10.jpg`, caption: '' },
  { src: `${import.meta.env.BASE_URL}pics/11.jpeg`, caption: '' },
  { src: `${import.meta.env.BASE_URL}pics/12.jpeg`, caption: '' },
])

/* ============================================================
   📷 GALLERY INTERACTIVITY — per-photo "love" hearts, a subtle
   pointer-driven tilt, and a full lightbox with prev/next nav,
   keyboard arrows, swipe-to-browse, and double-tap-to-like.
   ============================================================ */
const cardTilt = reactive<{ rx: number; ry: number; scale: number }[]>(
  photos.map(() => ({ rx: 0, ry: 0, scale: 1 })),
)

function handleTilt(e: MouseEvent, i: number) {
  const el = e.currentTarget as HTMLElement
  const rect = el.getBoundingClientRect()
  const px = (e.clientX - rect.left) / rect.width - 0.5
  const py = (e.clientY - rect.top) / rect.height - 0.5
  cardTilt[i] = { rx: py * -9, ry: px * 11, scale: 1.045 }
}
function resetTilt(i: number) {
  cardTilt[i] = { rx: 0, ry: 0, scale: 1 }
}
function polaroidTransform(i: number) {
  const base = tilts[i % tilts.length]
  const t = cardTilt[i] ?? { rx: 0, ry: 0, scale: 1 }
  return `rotate(${base}deg) scale(${t.scale}) rotateX(${t.rx}deg) rotateY(${t.ry}deg)`
}

const heartCount = computed(() => photos.filter((p) => p.liked).length)

/* small floating hearts that burst from wherever you tapped "like" */
const loveBursts = ref<{ id: string; x: number; y: number; glyph: string }[]>([])
function burstHeartsAt(evt?: MouseEvent | { clientX: number; clientY: number }) {
  const x = evt?.clientX ?? window.innerWidth / 2
  const y = evt?.clientY ?? window.innerHeight / 2
  const batch = []
  for (let i = 0; i < 8; i++) {
    batch.push({
      id: `${Date.now()}-${i}-${Math.random()}`,
      x: x + (Math.random() * 70 - 35),
      y: y + (Math.random() * 30 - 15),
      glyph: i % 3 === 0 ? '💛' : '♥',
    })
  }
  loveBursts.value.push(...batch)
  const ids = new Set(batch.map((b) => b.id))
  setTimeout(() => {
    loveBursts.value = loveBursts.value.filter((h) => !ids.has(h.id))
  }, 900)
}

function toggleLike(photo: Photo, evt?: MouseEvent) {
  photo.liked = !photo.liked
  if (photo.liked) burstHeartsAt(evt)
}

const lightboxIndex = ref(-1)
const lightboxPhoto = computed<Photo | null>(() =>
  lightboxIndex.value >= 0 ? (photos[lightboxIndex.value] ?? null) : null,
)

function openLightbox(i: number) {
  lightboxIndex.value = i
}
function closeLightbox() {
  lightboxIndex.value = -1
}
function nextPhoto() {
  if (lightboxIndex.value < 0) return
  lightboxIndex.value = (lightboxIndex.value + 1) % photos.length
}
function prevPhoto() {
  if (lightboxIndex.value < 0) return
  lightboxIndex.value = (lightboxIndex.value - 1 + photos.length) % photos.length
}
function onLightboxKeydown(e: KeyboardEvent) {
  if (lightboxIndex.value < 0) return
  if (e.key === 'ArrowRight') nextPhoto()
  else if (e.key === 'ArrowLeft') prevPhoto()
  else if (e.key === 'Escape') closeLightbox()
}

let lightboxTouchStartX = 0
function onLightboxTouchStart(e: TouchEvent) {
  lightboxTouchStartX = e.changedTouches[0]?.clientX ?? 0
}
function onLightboxTouchEnd(e: TouchEvent) {
  const endX = e.changedTouches[0]?.clientX ?? 0
  const delta = endX - lightboxTouchStartX
  if (Math.abs(delta) > 50) {
    if (delta < 0) nextPhoto()
    else prevPhoto()
  }
}

const mainVideo = reactive({
  src: `${import.meta.env.BASE_URL}vids/bdayvidgrace.mp4`,
  poster: '',
  caption:
    "A little something I put together. I hope you appreciate this small effort of mine and, even though we're miles apart, that you can still feel how much I love you.",
})

const musicUrl = `${import.meta.env.BASE_URL}music/Bruno Major - Nothing (Lyric Chord Video).mp3`
const wishText = `Whatever you wished for, I hope it finds you exactly when you need it. And if nothing crossed your mind — I'm already wishing it for you.`

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

const fillers = [
  { x: 58, y: 108, scale: 0.55, rotate: 8 },
  { x: 242, y: 106, scale: 0.55, rotate: -12 },
  { x: 38, y: 168, scale: 0.5, rotate: 20 },
  { x: 262, y: 166, scale: 0.5, rotate: -18 },
  { x: 150, y: 56, scale: 0.55, rotate: 0 },
  { x: 108, y: 68, scale: 0.42, rotate: -14 },
  { x: 192, y: 66, scale: 0.42, rotate: 14 },
  { x: 74, y: 92, scale: 0.4, rotate: 22 },
  { x: 226, y: 90, scale: 0.4, rotate: -22 },
  { x: 128, y: 168, scale: 0.38, rotate: -8 },
  { x: 172, y: 168, scale: 0.38, rotate: 8 },
]

/* ============================================================
   🌷 EXTRA BOUQUET BLOOMS — tulips and daisies woven in among
   the roses, plus a few twinkling sparkles and a little hanging
   note tag, so the bouquet feels like a real, full arrangement
   and not just one flower repeated.
   ============================================================ */
const tulipPetalPath = 'M0,0 C-9,-6 -11,-24 0,-34 C11,-24 9,-6 0,0 Z'
const daisyPetalPath = 'M0,0 C-3,-3 -3,-16 0,-20 C3,-16 3,-3 0,0 Z'

const tulips = [
  {
    x: 118,
    y: 150,
    scale: 0.85,
    rotate: -10,
    front: 'var(--blush)',
    side: 'var(--pink-mid)',
    stem: 'M150,300 C136,262 122,208 118,168',
  },
  {
    x: 182,
    y: 148,
    scale: 0.85,
    rotate: 10,
    front: '#fff2f6',
    side: 'var(--blush)',
    stem: 'M150,300 C164,262 178,206 182,166',
  },
  {
    x: 150,
    y: 182,
    scale: 0.72,
    rotate: 0,
    front: 'var(--pink-mid)',
    side: 'var(--wine-soft)',
    stem: 'M150,300 C150,262 150,214 150,196',
  },
]

const daisies = [
  { x: 80, y: 198, scale: 0.6, rotate: -12, stem: 'M150,300 C120,276 96,238 80,216' },
  { x: 220, y: 196, scale: 0.6, rotate: 14, stem: 'M150,300 C180,276 204,236 220,214' },
  { x: 100, y: 140, scale: 0.5, rotate: 20, stem: 'M150,300 C128,266 112,206 100,158' },
  { x: 200, y: 138, scale: 0.5, rotate: -18, stem: 'M150,300 C172,266 188,204 200,156' },
]

const bouquetSparkles = [
  { x: 70, y: 78, r: 2, delay: 0 },
  { x: 232, y: 88, r: 1.6, delay: 0.45 },
  { x: 150, y: 36, r: 2.2, delay: 0.9 },
  { x: 58, y: 152, r: 1.8, delay: 1.3 },
  { x: 242, y: 148, r: 1.6, delay: 0.2 },
  { x: 112, y: 58, r: 1.4, delay: 0.65 },
  { x: 190, y: 56, r: 1.4, delay: 1.1 },
]

/* ============================================================
   💌 TITO JOKES — "Reveal punchline"
   A little corny game: each card teases a setup, she taps to
   reveal the (groan-worthy, sweet) punchline. Collect them all
   to unlock a bonus message.
   ✏️ Edit the teaser/line pairs to match your own sense of humor.
   ============================================================ */
interface TitoJoke {
  teaser: string
  line: string
  icon: string
}

const titoJokes = reactive<TitoJoke[]>([
  {
    teaser: "Why don't scientists trust atoms anymore?",
    line: 'Because they make up literally everything — kind of like how you make up my entire world. 😂❤️',
    icon: '⚛️',
  },
  {
    teaser: 'What did the ocean say to the shore?',
    line: 'Nothing, love — it just waved. 🌊 Kind of like how you wave hello and my whole heart grins like an idiot.',
    icon: '🌊',
  },
  {
    teaser: 'Why did the coffee file a police report?',
    line: 'It got mugged! ☕ The only thing I want stolen around here is one more kiss from you.',
    icon: '☕',
  },
  {
    teaser: "Why don't eggs ever tell jokes?",
    line: "They'd crack each other up. 🥚 You crack me up too — in the best, most annoying-in-a-cute-way possible.",
    icon: '🥚',
  },
  {
    teaser: "What do you call cheese that isn't yours?",
    line: 'Nacho cheese! 🧀 …unlike my heart, which has always, only ever been yours.',
    icon: '🧀',
  },
  {
    teaser: "What's a tito's favorite kind of joke?",
    line: 'The corny kind — a good pun is its own re-word. 😅 And loving you? Best reward I never even had to work for.',
    icon: '😂',
  },
])

const jokeIndex = ref(0)
const jokeRevealed = ref(false)
const jokesComplete = ref(false)
const collectedIcons = reactive<string[]>([])

// TS's noUncheckedIndexedAccess flags titoJokes[jokeIndex.value] as possibly
// undefined even though the index is always in bounds by construction —
// this computed gives every reader a guaranteed, non-undefined joke.
const currentJoke = computed<TitoJoke>(
  () => titoJokes[jokeIndex.value] ?? { teaser: '', line: '', icon: '' },
)

function revealJoke() {
  if (jokeRevealed.value) return
  jokeRevealed.value = true
  collectedIcons.push(currentJoke.value.icon)
  burstPetals(50)
}

function nextJoke() {
  if (jokeIndex.value < titoJokes.length - 1) {
    jokeIndex.value++
    jokeRevealed.value = false
  } else {
    jokesComplete.value = true
    burstPetals(46)
  }
}

function resetJokes() {
  jokeIndex.value = 0
  jokeRevealed.value = false
  jokesComplete.value = false
  collectedIcons.splice(0, collectedIcons.length)
}

/* ============================================================
   💛 A little running tally of all the "love" collected across
   the page — gallery hearts, joke icons, and hero taps — shown
   quietly in the footer as one sweet closing touch.
   ============================================================ */
const loveTapCount = ref(0)
const loveTapMessages = [
  "That's one. ❤️",
  'Okay, I felt that one.',
  "You're making my heart race.",
  "Careful — it's already full of you.",
  'I love you a little more with every tap, I promise.',
  'Okay okay — infinite loop initiated. I love you forever.',
]
const loveTapMessage = computed(() => {
  const idx = Math.min(loveTapCount.value, loveTapMessages.length) - 1
  return idx >= 0 ? loveTapMessages[idx] : ''
})
function tapLove(e: MouseEvent) {
  loveTapCount.value++
  burstHeartsAt(e)
}

const totalLoveCollected = computed(
  () =>
    heartCount.value + collectedIcons.length + Math.min(loveTapCount.value, loveTapMessages.length),
)

/* ============================================================
   🎥 REACTION CAM — screen + webcam, combined
   Opt-in recorder. She taps one button, grants TWO permissions
   (share this tab/screen, then camera+mic), and we composite
   both feeds live onto a hidden canvas: the page behind her,
   her reaction as a small circular bubble in the corner — the
   same way a "reaction video" looks. A small live self-view
   bubble stays on screen while she scrolls, she taps stop
   whenever she's done, then can preview and save the single
   combined clip to her own device. Fully client-side — nothing
   is uploaded anywhere.
   ============================================================ */
const cameraSupported = ref(
  typeof navigator !== 'undefined' &&
    !!navigator.mediaDevices?.getUserMedia &&
    !!navigator.mediaDevices?.getDisplayMedia &&
    typeof MediaRecorder !== 'undefined',
)
const cameraStage = ref<'idle' | 'recording' | 'reviewing'>('idle')
const cameraStream = ref<MediaStream | null>(null) // webcam + mic
const screenStream = ref<MediaStream | null>(null) // shared tab/screen
const cameraError = ref('')
const recordSeconds = ref(0)
const recordedUrl = ref('')
const bubblePreview = ref<HTMLVideoElement | null>(null)

let mediaRecorder: MediaRecorder | null = null
let recordedChunks: BlobPart[] = []
let recordTimerId: ReturnType<typeof setInterval> | null = null
const MAX_RECORD_SECONDS = 15 * 60 // 15 min safety cap while she browses

// off-DOM elements used purely to feed the compositor canvas
let screenVideoEl: HTMLVideoElement | null = null
let webcamVideoEl: HTMLVideoElement | null = null
let mixCanvas: HTMLCanvasElement | null = null
let mixCtx: CanvasRenderingContext2D | null = null
let mixRAF = 0

async function startReactionRecording() {
  cameraError.value = ''
  try {
    // 1. ask her to share the tab/screen so we can record what she's browsing
    const display = await navigator.mediaDevices.getDisplayMedia({
      video: { frameRate: 30 },
      audio: true,
    })
    screenStream.value = display

    // 2. then ask for camera + mic so we can capture her reaction on top
    const cam = await navigator.mediaDevices.getUserMedia({ video: true, audio: true })
    cameraStream.value = cam

    cameraStage.value = 'recording'
    await nextTick()

    // live self-view bubble, mirrored, so she can see herself while browsing
    if (bubblePreview.value) {
      bubblePreview.value.srcObject = cam
      await bubblePreview.value.play().catch(() => {})
    }

    setupCompositor(display, cam)
    beginRecorder()

    // if she stops sharing from the browser's native "Stop sharing" bar
    // instead of our button, end the recording gracefully
    display.getVideoTracks()[0]?.addEventListener('ended', () => {
      if (cameraStage.value === 'recording') stopReactionRecording()
    })
  } catch (err) {
    console.error('Screen/camera permission failed:', err)
    cleanupStreams()
    cameraStage.value = 'idle'
    cameraError.value =
      "Couldn't start recording — this needs permission to share this tab AND to use your camera. Try again and allow both."
  }
}

// draws the shared screen as the background and the webcam as a small
// mirrored circular bubble on top, every frame, onto a hidden canvas
function setupCompositor(display: MediaStream, cam: MediaStream) {
  screenVideoEl = document.createElement('video')
  screenVideoEl.muted = true
  screenVideoEl.playsInline = true
  screenVideoEl.srcObject = display
  screenVideoEl.play().catch(() => {})

  webcamVideoEl = document.createElement('video')
  webcamVideoEl.muted = true
  webcamVideoEl.playsInline = true
  webcamVideoEl.srcObject = cam
  webcamVideoEl.play().catch(() => {})

  mixCanvas = document.createElement('canvas')
  mixCanvas.width = 1280
  mixCanvas.height = 720
  mixCtx = mixCanvas.getContext('2d')

  const draw = () => {
    if (!mixCtx || !mixCanvas) return
    const w = mixCanvas.width
    const h = mixCanvas.height

    if (screenVideoEl && screenVideoEl.readyState >= 2) {
      mixCtx.drawImage(screenVideoEl, 0, 0, w, h)
    } else {
      mixCtx.fillStyle = '#0a0210'
      mixCtx.fillRect(0, 0, w, h)
    }

    if (webcamVideoEl && webcamVideoEl.readyState >= 2) {
      const size = 220
      const margin = 24
      const x = margin
      const y = h - size - margin

      mixCtx.save()
      mixCtx.beginPath()
      mixCtx.arc(x + size / 2, y + size / 2, size / 2, 0, Math.PI * 2)
      mixCtx.closePath()
      mixCtx.clip()
      // mirror the webcam like a selfie view
      mixCtx.translate(x + size, y)
      mixCtx.scale(-1, 1)
      mixCtx.drawImage(webcamVideoEl, 0, 0, size, size)
      mixCtx.restore()

      mixCtx.beginPath()
      mixCtx.arc(x + size / 2, y + size / 2, size / 2, 0, Math.PI * 2)
      mixCtx.lineWidth = 4
      mixCtx.strokeStyle = '#dba384'
      mixCtx.stroke()
    }

    mixRAF = requestAnimationFrame(draw)
  }
  draw()
}

function beginRecorder() {
  if (!mixCanvas) return
  recordedChunks = []
  if (recordedUrl.value) URL.revokeObjectURL(recordedUrl.value)
  recordedUrl.value = ''
  recordSeconds.value = 0

  const canvasStream = mixCanvas.captureStream(30)

  // pull in her mic audio, plus tab/system audio if she allowed sharing it,
  // so both her reaction and whatever's playing on the page get captured
  const audioTracks: MediaStreamTrack[] = []
  cameraStream.value?.getAudioTracks().forEach((t) => audioTracks.push(t))
  screenStream.value?.getAudioTracks().forEach((t) => audioTracks.push(t))

  const combined = new MediaStream([...canvasStream.getVideoTracks(), ...audioTracks])

  const mimeCandidates = ['video/webm;codecs=vp9,opus', 'video/webm;codecs=vp8,opus', 'video/webm']
  const mimeType = mimeCandidates.find((m) => MediaRecorder.isTypeSupported?.(m)) ?? ''

  try {
    mediaRecorder = mimeType
      ? new MediaRecorder(combined, { mimeType })
      : new MediaRecorder(combined)
  } catch (err) {
    console.error('MediaRecorder init failed:', err)
    cameraError.value = 'Recording is not supported in this browser.'
    stopReactionRecording()
    return
  }

  mediaRecorder.ondataavailable = (e) => {
    if (e.data && e.data.size > 0) recordedChunks.push(e.data)
  }
  mediaRecorder.onstop = () => {
    const blob = new Blob(recordedChunks, { type: mediaRecorder?.mimeType || 'video/webm' })
    recordedUrl.value = URL.createObjectURL(blob)
  }

  mediaRecorder.start()

  recordTimerId = setInterval(() => {
    recordSeconds.value++
    if (recordSeconds.value >= MAX_RECORD_SECONDS) stopReactionRecording()
  }, 1000)
}

function cleanupStreams() {
  cameraStream.value?.getTracks().forEach((t) => t.stop())
  cameraStream.value = null
  screenStream.value?.getTracks().forEach((t) => t.stop())
  screenStream.value = null

  if (mixRAF) cancelAnimationFrame(mixRAF)
  mixRAF = 0
  screenVideoEl?.pause()
  webcamVideoEl?.pause()
  screenVideoEl = null
  webcamVideoEl = null
  mixCanvas = null
  mixCtx = null
}

function stopReactionRecording() {
  if (mediaRecorder && mediaRecorder.state !== 'inactive') {
    mediaRecorder.stop()
  }
  if (recordTimerId) {
    clearInterval(recordTimerId)
    recordTimerId = null
  }
  cleanupStreams()
  cameraStage.value = 'reviewing'
}

function recordAgain() {
  if (recordedUrl.value) URL.revokeObjectURL(recordedUrl.value)
  recordedUrl.value = ''
  startReactionRecording()
}

function finishReview() {
  if (recordedUrl.value) URL.revokeObjectURL(recordedUrl.value)
  recordedUrl.value = ''
  recordSeconds.value = 0
  cameraStage.value = 'idle'
}

function downloadRecording() {
  if (!recordedUrl.value) return
  const a = document.createElement('a')
  a.href = recordedUrl.value
  a.download = `birthday-reaction-${herName}.webm`
  document.body.appendChild(a)
  a.click()
  a.remove()
}

function formatSeconds(total: number) {
  const m = Math.floor(total / 60)
  const s = total % 60
  return `${m}:${s.toString().padStart(2, '0')}`
}

/* ============================================================
   ⚙️ SETTINGS MENU — a single small gear icon that tucks the
   music toggle and the reaction-recorder away, instead of
   scattering separate floating buttons across the page.
   ============================================================ */
const settingsOpen = ref(false)

function toggleSettings() {
  settingsOpen.value = !settingsOpen.value
}
function closeSettings() {
  settingsOpen.value = false
}
function handleRecordClick() {
  closeSettings()
  startReactionRecording()
}

/* ============================================================
   ✨ Premium chrome — top scroll progress bar + a quiet side
   nav so the whole page feels like one considered experience.
   ============================================================ */
const scrollProgress = ref(0)
function updateScrollProgress() {
  const doc = document.documentElement
  const scrollTop = window.scrollY || doc.scrollTop
  const height = doc.scrollHeight - doc.clientHeight
  scrollProgress.value = height > 0 ? Math.min(100, (scrollTop / height) * 100) : 0
}

const navDots = [
  { id: 'hero', label: 'Home', icon: '💌' },
  { id: 'wishes', label: 'Wishes', icon: '🎀' },
  { id: 'jokes', label: 'Jokes', icon: '😂' },
  { id: 'gallery', label: 'Moments', icon: '📷' },
  { id: 'film', label: 'Film', icon: '🎬' },
  { id: 'wish', label: 'Wish', icon: '🕯️' },
]
const activeSection = ref('hero')
let navObserver: IntersectionObserver | undefined

function scrollToSection(id: string) {
  document.getElementById(id)?.scrollIntoView({ behavior: 'smooth', block: 'start' })
}

/* ── State ── */
const sealBroken = ref(false)
const envelopeOpen = ref(false)
const bookOpen = ref(false)
const showBook = ref(false)
const musicPlaying = ref(false)
const audio = ref<HTMLAudioElement | null>(null)

/* cinema state */
const cinemaPhase = ref<'idle' | 'opening' | 'playing'>('idle')

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
  letterProgress.value = 0
}

/* ============================================================
   🎀 Letter reading ribbon — a slim gold thread across the top
   of the book that fills in as she scrolls through the letter,
   plus a soft synthesized "page turn" sound when the cover
   opens. Purely presentational — the message text itself is
   never touched.
   ============================================================ */
const letterProgress = ref(0)

function onLetterScroll(e: Event) {
  const el = e.target as HTMLElement
  const max = el.scrollHeight - el.clientHeight
  letterProgress.value = max > 0 ? Math.min(100, (el.scrollTop / max) * 100) : 100
}

let letterAudioCtx: AudioContext | null = null
function getLetterAudioCtx(): AudioContext | null {
  try {
    if (!letterAudioCtx) {
      const Ctx = window.AudioContext || (window as any).webkitAudioContext
      if (!Ctx) return null
      letterAudioCtx = new Ctx()
    }
    if (letterAudioCtx.state === 'suspended') {
      letterAudioCtx.resume().catch(() => {})
    }
    return letterAudioCtx
  } catch (err) {
    console.error('AudioContext unavailable:', err)
    return null
  }
}

/** A soft filtered-noise "paper rustle" — the sound of a page turning. */
function playPageTurnSound() {
  const ctx = getLetterAudioCtx()
  if (!ctx) return
  const now = ctx.currentTime

  const bufferSize = Math.floor(ctx.sampleRate * 0.35)
  const buffer = ctx.createBuffer(1, bufferSize, ctx.sampleRate)
  const data = buffer.getChannelData(0)
  for (let i = 0; i < bufferSize; i++) {
    data[i] = (Math.random() * 2 - 1) * Math.pow(1 - i / bufferSize, 1.4)
  }
  const noise = ctx.createBufferSource()
  noise.buffer = buffer

  const filter = ctx.createBiquadFilter()
  filter.type = 'bandpass'
  filter.Q.value = 0.7
  filter.frequency.setValueAtTime(900, now)
  filter.frequency.linearRampToValueAtTime(2600, now + 0.18)
  filter.frequency.linearRampToValueAtTime(1400, now + 0.34)

  const gain = ctx.createGain()
  gain.gain.setValueAtTime(0.0001, now)
  gain.gain.linearRampToValueAtTime(0.16, now + 0.04)
  gain.gain.exponentialRampToValueAtTime(0.0001, now + 0.34)

  noise.connect(filter).connect(gain).connect(ctx.destination)
  noise.start(now)
  noise.stop(now + 0.36)
}

function openCover() {
  if (bookOpen.value) return
  bookOpen.value = true
  playPageTurnSound()
}

/* ── Music: autoplay on load, loop forever, auto-duck for the video ── */

async function playMusic() {
  if (!audio.value) return
  try {
    await audio.value.play()
    musicPlaying.value = true
  } catch (err) {
    console.error('Music playback failed:', err)
    musicPlaying.value = false
  }
}

function pauseMusic() {
  if (!audio.value) return
  audio.value.pause()
  musicPlaying.value = false
}

async function toggleMusic() {
  if (musicPlaying.value) {
    pauseMusic()
  } else {
    await playMusic()
  }
}

// One-time fallback: if the browser blocks autoplay-with-sound, start
// music on the very first user interaction anywhere on the page.
function unlockAutoplayOnFirstInteraction() {
  if (musicPlaying.value) return
  playMusic()
}

/* ── Cinema: curtain open → spotlight → video ── */
function startCinema() {
  if (cinemaPhase.value !== 'idle') return
  cinemaPhase.value = 'opening'
  // music ducks out as soon as the curtain opens
  pauseMusic()
  // after curtain + spotlight animation finishes, show the video
  setTimeout(() => {
    cinemaPhase.value = 'playing'
  }, 2600)
}

// Video element events: music stops the instant the video plays,
// and automatically resumes the instant the video is paused/ends.
function onVideoPlay() {
  pauseMusic()
}
function onVideoPauseOrEnd() {
  playMusic()
}

const flipped = reactive(new Set<number>())
function toggleWish(i: number) {
  if (flipped.has(i)) flipped.delete(i)
  else flipped.add(i)
}

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

/* ============================================================
   🎂 CAKE — a two-tier "drip cake" built from primitives so it
   stays crisp at any size: gradient-shaded tiers, a hand-drawn
   melting-frosting edge on each tier (computed, not hand-typed,
   so the wave stays smooth), a gold ribbon + bow, sugar-rosette
   accents at the seam, and iced-on name + age lettering on the
   top tier. The candle + blow-to-wish interaction is unchanged.
   ============================================================ */
function dripPath(x0: number, x1: number, yTop: number, drops: number[], capHeight = 10) {
  const width = x1 - x0
  const n = drops.length
  const step = width / n
  let d = `M${x0},${yTop}`
  for (let i = 0; i < n; i++) {
    const segStart = x0 + step * i
    const segMid = segStart + step * 0.5
    const segEnd = segStart + step
    const dropY = yTop + (drops[i] ?? 10)
    d += ` C${segStart + step * 0.22},${yTop} ${segStart + step * 0.22},${dropY} ${segMid},${dropY}`
    d += ` C${segMid + step * 0.28},${dropY} ${segMid + step * 0.28},${yTop} ${segEnd},${yTop}`
  }
  d += ` L${x1},${yTop - capHeight} L${x0},${yTop - capHeight} Z`
  return d
}

const bottomDrips = [9, 17, 7, 21, 11, 19, 8, 15, 10, 18, 7]
const topDrips = [13, 21, 9, 23, 12, 19, 10]

const bottomTierDripPath = computed(() => dripPath(44, 256, 176, bottomDrips, 11))
const topTierDripPath = computed(() => dripPath(84, 216, 96, topDrips, 11))

function pearlRow(x0: number, x1: number, gap: number) {
  const arr: number[] = []
  for (let x = x0; x <= x1; x += gap) arr.push(x)
  return arr
}
const bottomPearls = computed(() => pearlRow(52, 248, 13))
const topPearls = computed(() => pearlRow(90, 210, 12))

const miniRoses = [
  { x: 96, y: 179, scale: 0.46, color: 'var(--wine-soft)', center: 'var(--gold-light)' },
  { x: 150, y: 184, scale: 0.5, color: 'var(--blush)', center: 'var(--gold)' },
  { x: 204, y: 179, scale: 0.46, color: 'var(--pink-mid)', center: 'var(--gold-light)' },
]

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
  if (window.matchMedia('(pointer: fine)').matches) {
    window.addEventListener('mousemove', handleMouseMove)
  }

  // Try to autoplay the music right away.
  playMusic()

  // Fallback for browsers that block autoplay with sound: start music
  // on the first click/tap/keypress anywhere on the page.
  window.addEventListener('click', unlockAutoplayOnFirstInteraction, { once: true })
  window.addEventListener('keydown', unlockAutoplayOnFirstInteraction, { once: true })
  window.addEventListener('touchstart', unlockAutoplayOnFirstInteraction, { once: true })

  // premium chrome: scroll progress + section nav highlighting
  window.addEventListener('scroll', updateScrollProgress, { passive: true })
  updateScrollProgress()

  // gallery lightbox keyboard navigation
  window.addEventListener('keydown', onLightboxKeydown)

  navObserver = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          activeSection.value = entry.target.id
        }
      })
    },
    { rootMargin: '-40% 0px -55% 0px', threshold: 0 },
  )
  navDots.forEach((s) => {
    const el = document.getElementById(s.id)
    if (el) navObserver?.observe(el)
  })
})
onUnmounted(() => {
  window.removeEventListener('mousemove', handleMouseMove)
  window.removeEventListener('click', unlockAutoplayOnFirstInteraction)
  window.removeEventListener('keydown', unlockAutoplayOnFirstInteraction)
  window.removeEventListener('touchstart', unlockAutoplayOnFirstInteraction)
  window.removeEventListener('scroll', updateScrollProgress)
  window.removeEventListener('keydown', onLightboxKeydown)
  if (navObserver) navObserver.disconnect()
  if (recordTimerId) clearInterval(recordTimerId)
  if (mediaRecorder && mediaRecorder.state !== 'inactive') mediaRecorder.stop()
  cleanupStreams()
  if (recordedUrl.value) URL.revokeObjectURL(recordedUrl.value)
  if (letterAudioCtx) {
    letterAudioCtx.close().catch(() => {})
  }
})

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
  <!-- premium chrome: scroll progress + quiet side nav -->
  <div class="scroll-progress" :style="{ width: scrollProgress + '%' }"></div>
  <nav class="side-nav" aria-hidden="true">
    <button
      v-for="s in navDots"
      :key="s.id"
      class="side-dot"
      :class="{ 'is-active': activeSection === s.id }"
      @click="scrollToSection(s.id)"
      :aria-label="s.label"
    >
      <span class="side-dot-core">{{ s.icon }}</span>
      <span class="side-dot-tooltip">{{ s.label }}</span>
    </button>
  </nav>

  <div class="grain-overlay"></div>

  <!-- soft night-sky ambiance, matching the gate page: twinkling stars
       + drifting bokeh light, sitting behind everything else -->
  <div class="stars-layer" aria-hidden="true">
    <span
      v-for="n in 44"
      :key="'star' + n"
      class="star"
      :style="{
        left: ((n * 13.7 + 3) % 100) + '%',
        top: ((n * 7.9 + 4) % 100) + '%',
        animationDelay: (n % 22) * 0.28 + 's',
        animationDuration: 2.2 + (n % 4) * 0.7 + 's',
        width: 1 + (n % 3) + 'px',
        height: 1 + (n % 3) + 'px',
      }"
    ></span>
  </div>
  <div class="bokeh-layer" aria-hidden="true">
    <span
      v-for="n in 10"
      :key="'bokeh' + n"
      class="bokeh"
      :style="{
        left: ((n * 17 + 4) % 92) + '%',
        top: ((n * 23 + 8) % 96) + '%',
        animationDelay: n * 1.3 + 's',
        animationDuration: 7 + (n % 3) + 's',
        width: 60 + (n % 3) * 30 + 'px',
        height: 60 + (n % 3) * 30 + 'px',
      }"
    ></span>
  </div>

  <div class="sparkle-layer">
    <span
      class="sparkle"
      v-for="s in sparkles"
      :key="s.id"
      :style="{ left: s.x + 'px', top: s.y + 'px' }"
      >💋</span
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

  <!-- floating hearts burst wherever a "like" is tapped -->
  <div class="love-burst-layer" aria-hidden="true">
    <span
      class="love-burst"
      v-for="h in loveBursts"
      :key="h.id"
      :style="{ left: h.x + 'px', top: h.y + 'px' }"
      >{{ h.glyph }}</span
    >
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

  <!-- ── HERO ── -->
  <section class="hero" id="hero">
    <p class="eyebrow label">a little something for</p>
    <h1 class="hero-title">Happy Birthday My Love</h1>
    <p class="hero-sub script" style="font-size: 1.6rem">{{ herName }}</p>

    <button class="hero-love-btn" @click="tapLove" aria-label="Tap for love">
      tap for love <span>♡</span>
    </button>
    <p class="hero-love-msg script" v-if="loveTapMessage">{{ loveTapMessage }}</p>

    <div class="envelope-wrap">
      <div class="envelope" :class="{ 'is-open': envelopeOpen, 'is-lifted': envelopeOpen }">
        <div class="postage-stamp"><span>💋</span></div>
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
    <p class="hint" v-else>tap the card above 💋</p>
  </section>

  <!-- ── LETTER ── -->
  <div class="scene-backdrop" v-if="showBook" @click.self="closeBook">
    <div class="book-stage">
      <div class="book" :class="{ 'is-open': bookOpen }">
        <div class="letter-progress-bar" v-if="bookOpen" aria-hidden="true">
          <div class="letter-progress-fill" :style="{ width: letterProgress + '%' }"></div>
        </div>
        <button class="close-scene" @click="closeBook" aria-label="Close">×</button>
        <div class="book-inside" v-if="bookOpen" @scroll="onLetterScroll">
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
        <div class="book-cover" @click="openCover" v-show="!bookOpen">
          <div class="monogram script">{{ herName.charAt(0) }}</div>
          <div class="cover-title script">For {{ herName }}</div>
          <div class="open-cta label">tap to open</div>
        </div>
      </div>

      <div class="bouquet-reveal" v-if="bouquetRevealed">
        <div class="bouquet-glow"></div>
        <svg class="bouquet-svg" viewBox="0 0 300 400" xmlns="http://www.w3.org/2000/svg">
          <path class="wrap-paper" d="M150,296 L252,392 L48,392 Z" />
          <path class="wrap-paper wrap-paper-fold" d="M150,296 L206,392 L94,392 Z" />
          <path v-for="(r, i) in roses" :key="'stem' + i" class="stem" :d="r.stem" />
          <path v-for="(t, i) in tulips" :key="'tstem' + i" class="stem" :d="t.stem" />
          <path v-for="(d, i) in daisies" :key="'dstem' + i" class="stem" :d="d.stem" />
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
          <!-- daisies: soft filler blooms tucked around the base -->
          <g
            v-for="(d, i) in daisies"
            :key="'daisy' + i"
            :transform="
              'translate(' + d.x + ',' + d.y + ') rotate(' + d.rotate + ') scale(' + d.scale + ')'
            "
          >
            <g v-for="n in 10" :key="'dp' + n" :transform="'rotate(' + n * 36 + ')'">
              <path class="daisy-petal" :d="daisyPetalPath" />
            </g>
            <circle class="daisy-center" r="5" />
          </g>
          <!-- tulips: a second flower woven in with the roses -->
          <g
            v-for="(t, i) in tulips"
            :key="'tulip' + i"
            :transform="
              'translate(' + t.x + ',' + t.y + ') rotate(' + t.rotate + ') scale(' + t.scale + ')'
            "
          >
            <path
              class="tulip-petal tulip-side"
              :d="tulipPetalPath"
              :fill="t.side"
              transform="rotate(-24) translate(-3,0)"
            />
            <path
              class="tulip-petal tulip-side"
              :d="tulipPetalPath"
              :fill="t.side"
              transform="rotate(24) translate(3,0)"
            />
            <path class="tulip-petal tulip-front" :d="tulipPetalPath" :fill="t.front" />
          </g>
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
          <g
            v-for="(f, i) in fillers"
            :key="'filler' + i"
            :transform="
              'translate(' + f.x + ',' + f.y + ') rotate(' + f.rotate + ') scale(' + f.scale + ')'
            "
          >
            <ellipse
              v-for="n in 6"
              :key="'fp' + n"
              class="filler-petal"
              :transform="'rotate(' + n * 60 + ')'"
              cx="0"
              cy="-7"
              rx="2.4"
              ry="6"
            />
            <circle class="filler-center" r="2.6" />
          </g>
          <!-- a few twinkling sparkles drifting over the whole arrangement -->
          <g class="bouquet-sparkles" aria-hidden="true">
            <circle
              v-for="(sp, i) in bouquetSparkles"
              :key="'sp' + i"
              :cx="sp.x"
              :cy="sp.y"
              :r="sp.r"
              class="bouquet-sparkle"
              :style="{ animationDelay: sp.delay + 's' }"
            />
          </g>
          <!-- a little hanging note tag tied to the ribbon — tap it for a burst of love -->
          <g
            class="bouquet-tag"
            transform="translate(150,330)"
            @click="burstHeartsAt"
            role="button"
            aria-label="A little note"
          >
            <line x1="0" y1="-33" x2="0" y2="-8" class="tag-thread" />
            <rect x="-48" y="-8" width="98" height="30" rx="6" class="tag-body" />
            <text x="0" y="11" class="tag-text" text-anchor="middle">with all my love</text>
          </g>
        </svg>
      </div>
    </div>
  </div>

  <!-- ── WISHES ── -->
  <section id="wishes" v-reveal>
    <div class="section-head">
      <h2>My Wishes for You</h2>
      <p>tap a card to reveal a wish</p>
    </div>
    <div class="wishes-grid">
      <div
        class="wish-card"
        v-for="(wish, i) in wishes"
        :key="i"
        :class="{ 'is-flipped': flipped.has(i) }"
        @click="toggleWish(i)"
        :style="{ transitionDelay: i * 0.06 + 's' }"
        v-reveal
      >
        <div class="wish-card-inner">
          <div class="wish-card-face wish-card-front">
            <span class="wish-index label">Wish {{ i + 1 }}</span>
            <span class="wish-glyph">♡</span>
          </div>
          <div class="wish-card-face wish-card-back">
            <p>{{ wish }}</p>
          </div>
        </div>
      </div>
    </div>
  </section>

  <!-- ── TITO JOKES ── -->
  <section id="jokes" v-reveal>
    <div class="section-head">
      <span class="corny-badge label">certified corny™</span>
      <h2>Tito Jokes to make you smile</h2>
      <p>tap the card to reveal the punchline — I make no apologies 😅</p>
    </div>

    <div class="joke-wrap">
      <div class="reward-tray">
        <div
          class="reward-slot"
          v-for="(c, i) in titoJokes"
          :key="'slot' + i"
          :class="{ 'is-filled': collectedIcons[i] }"
        >
          <span v-if="collectedIcons[i]">{{ collectedIcons[i] }}</span>
          <span v-else class="reward-slot-empty">·</span>
        </div>
      </div>

      <div class="joke-card" v-if="!jokesComplete">
        <div class="joke-card-crest">
          <span class="joke-card-crest-line"></span>
          <span class="joke-card-crest-icon">🎙️</span>
          <span class="joke-card-crest-line"></span>
        </div>
        <span class="quiz-progress label">Joke {{ jokeIndex + 1 }} of {{ titoJokes.length }}</span>
        <h3 class="quiz-question">{{ currentJoke.teaser }}</h3>

        <button class="quiz-option joke-reveal-btn" v-if="!jokeRevealed" @click="revealJoke">
          Reveal punchline 🥁
        </button>

        <p class="quiz-feedback is-good joke-line" v-else>{{ currentJoke.line }}</p>

        <button class="quiz-replay-btn" v-if="jokeRevealed" @click="nextJoke">
          {{ jokeIndex < titoJokes.length - 1 ? 'Next joke →' : 'Finish' }}
        </button>
      </div>

      <div class="joke-card quiz-complete" v-else>
        <div class="quiz-complete-bouquet">
          <span
            v-for="(icon, i) in collectedIcons"
            :key="'j' + i"
            :style="{ animationDelay: i * 0.12 + 's' }"
            >{{ icon }}</span
          >
        </div>
        <span class="corny-badge label">tito-approved</span>
        <h3 class="quiz-question">Okay, I'll retire from stand-up now 😂</h3>
        <p class="quiz-complete-note">
          Corny jokes aside — you smiling, even at these, is still my favorite thing in the world.
        </p>
        <button class="quiz-replay-btn" @click="resetJokes">Hear them again</button>
      </div>
    </div>
  </section>

  <!-- ── GALLERY ── -->
  <section id="gallery" v-reveal>
    <div class="section-head">
      <h2>Our Moments</h2>
      <p>tap a photo to look closer · tap ♡ to love it · swipe or use arrows in the viewer</p>
      <p class="gallery-love-counter label" v-if="heartCount">
        {{ heartCount }} of {{ photos.length }} moments loved 💛
      </p>
    </div>
    <div class="gallery-grid">
      <figure
        class="polaroid"
        v-for="(photo, i) in photos"
        :key="i"
        :class="{ 'is-liked': photo.liked }"
        :style="{
          transform: polaroidTransform(i),
          transitionDelay: i * 0.06 + 's',
        }"
        @mousemove="handleTilt($event, i)"
        @mouseleave="resetTilt(i)"
        @click="openLightbox(i)"
        v-reveal
      >
        <button
          class="polaroid-like"
          :class="{ 'is-liked': photo.liked }"
          @click.stop="toggleLike(photo, $event)"
          aria-label="Love this photo"
        >
          {{ photo.liked ? '♥' : '♡' }}
        </button>
        <div class="frame">
          <img v-if="photo.src" :src="photo.src" :alt="photo.caption" loading="lazy" />
          <svg v-else viewBox="0 0 24 24">
            <path
              d="M9 3 7.17 5H4a2 2 0 0 0-2 2v11a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-3.17L15 3H9zm3 15a5 5 0 1 1 0-10 5 5 0 0 1 0 10z"
            />
          </svg>
          <div class="frame-overlay"><span class="frame-overlay-icon">🔍</span></div>
        </div>
        <figcaption>
          <span class="polaroid-index">{{ i + 1 }}</span>
          <span v-if="photo.caption">{{ photo.caption }}</span>
        </figcaption>
      </figure>
    </div>
  </section>

  <!-- ══════════════════════════════════════════
       CINEMA SECTION
  ══════════════════════════════════════════ -->
  <section id="film" v-reveal>
    <div class="section-head">
      <h2>A Little Film for You</h2>
      <p>{{ mainVideo.caption }}</p>
    </div>

    <div class="cinema-room" :class="'cinema--' + cinemaPhase">
      <!-- gold arch trim top -->
      <div class="cinema-arch-top"></div>

      <!-- wall sconces -->
      <div class="sconces">
        <div class="sconce" v-for="n in 6" :key="n">
          <div class="sconce-bracket"></div>
          <div class="sconce-bulb" :style="{ animationDelay: n * 0.35 + 's' }"></div>
        </div>
      </div>

      <!-- screen + curtains -->
      <div class="cinema-stage">
        <!-- gold curtain rod with finials -->
        <div class="curtain-rod">
          <span class="rod-finial rod-finial-l"></span>
          <span class="rod-finial rod-finial-r"></span>
        </div>

        <div class="stage-inner">
          <!-- left curtain -->
          <div class="curtain curtain-l" :class="{ 'curtain-open': cinemaPhase !== 'idle' }">
            <div class="curtain-fold"></div>
            <div class="curtain-fold"></div>
          </div>
          <!-- right curtain -->
          <div class="curtain curtain-r" :class="{ 'curtain-open': cinemaPhase !== 'idle' }">
            <div class="curtain-fold"></div>
            <div class="curtain-fold"></div>
          </div>

          <!-- spotlight cone (shows during opening phase) -->
          <div class="spotlight" v-if="cinemaPhase === 'opening'"></div>

          <!-- the screen bezel -->
          <div class="cinema-bezel">
            <div class="scanlines"></div>
            <div class="screen-glare"></div>
            <div class="screen-vignette"></div>

            <!-- IDLE: show play prompt -->
            <div v-if="cinemaPhase === 'idle'" class="film-thumb" @click="startCinema">
              <img v-if="mainVideo.poster" :src="mainVideo.poster" class="film-poster" alt="" />
              <div class="cinema-play-wrap">
                <div class="pulse-ring ring-1"></div>
                <div class="pulse-ring ring-2"></div>
                <div class="pulse-ring ring-3"></div>
                <div class="play-btn big">
                  <svg viewBox="0 0 24 24"><path d="M8 5v14l11-7z" /></svg>
                </div>
              </div>
              <div v-if="!mainVideo.src" class="video-placeholder">
                Add your video URL in the config to play it here.
              </div>
            </div>

            <!-- OPENING: curtain animation screen (dark) -->
            <div v-else-if="cinemaPhase === 'opening'" class="film-thumb film-thumb--dark">
              <div class="opening-text script">now showing…</div>
            </div>

            <!-- PLAYING: real video -->
            <video
              v-else-if="cinemaPhase === 'playing' && mainVideo.src"
              :src="mainVideo.src"
              class="film-video"
              controls
              autoplay
              @play="onVideoPlay"
              @pause="onVideoPauseOrEnd"
              @ended="onVideoPauseOrEnd"
            ></video>
          </div>
        </div>
      </div>

      <!-- floor fade -->
      <div class="cinema-floor-fade"></div>

      <!-- audience seat silhouettes -->
      <div class="cinema-seats">
        <div class="seat-row">
          <div class="seat" v-for="n in 9" :key="n"></div>
        </div>
      </div>

      <!-- gold arch trim bottom -->
      <div class="cinema-arch-bottom"></div>
    </div>
  </section>

  <!-- ── WISH / CANDLE ── -->
  <section id="wish" v-reveal>
    <div class="section-head">
      <h2>Make a Wish</h2>
      <p v-if="candleLit">tap the candle to blow it out</p>
      <p v-else>&nbsp;</p>
    </div>
    <div class="cake-wrap">
      <svg
        class="cake"
        viewBox="0 0 300 300"
        @click="blowCandle"
        role="button"
        aria-label="Blow out the candle"
      >
        <defs>
          <linearGradient id="tierBottomGrad" x1="0" y1="0" x2="0" y2="1">
            <stop offset="0%" stop-color="var(--wine-soft)" />
            <stop offset="55%" stop-color="var(--wine)" />
            <stop offset="100%" stop-color="var(--wine-deep)" />
          </linearGradient>
          <linearGradient id="tierTopGrad" x1="0" y1="0" x2="0" y2="1">
            <stop offset="0%" stop-color="var(--wine)" />
            <stop offset="60%" stop-color="var(--wine-deep)" />
            <stop offset="100%" stop-color="#3a0c26" />
          </linearGradient>
          <linearGradient id="frostingGrad" x1="0" y1="0" x2="0" y2="1">
            <stop offset="0%" stop-color="#fffaf6" />
            <stop offset="100%" stop-color="var(--blush)" />
          </linearGradient>
          <radialGradient id="plateGrad" cx="40%" cy="35%" r="70%">
            <stop offset="0%" stop-color="var(--gold-light)" />
            <stop offset="100%" stop-color="var(--gold)" />
          </radialGradient>
          <linearGradient id="ribbonGrad" x1="0" y1="0" x2="1" y2="0">
            <stop offset="0%" stop-color="var(--gold)" />
            <stop offset="50%" stop-color="var(--gold-light)" />
            <stop offset="100%" stop-color="var(--gold)" />
          </linearGradient>
          <linearGradient id="candleGrad" x1="0" y1="0" x2="1" y2="0">
            <stop offset="0%" stop-color="var(--gold)" />
            <stop offset="50%" stop-color="#fff8f2" />
            <stop offset="100%" stop-color="var(--gold)" />
          </linearGradient>
        </defs>

        <!-- plate -->
        <ellipse cx="150" cy="270" rx="128" ry="13" class="cake-plate-shadow" />
        <ellipse cx="150" cy="261" rx="120" ry="11" fill="url(#plateGrad)" />
        <ellipse cx="150" cy="258" rx="108" ry="8" class="cake-plate-top" />

        <!-- bottom tier -->
        <rect x="44" y="178" width="212" height="66" rx="7" fill="url(#tierBottomGrad)" />
        <!-- drip frosting cap -->
        <path :d="bottomTierDripPath" fill="url(#frostingGrad)" class="frosting-drip" />
        <!-- gold ribbon band -->
        <rect x="44" y="209" width="212" height="11" fill="url(#ribbonGrad)" class="ribbon-strip" />
        <g class="cake-bow" transform="translate(150,214)">
          <path class="ribbon-loop" d="M0,0 C-19,-6 -22,-17 -10,-19 C-3,-20 1,-13 0,0 Z" />
          <path class="ribbon-loop" d="M0,0 C19,-6 22,-17 10,-19 C3,-20 -1,-13 0,0 Z" />
          <rect
            class="ribbon-knot"
            x="-5.5"
            y="-5.5"
            width="11"
            height="11"
            rx="2"
            transform="rotate(45)"
          />
        </g>
        <!-- pearl border along the base -->
        <circle
          v-for="(x, i) in bottomPearls"
          :key="'bp' + i"
          :cx="x"
          cy="243"
          r="2.6"
          class="cake-pearl"
        />

        <!-- sugar rosettes sitting in the seam between tiers -->
        <g
          v-for="(mr, i) in miniRoses"
          :key="'mr' + i"
          :transform="'translate(' + mr.x + ',' + mr.y + ') scale(' + mr.scale + ')'"
        >
          <g v-for="n in 6" :key="'mrp' + n" :transform="'rotate(' + n * 60 + ')'">
            <ellipse cx="0" cy="-9" rx="6" ry="9" :fill="mr.color" class="mini-rose-petal" />
          </g>
          <circle r="4" :fill="mr.center" />
        </g>
        <g transform="translate(122,171) scale(0.5)">
          <path class="leaf" :d="leafPath" transform="rotate(-40)" />
        </g>
        <g transform="translate(178,171) scale(0.5)">
          <path class="leaf" :d="leafPath" transform="rotate(220)" />
        </g>

        <!-- top tier -->
        <rect x="84" y="98" width="132" height="72" rx="7" fill="url(#tierTopGrad)" />
        <path :d="topTierDripPath" fill="url(#frostingGrad)" class="frosting-drip" />
        <circle
          v-for="(x, i) in topPearls"
          :key="'tp' + i"
          :cx="x"
          cy="164"
          r="2.2"
          class="cake-pearl"
        />

        <!-- iced-on name + age -->
        <text x="150" y="129" text-anchor="middle" class="cake-name script">{{ herName }}</text>
        <line x1="118" y1="139" x2="182" y2="139" class="cake-name-underline" />
        <text x="150" y="157" text-anchor="middle" class="cake-age">{{ herAge }}</text>

        <!-- candle -->
        <g transform="translate(150,0)">
          <rect
            x="-6"
            y="48"
            width="12"
            height="40"
            rx="3"
            fill="url(#candleGrad)"
            class="candle-body"
          />
          <rect x="-6" y="58" width="12" height="4" class="candle-stripe" />
          <rect x="-6" y="70" width="12" height="4" class="candle-stripe" />
          <path d="M-6,48 Q0,44 6,48 L6,52 Q0,49 -6,52 Z" class="candle-drip" />
          <g v-if="candleLit" class="flame-group" transform="translate(0,48)">
            <path class="flame-glow" d="M0,-2 C-11,-16 -16,-28 0,-40 C16,-28 11,-16 0,-2 Z" />
            <path
              class="flame"
              d="M0,-2 C-7,-12 -11,-20 0,-30 C11,-20 7,-12 0,-2 Z"
              fill="#ffb84d"
            />
            <path
              class="flame-core"
              d="M0,-2 C-3,-8 -4,-13 0,-18 C4,-13 3,-8 0,-2 Z"
              fill="#fff3c4"
            />
          </g>
          <g v-else class="smoke" transform="translate(0,48)">
            <path
              d="M0,0 Q6,-10 0,-20 Q-6,-30 0,-40"
              stroke="rgba(255,243,247,0.5)"
              stroke-width="2.5"
              fill="none"
              stroke-linecap="round"
            />
          </g>
        </g>
      </svg>
      <p class="wish-hint" v-if="candleLit">🕯️ tap</p>
      <p class="wish-text script" v-else-if="wishGranted">{{ wishText }}</p>
    </div>
  </section>

  <footer v-reveal>
    <span class="script">Happy Birthday, My Love {{ herName }}. I love you so much.</span>
    <p>— {{ yourName }}</p>
    <p class="footer-love-tally label" v-if="totalLoveCollected">
      you've collected {{ totalLoveCollected }} little pieces of love here 💛
    </p>
  </footer>

  <!-- gallery lightbox: prev/next nav, keyboard arrows, swipe, double-tap-like -->
  <div
    class="modal gallery-lightbox"
    v-if="lightboxPhoto"
    @click.self="closeLightbox"
    @touchstart="onLightboxTouchStart"
    @touchend="onLightboxTouchEnd"
  >
    <button class="modal-close" @click="closeLightbox" aria-label="Close">×</button>
    <button class="lightbox-nav lightbox-prev" @click.stop="prevPhoto" aria-label="Previous photo">
      ‹
    </button>
    <button class="lightbox-nav lightbox-next" @click.stop="nextPhoto" aria-label="Next photo">
      ›
    </button>
    <figure @dblclick="toggleLike(lightboxPhoto, $event)">
      <img
        :key="lightboxIndex"
        v-if="lightboxPhoto.src"
        :src="lightboxPhoto.src"
        :alt="lightboxPhoto.caption"
        class="lightbox-fade"
      />
      <figcaption>
        <span v-if="lightboxPhoto.caption">{{ lightboxPhoto.caption }}</span>
        <span class="lightbox-counter label">{{ lightboxIndex + 1 }} / {{ photos.length }}</span>
      </figcaption>
    </figure>
    <button
      class="lightbox-like"
      :class="{ 'is-liked': lightboxPhoto.liked }"
      @click.stop="toggleLike(lightboxPhoto, $event)"
    >
      {{ lightboxPhoto.liked ? '♥ loved' : '♡ love this' }}
    </button>
  </div>

  <audio ref="audio" :src="musicUrl" loop></audio>

  <!-- ══════════════════════════════════════════
       SETTINGS — one small gear icon holds music +
       the reaction recorder, instead of two separate
       floating buttons cluttering the corner.
  ══════════════════════════════════════════ -->
  <div class="settings-overlay" v-if="settingsOpen" @click="closeSettings"></div>
  <div class="settings-wrap">
    <transition name="settings-pop">
      <div class="settings-panel" v-if="settingsOpen">
        <div class="settings-row">
          <span class="settings-row-icon">🎵</span>
          <span class="settings-row-label">Music</span>
          <button
            class="settings-switch"
            :class="{ 'is-on': musicPlaying }"
            @click="toggleMusic"
            :aria-pressed="musicPlaying"
            aria-label="Toggle music"
          >
            <span class="settings-switch-knob"></span>
          </button>
        </div>

        <div class="settings-divider" v-if="cameraSupported"></div>

        <div class="settings-row" v-if="cameraSupported">
          <span class="settings-row-icon">🎥</span>
          <span class="settings-row-label">Record reaction</span>
          <button
            class="settings-action-btn"
            @click="handleRecordClick"
            :disabled="cameraStage !== 'idle'"
          >
            {{ cameraStage === 'idle' ? 'Start' : '…' }}
          </button>
        </div>

        <p class="settings-error" v-if="cameraError">{{ cameraError }}</p>
      </div>
    </transition>

    <button
      class="settings-toggle"
      :class="{ 'is-open': settingsOpen }"
      @click="toggleSettings"
      aria-label="Settings"
    >
      <svg viewBox="0 0 24 24" class="settings-gear">
        <path
          d="M19.14 12.94c.04-.3.06-.61.06-.94 0-.32-.02-.64-.07-.94l2.03-1.58a.5.5 0 0 0 .12-.65l-1.92-3.32a.5.5 0 0 0-.61-.22l-2.39.96a7.3 7.3 0 0 0-1.62-.94l-.36-2.54a.5.5 0 0 0-.5-.42h-3.84a.5.5 0 0 0-.5.42l-.36 2.54c-.59.24-1.13.56-1.62.94l-2.39-.96a.5.5 0 0 0-.61.22L2.62 8.83a.5.5 0 0 0 .12.65l2.03 1.58c-.05.3-.07.62-.07.94 0 .32.02.64.07.94l-2.03 1.58a.5.5 0 0 0-.12.65l1.92 3.32c.14.24.42.32.61.22l2.39-.96c.49.38 1.03.7 1.62.94l.36 2.54a.5.5 0 0 0 .5.42h3.84a.5.5 0 0 0 .5-.42l.36-2.54c.59-.24 1.13-.56 1.62-.94l2.39.96c.24.1.47 0 .61-.22l1.92-3.32a.5.5 0 0 0-.12-.65l-2.03-1.58ZM12 15.5a3.5 3.5 0 1 1 0-7 3.5 3.5 0 0 1 0 7Z"
        />
      </svg>
    </button>
  </div>

  <!-- reaction camera: live floating bubble while she browses -->
  <div class="camera-bubble" v-if="cameraStage === 'recording'">
    <video ref="bubblePreview" class="camera-bubble-video" muted playsinline></video>
    <div class="camera-bubble-rec">
      <span class="rec-dot"></span> {{ formatSeconds(recordSeconds) }}
    </div>
    <button class="camera-bubble-stop" @click="stopReactionRecording" aria-label="Stop recording">
      <span class="stop-square"></span>
    </button>
  </div>

  <!-- reaction camera: review after stopping -->
  <div class="camera-panel-backdrop" v-if="cameraStage === 'reviewing'" @click.self="finishReview">
    <div class="camera-panel">
      <button class="camera-panel-close" @click="finishReview" aria-label="Close">×</button>
      <h3 class="camera-panel-title script">That's a wrap 🎥</h3>
      <p class="camera-panel-sub">
        Here's the page and your reaction together — totally yours, nothing was uploaded anywhere.
      </p>

      <div class="camera-stage">
        <video
          class="camera-video camera-video-playback"
          :src="recordedUrl"
          controls
          playsinline
        ></video>
      </div>

      <div class="camera-controls">
        <button class="quiz-replay-btn" @click="downloadRecording">⬇ Save video</button>
        <button class="quiz-replay-btn rec-secondary-btn" @click="recordAgain">Record again</button>
        <button class="quiz-replay-btn rec-secondary-btn" @click="finishReview">Done</button>
      </div>
    </div>
  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Great+Vibes&display=swap');

:root {
  --wine: #8c1c4d;
  --wine-deep: #4a0f30;
  --wine-soft: #b23368;
  --blush: #ffd6e4;
  --paper: #fff3f6;
  --paper-shadow: #f3d9e2;
  --gold: #dba384;
  --gold-light: #f6d9c4;
  --ink: #5c1638;
  --pink-mid: #d46a8c;
  --leaf: #8fa377;
  --leaf-dark: #5f7350;
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

/* ── premium chrome: scroll progress + side nav ── */
.scroll-progress {
  position: fixed;
  top: 0;
  left: 0;
  height: 3px;
  background: linear-gradient(90deg, var(--wine-soft), var(--gold-light), var(--wine-soft));
  z-index: 50;
  transition: width 0.15s linear;
  box-shadow: 0 0 12px rgba(219, 163, 132, 0.6);
}
.side-nav {
  position: fixed;
  right: 18px;
  top: 50%;
  transform: translateY(-50%);
  z-index: 40;
  display: flex;
  flex-direction: column;
  gap: 14px;
}
.side-dot {
  position: relative;
  width: 30px;
  height: 30px;
  border-radius: 50%;
  border: 1px solid rgba(246, 217, 196, 0.35);
  background: rgba(74, 15, 48, 0.35);
  backdrop-filter: blur(4px);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.8rem;
  opacity: 0.55;
  transition: all 0.25s ease;
}
.side-dot:hover,
.side-dot.is-active {
  opacity: 1;
  transform: scale(1.12);
  border-color: var(--gold-light);
  box-shadow: 0 0 14px rgba(219, 163, 132, 0.5);
}
.side-dot-tooltip {
  position: absolute;
  right: 40px;
  top: 50%;
  transform: translateY(-50%);
  background: rgba(30, 6, 20, 0.9);
  color: var(--gold-light);
  font-family: 'Jost', sans-serif;
  font-size: 0.68rem;
  letter-spacing: 0.06em;
  padding: 0.3rem 0.6rem;
  border-radius: 6px;
  white-space: nowrap;
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.2s ease;
}
.side-dot:hover .side-dot-tooltip {
  opacity: 1;
}
@media (max-width: 900px) {
  .side-nav {
    display: none;
  }
}

/* ambient layers */
.grain-overlay {
  position: fixed;
  inset: 0;
  z-index: 2;
  pointer-events: none;
  opacity: 0.035;
  mix-blend-mode: overlay;
  background-image: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='120' height='120'><filter id='n'><feTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='2' stitchTiles='stitch'/></filter><rect width='100%25' height='100%25' filter='url(%23n)'/></svg>");
}

/* soft night-sky ambiance, matching the gate page: twinkling stars
   + drifting bokeh light, fixed behind the content while she scrolls */
.stars-layer {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 0;
  overflow: hidden;
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
  position: fixed;
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

/* floating hearts burst on "like" taps */
.love-burst-layer {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 45;
  overflow: hidden;
}
.love-burst {
  position: fixed;
  transform: translate(-50%, -50%);
  font-size: 1.1rem;
  animation: loveBurstFloat 0.9s ease-out forwards;
}
@keyframes loveBurstFloat {
  0% {
    opacity: 1;
    transform: translate(-50%, -50%) scale(0.6);
  }
  30% {
    opacity: 1;
    transform: translate(-50%, -140%) scale(1.15);
  }
  100% {
    opacity: 0;
    transform: translate(-50%, -220%) scale(0.9);
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
    transform: translateY(0) translateX(0) rotate(0);
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

/* scroll reveal */
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

/* hero */
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
  margin-bottom: 1.2rem;
  color: var(--blush);
}
.hero-love-btn {
  background: rgba(255, 255, 255, 0.06);
  border: 1px solid rgba(246, 217, 196, 0.35);
  color: var(--gold-light);
  border-radius: 30px;
  padding: 0.5rem 1.2rem;
  font-family: 'Jost', sans-serif;
  font-size: 0.78rem;
  letter-spacing: 0.08em;
  display: inline-flex;
  align-items: center;
  gap: 0.4rem;
  transition:
    transform 0.2s ease,
    background 0.2s ease;
  margin-bottom: 0.7rem;
}
.hero-love-btn:hover {
  transform: translateY(-2px);
  background: rgba(255, 255, 255, 0.1);
}
.hero-love-msg {
  font-size: 1.15rem;
  color: var(--blush);
  margin: 0 0 1.6rem;
  animation: revealP 0.5s ease forwards;
}

/* envelope */
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

/* letter overlay */
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

/* reading-progress ribbon: fills in gold as she scrolls through
   the letter, pinned to the very top edge of the book */
.letter-progress-bar {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 3px;
  background: rgba(140, 28, 77, 0.28);
  border-radius: 6px 6px 0 0;
  overflow: hidden;
  z-index: 7;
}
.letter-progress-fill {
  height: 100%;
  width: 0%;
  background: linear-gradient(90deg, var(--wine-soft), var(--gold-light), var(--wine-soft));
  box-shadow: 0 0 10px rgba(219, 163, 132, 0.65);
  transition: width 0.15s ease-out;
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
  font-family: 'Great Vibes', cursive;
  font-size: 1.75rem;
  line-height: 1.65;
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

/* bouquet svg */
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
  fill: var(--blush);
}
.wrap-paper-fold {
  fill: var(--pink-mid);
}
.filler-petal {
  fill: #ffffff;
  opacity: 0.95;
}
.filler-center {
  fill: #f4c440;
}
.ribbon-loop {
  fill: var(--gold-light);
}
.ribbon-knot {
  fill: var(--gold);
}

/* extra bouquet blooms: daisies, tulips, sparkles, hanging tag */
.daisy-petal {
  fill: #fffaf5;
  filter: drop-shadow(0 1px 1px rgba(0, 0, 0, 0.08));
}
.daisy-center {
  fill: #f4c440;
}
.tulip-petal {
  filter: drop-shadow(0 1px 1px rgba(0, 0, 0, 0.12));
}
.tulip-front {
  opacity: 0.98;
}
.tulip-side {
  opacity: 0.9;
}
.bouquet-sparkle {
  fill: #fff8f2;
  opacity: 0;
  animation: bouquetTwinkle 2.6s ease-in-out infinite;
  filter: drop-shadow(0 0 3px rgba(255, 248, 242, 0.9));
}
@keyframes bouquetTwinkle {
  0%,
  100% {
    opacity: 0;
    transform: scale(0.6);
  }
  50% {
    opacity: 1;
    transform: scale(1.2);
  }
}
.bouquet-tag {
  cursor: pointer;
}
.tag-thread {
  stroke: var(--gold);
  stroke-width: 1.2;
  opacity: 0.8;
}
.tag-body {
  fill: #fff8f2;
  stroke: var(--gold);
  stroke-width: 1;
  filter: drop-shadow(0 4px 8px rgba(0, 0, 0, 0.3));
  transition: transform 0.2s ease;
}
.bouquet-tag:hover .tag-body {
  transform: translateY(-2px);
}
.tag-text {
  font-family: 'Dancing Script', cursive;
  font-size: 13px;
  fill: var(--wine);
}

/* sections */
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
.gallery-love-counter {
  margin-top: 0.6rem;
  color: var(--gold-light);
  opacity: 0.9;
}

/* wishes */
.wishes-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
  gap: 1.6rem;
}
.wish-card {
  aspect-ratio: 3/4;
  perspective: 1200px;
  cursor: pointer;
}
.wish-card-inner {
  position: relative;
  width: 100%;
  height: 100%;
  transition: transform 0.7s cubic-bezier(0.4, 0.2, 0.2, 1);
  transform-style: preserve-3d;
}
.wish-card.is-flipped .wish-card-inner {
  transform: rotateY(180deg);
}
.wish-card-face {
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
.wish-card-front {
  background: linear-gradient(155deg, var(--wine-soft), var(--wine-deep));
  border: 1px solid rgba(246, 217, 196, 0.35);
  gap: 0.8rem;
}
.wish-glyph {
  font-size: 1.8rem;
  color: var(--gold-light);
}
.wish-card-back {
  background: var(--paper);
  color: var(--ink);
  transform: rotateY(180deg);
  font-size: 1.05rem;
  line-height: 1.4;
}

/* ═══════════════════════════════════════════════
   TITO JOKES
═══════════════════════════════════════════════ */
.corny-badge {
  display: inline-block;
  color: var(--wine-deep);
  background: linear-gradient(120deg, var(--gold-light), var(--gold));
  padding: 0.32rem 0.9rem;
  border-radius: 30px;
  font-size: 0.62rem;
  margin-bottom: 0.7rem;
  box-shadow: 0 6px 14px rgba(0, 0, 0, 0.28);
  opacity: 1;
}
.joke-wrap {
  max-width: 560px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.6rem;
}
.reward-tray {
  display: flex;
  gap: 0.7rem;
  flex-wrap: wrap;
  justify-content: center;
}
.reward-slot {
  width: 44px;
  height: 44px;
  border-radius: 50%;
  border: 1.5px dashed rgba(246, 217, 196, 0.4);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.3rem;
  background: rgba(255, 255, 255, 0.04);
  transition:
    transform 0.3s ease,
    border-color 0.3s ease,
    background 0.3s ease;
}
.reward-slot.is-filled {
  border-style: solid;
  border-color: var(--gold);
  background: radial-gradient(circle, rgba(246, 217, 196, 0.25), transparent 70%);
  transform: scale(1.12);
  animation: rewardPop 0.5s ease;
}
.reward-slot-empty {
  opacity: 0.3;
}
@keyframes rewardPop {
  0% {
    transform: scale(0.4);
    opacity: 0;
  }
  60% {
    transform: scale(1.3);
  }
  100% {
    transform: scale(1.12);
    opacity: 1;
  }
}
.joke-card {
  position: relative;
  width: 100%;
  background: linear-gradient(165deg, rgba(255, 243, 246, 0.09), rgba(74, 15, 48, 0.42));
  border: 1px solid rgba(246, 217, 196, 0.35);
  border-radius: 22px;
  padding: 2.3rem 2rem;
  backdrop-filter: blur(10px);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.1rem;
  text-align: center;
  box-shadow:
    0 24px 50px rgba(0, 0, 0, 0.4),
    inset 0 0 0 1px rgba(246, 217, 196, 0.08);
}
.joke-card::before {
  content: '';
  position: absolute;
  inset: 0;
  border-radius: 22px;
  padding: 1px;
  background: linear-gradient(
    140deg,
    var(--gold-light),
    transparent 35%,
    transparent 65%,
    var(--gold)
  );
  -webkit-mask:
    linear-gradient(#fff 0 0) content-box,
    linear-gradient(#fff 0 0);
  -webkit-mask-composite: xor;
  mask-composite: exclude;
  opacity: 0.55;
  pointer-events: none;
}
.joke-card-crest {
  display: flex;
  align-items: center;
  gap: 0.6rem;
  width: 100%;
  justify-content: center;
  opacity: 0.85;
}
.joke-card-crest-line {
  height: 1px;
  width: 40px;
  background: linear-gradient(90deg, transparent, var(--gold), transparent);
}
.joke-card-crest-icon {
  font-size: 1rem;
}
.quiz-progress {
  color: var(--gold-light);
}
.quiz-question {
  color: var(--gold-light);
  font-size: clamp(1.15rem, 3vw, 1.4rem);
  line-height: 1.5;
  margin: 0;
}
.quiz-options {
  display: flex;
  flex-direction: column;
  gap: 0.7rem;
  width: 100%;
}
.quiz-option {
  font-family: 'Jost', sans-serif;
  font-size: 0.92rem;
  padding: 0.75rem 1rem;
  border-radius: 12px;
  border: 1.5px solid rgba(246, 217, 196, 0.35);
  background: rgba(255, 255, 255, 0.05);
  color: var(--paper);
  transition:
    transform 0.2s ease,
    background 0.2s ease,
    border-color 0.2s ease;
}
.quiz-option:hover {
  transform: translateY(-2px);
  border-color: var(--gold);
  background: rgba(255, 255, 255, 0.09);
}
.joke-reveal-btn {
  padding: 0.85rem 2.2rem;
  font-size: 1rem;
  border-radius: 30px;
  background: linear-gradient(135deg, var(--wine-soft), var(--wine));
  border-color: var(--gold);
  color: var(--gold-light);
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.3);
}
.joke-reveal-btn:hover {
  transform: translateY(-2px) scale(1.03);
  box-shadow: 0 12px 24px rgba(0, 0, 0, 0.35);
}
.joke-line {
  font-family: 'Great Vibes', cursive;
  font-size: 1.7rem !important;
  line-height: 1.5;
  color: var(--gold-light) !important;
  animation: revealP 0.6s ease forwards;
}
.quiz-option.is-correct {
  background: linear-gradient(135deg, #4caf7d, #2e7d54);
  border-color: #7ee6ab;
  color: #fff;
}
.quiz-option.is-wrong {
  background: linear-gradient(135deg, #b23368, #7a1c46);
  border-color: #ff9db8;
  color: #fff;
  animation: quizShake 0.4s ease;
}
@keyframes quizShake {
  0%,
  100% {
    transform: translateX(0);
  }
  25% {
    transform: translateX(-6px);
  }
  75% {
    transform: translateX(6px);
  }
}
.quiz-feedback {
  font-family: 'Cormorant Garamond', serif;
  font-size: 1.05rem;
  margin: 0;
  opacity: 0.9;
}
.quiz-feedback.is-good {
  color: #b6f0cf;
}
.quiz-feedback.is-bad {
  color: #ffc4d6;
}
.quiz-complete-bouquet {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
  justify-content: center;
  font-size: 2rem;
}
.quiz-complete-bouquet span {
  animation: bouquetPopIn 0.5s ease both;
}
@keyframes bouquetPopIn {
  0% {
    transform: scale(0) rotate(-20deg);
    opacity: 0;
  }
  100% {
    transform: scale(1) rotate(0deg);
    opacity: 1;
  }
}
.quiz-complete-note {
  font-family: 'Cormorant Garamond', serif;
  font-size: 1.05rem;
  opacity: 0.85;
  max-width: 380px;
  line-height: 1.6;
}
.quiz-replay-btn {
  margin-top: 0.4rem;
  background: linear-gradient(135deg, var(--wine-soft), var(--wine));
  border: 1px solid var(--gold);
  color: var(--gold-light);
  border-radius: 30px;
  padding: 0.65rem 1.6rem;
  font-family: 'Jost', sans-serif;
  font-size: 0.85rem;
  letter-spacing: 0.08em;
  box-shadow: 0 8px 18px rgba(0, 0, 0, 0.25);
  transition:
    transform 0.25s ease,
    box-shadow 0.25s ease;
}
.quiz-replay-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 12px 22px rgba(0, 0, 0, 0.3);
}
@media (max-width: 520px) {
  .joke-card {
    padding: 1.7rem 1.3rem;
  }
  .reward-slot {
    width: 38px;
    height: 38px;
    font-size: 1.1rem;
  }
}

/* gallery */
.gallery-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(190px, 1fr));
  gap: 2.2rem 1.6rem;
  justify-items: center;
  perspective: 1000px;
}
.polaroid {
  position: relative;
  background: var(--paper);
  padding: 0.7rem 0.7rem 2rem;
  width: 190px;
  border-radius: 2px;
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.35);
  cursor: pointer;
  transform-style: preserve-3d;
  transition:
    transform 0.25s ease,
    box-shadow 0.3s ease;
}
.polaroid:hover {
  box-shadow: 0 18px 30px rgba(0, 0, 0, 0.45);
  z-index: 5;
}
.polaroid.is-liked {
  box-shadow:
    0 10px 20px rgba(0, 0, 0, 0.35),
    0 0 0 2px var(--gold);
}
.polaroid-like {
  position: absolute;
  top: 0.35rem;
  right: 0.35rem;
  z-index: 3;
  width: 30px;
  height: 30px;
  border-radius: 50%;
  border: none;
  background: rgba(30, 6, 20, 0.55);
  color: #ffdfe6;
  font-size: 1rem;
  display: flex;
  align-items: center;
  justify-content: center;
  backdrop-filter: blur(3px);
  transition:
    transform 0.2s ease,
    background 0.2s ease,
    color 0.2s ease;
}
.polaroid-like:hover {
  transform: scale(1.15);
}
.polaroid-like.is-liked {
  background: linear-gradient(135deg, var(--wine-soft), var(--wine));
  color: #ff8fa8;
  animation: likePop 0.4s ease;
}
@keyframes likePop {
  0% {
    transform: scale(0.6);
  }
  60% {
    transform: scale(1.3);
  }
  100% {
    transform: scale(1);
  }
}
.polaroid .frame {
  position: relative;
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
.frame-overlay {
  position: absolute;
  inset: 0;
  background: rgba(30, 6, 20, 0.32);
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  transition: opacity 0.25s ease;
}
.polaroid:hover .frame-overlay {
  opacity: 1;
}
.frame-overlay-icon {
  font-size: 1.4rem;
  filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.4));
}
.polaroid figcaption {
  display: flex;
  align-items: baseline;
  justify-content: center;
  gap: 0.35rem;
  text-align: center;
  font-family: 'Jost', sans-serif;
  font-size: 0.78rem;
  color: var(--ink);
  margin-top: 0.6rem;
  opacity: 0.75;
}
.polaroid-index {
  font-size: 0.65rem;
  letter-spacing: 0.08em;
  opacity: 0.55;
}

/* ═══════════════════════════════════════════════
   CINEMA — premium theatre experience
═══════════════════════════════════════════════ */
#film {
  max-width: 1020px;
}

.cinema-room {
  position: relative;
  background: linear-gradient(180deg, #06020a 0%, #0e0610 45%, #160a18 100%);
  border-radius: 22px;
  padding: 0 0 0;
  overflow: hidden;
  box-shadow:
    0 50px 100px rgba(0, 0, 0, 0.8),
    0 0 0 1px rgba(219, 163, 132, 0.14),
    inset 0 1px 0 rgba(255, 255, 255, 0.05);
}

/* ceiling glow */
.cinema-room::before {
  content: '';
  position: absolute;
  top: 0;
  left: 5%;
  right: 5%;
  height: 80px;
  background: radial-gradient(ellipse at 50% 0%, rgba(100, 18, 52, 0.55) 0%, transparent 70%);
  pointer-events: none;
  z-index: 0;
}

.cinema-arch-top {
  height: 4px;
  background: linear-gradient(
    90deg,
    transparent 0%,
    #7a3050 10%,
    #dba384 30%,
    #f6d9c4 50%,
    #dba384 70%,
    #7a3050 90%,
    transparent 100%
  );
  position: relative;
  z-index: 2;
}
.cinema-arch-bottom {
  height: 4px;
  background: linear-gradient(
    90deg,
    transparent 0%,
    #7a3050 10%,
    #dba384 30%,
    #f6d9c4 50%,
    #dba384 70%,
    #7a3050 90%,
    transparent 100%
  );
  border-radius: 0 0 22px 22px;
}

/* wall sconces */
.sconces {
  display: flex;
  justify-content: space-between;
  padding: 12px 28px 10px;
  position: relative;
  z-index: 2;
}
.sconce {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
}
.sconce-bracket {
  width: 2px;
  height: 10px;
  background: linear-gradient(180deg, rgba(122, 48, 80, 0.8), rgba(219, 163, 132, 0.6));
}
.sconce-bulb {
  width: 9px;
  height: 9px;
  border-radius: 50%;
  background: radial-gradient(circle at 35% 30%, #fff8f0, #f6d9c4 55%, #dba384);
  box-shadow:
    0 0 10px 5px rgba(246, 217, 196, 0.55),
    0 0 28px 10px rgba(219, 163, 132, 0.2);
  animation: sconcePulse 3.8s ease-in-out infinite;
}
@keyframes sconcePulse {
  0%,
  100% {
    opacity: 1;
    box-shadow:
      0 0 10px 5px rgba(246, 217, 196, 0.55),
      0 0 28px 10px rgba(219, 163, 132, 0.2);
  }
  50% {
    opacity: 0.78;
    box-shadow:
      0 0 6px 3px rgba(246, 217, 196, 0.38),
      0 0 18px 6px rgba(219, 163, 132, 0.12);
  }
}

/* stage layout */
.cinema-stage {
  position: relative;
  padding: 0 20px;
  z-index: 1;
}

/* curtain rod */
.curtain-rod {
  height: 9px;
  background: linear-gradient(
    90deg,
    #100408 0%,
    #5c1238 12%,
    #c47850 35%,
    #f6d9c4 50%,
    #c47850 65%,
    #5c1238 88%,
    #100408 100%
  );
  border-radius: 5px;
  position: relative;
  z-index: 5;
  box-shadow:
    0 3px 12px rgba(0, 0, 0, 0.8),
    0 0 20px rgba(219, 163, 132, 0.25);
  margin-bottom: -1px;
}
.rod-finial {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: radial-gradient(circle at 35% 30%, #f6d9c4, #dba384 55%, #8a5560);
  box-shadow: 0 0 10px rgba(219, 163, 132, 0.5);
}
.rod-finial-l {
  left: -7px;
}
.rod-finial-r {
  right: -7px;
}

/* stage inner holds curtains + bezel */
.stage-inner {
  position: relative;
}

/* curtains */
.curtain {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 80px;
  z-index: 4;
  pointer-events: none;
  transition:
    width 1.4s cubic-bezier(0.76, 0, 0.24, 1),
    opacity 1.4s ease;
}
.curtain-l {
  left: 0;
  background: linear-gradient(
    90deg,
    #2e0818 0%,
    #5c1238 30%,
    #8c1c4d 60%,
    rgba(140, 28, 77, 0) 100%
  );
  border-radius: 0 0 0 3px;
}
.curtain-r {
  right: 0;
  background: linear-gradient(
    270deg,
    #2e0818 0%,
    #5c1238 30%,
    #8c1c4d 60%,
    rgba(140, 28, 77, 0) 100%
  );
  border-radius: 0 0 3px 0;
}
/* curtain opens by squeezing width outward */
.curtain.curtain-open {
  width: 18px;
}
/* velvet fold lines */
.curtain-fold {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 1px;
}
.curtain-l .curtain-fold:nth-child(1) {
  left: 28px;
  background: rgba(255, 255, 255, 0.05);
}
.curtain-l .curtain-fold:nth-child(2) {
  left: 52px;
  background: rgba(0, 0, 0, 0.18);
}
.curtain-r .curtain-fold:nth-child(1) {
  right: 28px;
  background: rgba(255, 255, 255, 0.05);
}
.curtain-r .curtain-fold:nth-child(2) {
  right: 52px;
  background: rgba(0, 0, 0, 0.18);
}

/* spotlight cone during opening */
.spotlight {
  position: absolute;
  top: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 0;
  height: 0;
  border-left: 120px solid transparent;
  border-right: 120px solid transparent;
  border-top: 260px solid rgba(246, 217, 196, 0.07);
  z-index: 3;
  pointer-events: none;
  animation: spotFade 2.4s ease forwards;
  filter: blur(18px);
}
@keyframes spotFade {
  0% {
    opacity: 0;
  }
  20% {
    opacity: 1;
  }
  85% {
    opacity: 1;
  }
  100% {
    opacity: 0;
  }
}

/* the bezel */
.cinema-bezel {
  position: relative;
  background: #000;
  border-radius: 3px;
  border: 8px solid #0a0210;
  box-shadow:
    0 0 0 1px rgba(219, 163, 132, 0.2),
    0 0 0 3px #04010a,
    0 0 0 4px rgba(219, 163, 132, 0.07),
    inset 0 0 50px rgba(0, 0, 0, 0.65);
  overflow: hidden;
}

/* CRT scanlines */
.scanlines {
  position: absolute;
  inset: 0;
  background: repeating-linear-gradient(
    0deg,
    transparent,
    transparent 2px,
    rgba(0, 0, 0, 0.07) 2px,
    rgba(0, 0, 0, 0.07) 4px
  );
  pointer-events: none;
  z-index: 10;
}
/* glass glare */
.screen-glare {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 38%;
  background: linear-gradient(180deg, rgba(255, 255, 255, 0.045) 0%, transparent 100%);
  pointer-events: none;
  z-index: 11;
  border-radius: 2px 2px 0 0;
}
/* vignette */
.screen-vignette {
  position: absolute;
  inset: 0;
  background: radial-gradient(ellipse at 50% 50%, transparent 50%, rgba(0, 0, 0, 0.55) 100%);
  pointer-events: none;
  z-index: 9;
}

/* idle / opening screen */
.film-thumb {
  width: 100%;
  aspect-ratio: 16/9;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #100618 0%, #060208 50%, #0e0412 100%);
  cursor: pointer;
  position: relative;
}
.film-thumb--dark {
  cursor: default;
}
.film-poster {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
}

/* "now showing" text */
.opening-text {
  font-size: clamp(1.4rem, 3vw, 2rem);
  color: rgba(246, 217, 196, 0.7);
  letter-spacing: 0.12em;
  z-index: 2;
  animation: openingFade 2.4s ease forwards;
}
@keyframes openingFade {
  0% {
    opacity: 0;
    transform: scale(0.92);
  }
  25% {
    opacity: 1;
    transform: scale(1);
  }
  80% {
    opacity: 1;
  }
  100% {
    opacity: 0;
  }
}

/* pulsing rings around play button */
.cinema-play-wrap {
  position: relative;
  z-index: 2;
  display: flex;
  align-items: center;
  justify-content: center;
}
.pulse-ring {
  position: absolute;
  border-radius: 50%;
  border: 1.5px solid rgba(246, 217, 196, 0.28);
  animation: ringPulse 2.6s ease-out infinite;
}
.ring-1 {
  width: 100px;
  height: 100px;
  animation-delay: 0s;
}
.ring-2 {
  width: 140px;
  height: 140px;
  animation-delay: 0.7s;
}
.ring-3 {
  width: 180px;
  height: 180px;
  border-color: rgba(219, 163, 132, 0.14);
  animation-delay: 1.3s;
}
@keyframes ringPulse {
  0% {
    transform: scale(0.82);
    opacity: 0.9;
  }
  100% {
    transform: scale(1.22);
    opacity: 0;
  }
}

.play-btn {
  width: 52px;
  height: 52px;
  border-radius: 50%;
  background: rgba(255, 243, 246, 0.95);
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  z-index: 2;
}
.play-btn.big {
  width: 78px;
  height: 78px;
  box-shadow:
    0 0 32px rgba(246, 217, 196, 0.45),
    0 0 64px rgba(140, 28, 77, 0.3);
}
.play-btn svg {
  width: 28px;
  height: 28px;
  fill: var(--wine);
  margin-left: 4px;
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

.film-video {
  width: 100%;
  aspect-ratio: 16/9;
  display: block;
  background: #000;
}

/* floor + seats */
.cinema-floor-fade {
  height: 24px;
  background: linear-gradient(180deg, rgba(14, 6, 16, 0) 0%, rgba(6, 2, 10, 0.85) 100%);
}
.cinema-seats {
  display: flex;
  justify-content: center;
  padding: 8px 28px 18px;
  background: #04010a;
}
.seat-row {
  display: flex;
  gap: 8px;
  align-items: flex-end;
}
.seat {
  width: 32px;
  height: 28px;
  background: linear-gradient(180deg, #2e0d1c 0%, #190810 100%);
  border-radius: 5px 5px 0 0;
  border: 1px solid rgba(100, 24, 58, 0.35);
  position: relative;
}
.seat::before {
  content: '';
  position: absolute;
  top: -9px;
  left: 3px;
  right: 3px;
  height: 11px;
  background: linear-gradient(180deg, #3d1428, #2a0d1c);
  border-radius: 4px 4px 0 0;
  border: 1px solid rgba(100, 24, 58, 0.28);
  border-bottom: none;
}
.seat::after {
  content: '';
  position: absolute;
  top: 6px;
  left: 5px;
  right: 5px;
  height: 3px;
  background: rgba(255, 255, 255, 0.04);
  border-radius: 2px;
}

/* ═══════════════════════════════════════════════
   CAKE — premium two-tier drip cake
═══════════════════════════════════════════════ */
.cake-wrap {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.2rem;
}
.cake {
  width: 280px;
  max-width: 74vw;
  cursor: pointer;
  overflow: visible;
  filter: drop-shadow(0 22px 34px rgba(0, 0, 0, 0.45));
}
.cake-plate-shadow {
  fill: rgba(0, 0, 0, 0.35);
  filter: blur(4px);
}
.cake-plate-top {
  fill: rgba(255, 255, 255, 0.28);
}
.frosting-drip {
  filter: drop-shadow(0 2px 2px rgba(0, 0, 0, 0.12));
}
.ribbon-strip {
  filter: drop-shadow(0 1px 2px rgba(0, 0, 0, 0.25));
}
.cake-pearl {
  fill: var(--gold-light);
  opacity: 0.9;
}
.mini-rose-petal {
  filter: drop-shadow(0 1px 1px rgba(0, 0, 0, 0.15));
}
.cake-name {
  font-family: 'Great Vibes', cursive;
  font-size: 27px;
  fill: var(--gold-light);
  paint-order: stroke;
  stroke: rgba(74, 15, 48, 0.35);
  stroke-width: 0.6px;
}
.cake-name-underline {
  stroke: var(--gold-light);
  stroke-width: 1;
  opacity: 0.55;
}
.cake-age {
  font-family: 'Playfair Display', serif;
  font-size: 15px;
  letter-spacing: 0.18em;
  fill: var(--gold-light);
  opacity: 0.92;
}
.candle-stripe {
  fill: var(--wine-soft);
  opacity: 0.55;
}
.candle-drip {
  fill: #fff8f2;
  opacity: 0.9;
}
.flame-glow {
  fill: rgba(255, 184, 77, 0.35);
  filter: blur(2px);
  animation: flicker 1.4s ease-in-out infinite;
  transform-origin: 0 -18px;
}
.flame-group {
  transform-origin: 0 -18px;
}
.flame {
  animation: flicker 1.4s ease-in-out infinite;
  transform-origin: 0 -18px;
}
.flame-core {
  animation: flicker 1.4s ease-in-out infinite reverse;
  transform-origin: 0 -10px;
}
@keyframes flicker {
  0%,
  100% {
    transform: scale(1) skewX(0);
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

/* modals */
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
  display: flex;
  flex-direction: column;
  gap: 0.3rem;
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
  z-index: 32;
}

/* gallery lightbox extras: prev/next nav, counter, like button */
.gallery-lightbox {
  flex-direction: column;
}
.lightbox-fade {
  animation: fadeIn 0.35s ease;
}
.lightbox-nav {
  position: fixed;
  top: 50%;
  transform: translateY(-50%);
  width: 44px;
  height: 44px;
  border-radius: 50%;
  background: rgba(30, 6, 20, 0.55);
  border: 1px solid rgba(246, 217, 196, 0.35);
  color: var(--gold-light);
  font-size: 1.6rem;
  display: flex;
  align-items: center;
  justify-content: center;
  transition:
    transform 0.2s ease,
    background 0.2s ease;
  z-index: 31;
}
.lightbox-nav:hover {
  transform: translateY(-50%) scale(1.1);
  background: rgba(140, 28, 77, 0.65);
}
.lightbox-prev {
  left: 1.2rem;
}
.lightbox-next {
  right: 1.2rem;
}
.lightbox-counter {
  color: var(--gold-light);
  opacity: 0.7;
}
.lightbox-like {
  margin-top: 1rem;
  font-family: 'Jost', sans-serif;
  font-size: 0.85rem;
  letter-spacing: 0.06em;
  padding: 0.55rem 1.3rem;
  border-radius: 30px;
  border: 1px solid var(--gold);
  background: rgba(255, 255, 255, 0.06);
  color: var(--paper);
  transition:
    transform 0.2s ease,
    background 0.2s ease;
}
.lightbox-like:hover {
  transform: translateY(-2px);
}
.lightbox-like.is-liked {
  background: linear-gradient(135deg, var(--wine-soft), var(--wine));
  color: #ff8fa8;
  border-color: var(--gold-light);
}
@media (max-width: 600px) {
  .lightbox-nav {
    width: 36px;
    height: 36px;
    font-size: 1.3rem;
  }
  .lightbox-prev {
    left: 0.5rem;
  }
  .lightbox-next {
    right: 0.5rem;
  }
}

/* footer */
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
.footer-love-tally {
  margin-top: 1rem;
  color: var(--gold-light);
  opacity: 0.9;
}

/* ═══════════════════════════════════════════════
   SETTINGS — one small gear icon, tucked bottom-right,
   opens a compact panel for Music + Reaction recorder.
═══════════════════════════════════════════════ */
.settings-overlay {
  position: fixed;
  inset: 0;
  z-index: 16;
  background: transparent;
}
.settings-wrap {
  position: fixed;
  bottom: 1.4rem;
  right: 1.4rem;
  z-index: 17;
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 0.8rem;
}
.settings-toggle {
  width: 34px;
  height: 34px;
  border-radius: 50%;
  background: linear-gradient(155deg, var(--gold-light), var(--gold));
  border: 1px solid rgba(255, 255, 255, 0.4);
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.4);
  transition:
    transform 0.3s ease,
    box-shadow 0.25s ease;
}
.settings-toggle:hover {
  transform: translateY(-2px) scale(1.05);
  box-shadow: 0 8px 18px rgba(0, 0, 0, 0.45);
}
.settings-toggle.is-open {
  transform: rotate(35deg);
}
.settings-gear {
  width: 15px;
  height: 15px;
  fill: var(--wine-deep);
}
.settings-panel {
  width: 236px;
  max-width: 76vw;
  background: linear-gradient(165deg, rgba(178, 51, 104, 0.28), rgba(30, 6, 20, 0.94));
  border: 1px solid rgba(246, 217, 196, 0.3);
  border-radius: 16px;
  padding: 0.9rem 1rem;
  backdrop-filter: blur(14px);
  box-shadow: 0 20px 44px rgba(0, 0, 0, 0.45);
  display: flex;
  flex-direction: column;
  gap: 0.2rem;
}
.settings-row {
  display: flex;
  align-items: center;
  gap: 0.6rem;
  padding: 0.5rem 0.1rem;
}
.settings-row-icon {
  font-size: 1.1rem;
  width: 22px;
  text-align: center;
  flex-shrink: 0;
}
.settings-row-label {
  flex: 1;
  font-family: 'Jost', sans-serif;
  font-size: 0.82rem;
  letter-spacing: 0.02em;
  color: var(--gold-light);
}
.settings-divider {
  height: 1px;
  background: rgba(246, 217, 196, 0.18);
  margin: 0.2rem 0;
}
.settings-switch {
  position: relative;
  width: 40px;
  height: 22px;
  border-radius: 20px;
  background: rgba(255, 255, 255, 0.15);
  border: 1px solid rgba(246, 217, 196, 0.3);
  flex-shrink: 0;
  transition: background 0.25s ease;
}
.settings-switch.is-on {
  background: linear-gradient(135deg, var(--wine-soft), var(--wine));
  border-color: var(--gold);
}
.settings-switch-knob {
  position: absolute;
  top: 2px;
  left: 2px;
  width: 16px;
  height: 16px;
  border-radius: 50%;
  background: var(--gold-light);
  transition: transform 0.25s ease;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
}
.settings-switch.is-on .settings-switch-knob {
  transform: translateX(18px);
}
.settings-action-btn {
  font-family: 'Jost', sans-serif;
  font-size: 0.72rem;
  letter-spacing: 0.05em;
  text-transform: uppercase;
  padding: 0.35rem 0.7rem;
  border-radius: 20px;
  border: 1px solid var(--gold);
  background: linear-gradient(135deg, var(--wine-soft), var(--wine));
  color: var(--gold-light);
  flex-shrink: 0;
  transition:
    transform 0.2s ease,
    opacity 0.2s ease;
}
.settings-action-btn:hover:not(:disabled) {
  transform: translateY(-1px);
}
.settings-action-btn:disabled {
  opacity: 0.5;
  cursor: default;
}
.settings-error {
  font-family: 'Jost', sans-serif;
  font-size: 0.7rem;
  color: #ffc4d6;
  margin: 0.4rem 0 0.1rem;
  line-height: 1.4;
}
.settings-pop-enter-active {
  transition:
    opacity 0.22s ease,
    transform 0.22s cubic-bezier(0.2, 0.8, 0.3, 1.1);
}
.settings-pop-leave-active {
  transition:
    opacity 0.16s ease,
    transform 0.16s ease;
}
.settings-pop-enter-from,
.settings-pop-leave-to {
  opacity: 0;
  transform: translateY(8px) scale(0.96);
}

/* ═══════════════════════════════════════════════
   REACTION CAM — live bubble + review panel
═══════════════════════════════════════════════ */

/* live self-view bubble, stays put while she scrolls */
.camera-bubble {
  position: fixed;
  bottom: 1.4rem;
  left: 1.4rem;
  z-index: 18;
  width: 132px;
  height: 132px;
  border-radius: 50%;
  overflow: hidden;
  border: 2px solid var(--gold);
  box-shadow:
    0 10px 26px rgba(0, 0, 0, 0.5),
    0 0 0 5px rgba(219, 163, 132, 0.15);
  background: #05010a;
  animation: bubbleIn 0.4s cubic-bezier(0.2, 0.8, 0.3, 1.1);
}
@keyframes bubbleIn {
  0% {
    opacity: 0;
    transform: scale(0.7);
  }
  100% {
    opacity: 1;
    transform: scale(1);
  }
}
.camera-bubble-video {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
  transform: scaleX(-1);
  background: #000;
}
.camera-bubble-rec {
  position: absolute;
  top: 8px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  align-items: center;
  gap: 0.32rem;
  background: rgba(0, 0, 0, 0.55);
  border-radius: 20px;
  padding: 0.2rem 0.55rem;
  font-family: 'Jost', sans-serif;
  font-size: 0.66rem;
  letter-spacing: 0.04em;
  color: #ffdfe6;
}
.camera-bubble-stop {
  position: absolute;
  bottom: 8px;
  left: 50%;
  transform: translateX(-50%);
  width: 30px;
  height: 30px;
  border-radius: 50%;
  background: rgba(0, 0, 0, 0.6);
  border: 1.5px solid rgba(255, 255, 255, 0.6);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: transform 0.2s ease;
}
.camera-bubble-stop:hover {
  transform: translateX(-50%) scale(1.08);
}
.stop-square {
  width: 11px;
  height: 11px;
  border-radius: 2px;
  background: #ff4d6d;
}
.rec-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: #ff4d6d;
  animation: recBlink 1.1s ease-in-out infinite;
}
@keyframes recBlink {
  0%,
  100% {
    opacity: 1;
  }
  50% {
    opacity: 0.25;
  }
}

/* review panel after stopping */
.camera-panel-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(30, 6, 20, 0.8);
  backdrop-filter: blur(3px);
  z-index: 25;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 1.5rem;
  animation: fadeIn 0.3s ease;
  overflow-y: auto;
}
.camera-panel {
  position: relative;
  width: 420px;
  max-width: 92vw;
  background: linear-gradient(165deg, rgba(178, 51, 104, 0.22), rgba(30, 6, 20, 0.92));
  border: 1px solid rgba(246, 217, 196, 0.3);
  border-radius: 20px;
  padding: 2.2rem 1.8rem 1.8rem;
  backdrop-filter: blur(14px);
  box-shadow: 0 30px 60px rgba(0, 0, 0, 0.5);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
  text-align: center;
  margin: auto;
}
.camera-panel-close {
  position: absolute;
  top: 0.8rem;
  right: 1rem;
  background: none;
  border: none;
  color: var(--gold-light);
  font-size: 1.6rem;
  opacity: 0.85;
}
.camera-panel-title {
  font-size: 1.8rem;
  color: var(--gold-light);
  margin: 0;
}
.camera-panel-sub {
  font-size: 0.92rem;
  opacity: 0.75;
  line-height: 1.5;
  max-width: 340px;
  margin: -0.4rem 0 0;
}
.camera-stage {
  position: relative;
  width: 100%;
  aspect-ratio: 4/3;
  background: #05010a;
  border-radius: 14px;
  overflow: hidden;
  border: 1px solid rgba(246, 217, 196, 0.25);
  box-shadow: inset 0 0 40px rgba(0, 0, 0, 0.5);
}
.camera-video {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
  background: #000;
}
.camera-controls {
  display: flex;
  gap: 0.7rem;
  flex-wrap: wrap;
  justify-content: center;
}
.rec-secondary-btn {
  background: rgba(255, 255, 255, 0.06);
}
@media (max-width: 520px) {
  .settings-toggle {
    width: 30px;
    height: 30px;
  }
  .settings-gear {
    width: 13px;
    height: 13px;
  }
  .settings-panel {
    width: 208px;
  }
  .camera-bubble {
    width: 104px;
    height: 104px;
  }
  .camera-panel {
    padding: 1.8rem 1.3rem 1.5rem;
  }
}

@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.001s !important;
    transition-duration: 0.2s !important;
  }
}

@media (max-width: 1024px) {
  section {
    padding: 4rem 1.4rem;
  }
}

@media (max-width: 768px) {
  section {
    padding: 3.5rem 1.2rem;
  }
  .section-head {
    margin-bottom: 2.2rem;
  }
  .wishes-grid {
    grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
    gap: 1.2rem;
  }
  .cinema-room {
    border-radius: 14px;
  }
  .sconces {
    padding: 10px 16px 8px;
  }
  .cinema-stage {
    padding: 0 12px;
  }
  .curtain {
    width: 56px;
  }
  .curtain.curtain-open {
    width: 12px;
  }
  .seat {
    width: 26px;
    height: 22px;
  }
  .seat-row {
    gap: 5px;
  }
}

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
  .wishes-grid {
    grid-template-columns: repeat(2, 1fr);
  }
  .book-inside {
    padding: 2rem 1.4rem;
  }
  .letter-p {
    font-size: 1.45rem;
  }
  .cake {
    width: 240px;
    max-width: 80vw;
  }
  .bouquet-reveal {
    width: 230px;
  }
  .cinema-room {
    border-radius: 10px;
  }
  .sconces {
    padding: 8px 12px 6px;
  }
  .cinema-stage {
    padding: 0 8px;
  }
  .curtain {
    width: 40px;
  }
  .curtain.curtain-open {
    width: 8px;
  }
  .seat {
    width: 20px;
    height: 18px;
  }
  .seat-row {
    gap: 4px;
  }
  .cinema-seats {
    padding: 6px 16px 14px;
  }
  .ring-1 {
    width: 78px;
    height: 78px;
  }
  .ring-2 {
    width: 110px;
    height: 110px;
  }
  .ring-3 {
    width: 142px;
    height: 142px;
  }
}

@media (max-width: 360px) {
  .wishes-grid {
    grid-template-columns: 1fr;
  }
  .envelope {
    height: 170px;
  }
}
</style>
