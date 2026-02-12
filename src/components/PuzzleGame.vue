<template>
  <div v-if="!showUniverseHub && !showBirthdayScreen && !showAsteroidGame && !showMazeGame && !showVault" class="p-3 sm:p-4 flex flex-col items-center justify-center gap-4 retro-bg min-h-screen">
    <!-- Title -->
    <h1 class="font-press-start text-xl sm:text-2xl text-pink-400 mb-2 mt-10">PHOTO PUZZLE</h1>

    <!-- Difficulty -->
    <div class="flex flex-col gap-3 items-center mb-3" :class="{ 'sm:flex-row': !isSmallScreen }">
      <label class="flex items-center gap-2 font-press-start text-[10px] text-pink-200">
        LEVEL
        <select v-model.number="gridSize" class="retro-select">
          <option :value="2">EASY (2×2)</option>
          <option :value="3">NORMAL (3×3)</option>
          <option :value="4">HARD (4×4)</option>
          <option :value="5">INSANE (5×5)</option>
        </select>
      </label>

      <button @click="initGame" class="retro-btn">RESTART</button>
      <button @click="returnToHub" class="retro-btn">BACK TO HUB</button>
    </div>

    <!-- Timer -->
    <p class="font-press-start text-pink-200 text-xs">
      TIME LEFT: <span :class="{ 'text-red-400': timeLeft < 10 }">{{ timeLeft }}</span> s
    </p>

    <!-- Puzzle board -->
    <div
      ref="board"
      :style="{ width: boardSize + 'px', height: boardSize + 'px' }"
      class="relative border-4 border-pink-400 crt-border"
    >
      <div
        v-for="tile in tiles"
        :key="tile.id"
        class="tile absolute cursor-pointer"
        :class="{ 'tile-selected': selected === tile.index }"
        :style="tileStyle(tile)"
        @click="onTileClick(tile.index)"
      />
    </div>

    <!-- Win Modal -->
    <div
      v-if="showWinModal"
      class="fixed inset-0 bg-black/90 flex items-center justify-center z-50"
    >
      <div class="retro-modal text-center animate-blink">
        <h2 class="font-press-start text-lg sm:text-xl text-pink-400 mb-3">YOU DID IT! 🎉</h2>
        <p class="text-pink-200 text-[10px] sm:text-xs mb-3">The puzzle is complete.</p>
        <p class="text-yellow-300 font-press-start text-sm sm:text-base mb-4">
          HAPPY BIRTHDAY ANGEL❤️
        </p>
        <div class="flex flex-col gap-3 justify-center" :class="{ 'sm:flex-row': !isSmallScreen }">
          <button @click="unlockUniverse" class="retro-btn">UNLOCK THE VAULT</button>
          <button @click="initGame" class="retro-btn">PLAY AGAIN</button>
        </div>
      </div>
    </div>

    <!-- Game Over Modal -->
    <div
      v-if="showLoseModal"
      class="fixed inset-0 bg-black/90 flex items-center justify-center z-50"
    >
      <div class="retro-modal text-center">
        <h2 class="font-press-start text-lg sm:text-xl text-red-400 mb-3">GAME OVER!</h2>
        <p class="text-pink-200 text-[10px] sm:text-xs mb-4">Time's up! Try again?</p>
        <button @click="initGame" class="retro-btn">PLAY AGAIN</button>
      </div>
    </div>

    <p class="mt-3 text-[10px] text-pink-200 font-press-start">
      Click tiles to swap and complete the picture!
    </p>
  </div>

  <!-- Glitch Transition -->
  <div v-if="showGlitch" class="fixed inset-0 bg-black flex items-center justify-center z-60 glitch">
    <div class="glitch-text font-press-start text-2xl text-pink-400">ENTERING NEW UNIVERSE...</div>
  </div>

  <!-- Birthday Special Screen -->
  <div v-if="showBirthdayScreen" class="p-3 sm:p-4 flex flex-col items-center justify-center gap-4 galaxy-bg min-h-screen">
    <div class="fireworks"></div>
    <div class="flex items-center justify-center w-full max-w-xs" :class="{ 'sm:flex-row': !isSmallScreen }">
      <button @click="showBirthdayScreen = false; showUniverseHub = true" class="retro-btn">ENTER ARCADE UNIVERSE</button>
    </div>
  </div>

  <!-- Universe Hub -->
  <div v-if="showUniverseHub" class="p-3 sm:p-4 flex flex-col items-center justify-center gap-4 galaxy-bg min-h-screen">
    <h1 class="font-press-start text-xl sm:text-2xl text-yellow-300 mb-4">ARCADE UNIVERSE</h1>
    <div class="flex flex-col gap-3 w-full max-w-xs">
      <button @click="returnToPuzzle" class="retro-btn">🧩 PUZZLE MODE</button>
      <button @click="startAsteroidGame" class="retro-btn">🚀 ASTEROID LOVE</button>
      <button @click="startMazeGame" class="retro-btn">🌌 STAR MAZE</button>
    </div>
    <div v-for="(note, index) in loveNotes" :key="index" class="heart" :style="heartStyle(index)" @click="showLoveNote(index)"></div>
    <div v-if="showNote" class="retro-modal text-center">
      <p class="font-press-start text-[10px] sm:text-xs text-pink-200 mb-3">{{ currentNote }}</p>
      <button @click="showNote = false" class="retro-btn">CLOSE</button>
    </div>
    <p v-if="secretUnlocked" class="font-press-start text-[10px] sm:text-xs text-yellow-300 mt-4 animate-blink">
      SECRET UNLOCKED: You're my forever player one! ❤️
    </p>
  </div>

  <!-- Asteroid Love Game -->
  <div v-if="showAsteroidGame" class="p-3 sm:p-4 flex flex-col items-center justify-center gap-4 galaxy-bg min-h-screen">
    <h1 class="font-press-start text-xl sm:text-2xl text-yellow-300 mb-4">ASTEROID LOVE</h1>
    <p class="font-press-start text-[10px] sm:text-xs text-pink-200">Level: {{ asteroidLevel }} Score: {{ score }}</p>
    <canvas
      ref="gameCanvas"
      :width="gameWidth"
      :height="gameHeight"
      class="border-4 border-pink-400 crt-border"
    ></canvas>
    <div class="flex flex-col gap-3 w-full max-w-xs" :class="{ 'sm:flex-row': !isSmallScreen }">
      <button @click="startAsteroidGame" class="retro-btn">RESTART</button>
      <button @click="returnToHub" class="retro-btn">BACK TO HUB</button>
    </div>
    <p class="mt-3 text-[10px] text-pink-200 font-press-start">
      Use ←→ to move, Space to shoot hearts!
    </p>

    <!-- Game Over Modal for Asteroid -->
    <div v-if="showAsteroidLose" class="fixed inset-0 bg-black/90 flex items-center justify-center z-50">
      <div class="retro-modal text-center">
        <h2 class="font-press-start text-lg sm:text-xl text-red-400 mb-3">GAME OVER!</h2>
        <p class="text-pink-200 text-[10px] sm:text-xs mb-4">Asteroid hit! Try again?</p>
        <button @click="startAsteroidGame" class="retro-btn">TRY AGAIN</button>
      </div>
    </div>

    <!-- Win Modal for Asteroid -->
    <div v-if="showAsteroidWin" class="fixed inset-0 bg-black/90 flex items-center justify-center z-50">
      <div class="retro-modal text-center animate-blink">
        <h2 class="font-press-start text-lg sm:text-xl text-pink-400 mb-3">YOU WIN! 🎉</h2>
        <p class="text-pink-200 text-[10px] sm:text-xs mb-4">All levels cleared!</p>
        <button @click="returnToHub" class="retro-btn">BACK TO HUB</button>
      </div>
    </div>
  </div>

  <!-- Star Maze Game -->
  <div v-if="showMazeGame" class="p-3 sm:p-4 flex flex-col items-center justify-center gap-4 galaxy-bg min-h-screen">
    <h1 class="font-press-start text-xl sm:text-2xl text-yellow-300 mb-4">STAR MAZE</h1>
    <p class="font-press-start text-[10px] sm:text-xs text-pink-200">Level: {{ mazeLevel }} Hearts: {{ mazeScore }} Time: {{ mazeTime }}</p>
    <canvas
      ref="mazeCanvas"
      :width="mazeWidth"
      :height="mazeHeight"
      class="border-4 border-pink-400 crt-border"
    ></canvas>
    <div class="flex flex-col gap-3 w-full max-w-xs" :class="{ 'sm:flex-row': !isSmallScreen }">
      <button @click="startMazeGame" class="retro-btn">RESTART</button>
      <button @click="returnToHub" class="retro-btn">BACK TO HUB</button>
    </div>
    <p class="mt-3 text-[10px] text-pink-200 font-press-start">
      Use ↑↓←→ to navigate and collect hearts!
    </p>

    <!-- Game Over Modal for Maze -->
    <div v-if="showMazeLose" class="fixed inset-0 bg-black/90 flex items-center justify-center z-50">
      <div class="retro-modal text-center">
        <h2 class="font-press-start text-lg sm:text-xl text-red-400 mb-3">GAME OVER!</h2>
        <p class="text-pink-200 text-[10px] sm:text-xs mb-4">Time's up! Try again?</p>
        <button @click="startMazeGame" class="retro-btn">TRY AGAIN</button>
      </div>
    </div>

    <!-- Win Modal for Maze -->
    <div v-if="showMazeWin" class="fixed inset-0 bg-black/90 flex items-center justify-center z-50">
      <div class="retro-modal text-center animate-blink">
        <h2 class="font-press-start text-lg sm:text-xl text-pink-400 mb-3">YOU WIN! 🎉</h2>
        <p class="text-pink-200 text-[10px] sm:text-xs mb-4">All levels cleared!</p>
        <button @click="returnToHub" class="retro-btn">BACK TO HUB</button>
      </div>
    </div>

    <!-- Easter Egg Modal for Maze -->
    <div v-if="showEasterEgg" class="fixed inset-0 bg-black/90 flex items-center justify-center z-50">
      <div class="retro-modal text-center">
        <h2 class="font-press-start text-lg sm:text-xl text-yellow-300 mb-3">EASTER EGG FOUND!</h2>
        <p class="text-pink-200 text-[10px] sm:text-xs mb-4">I pray to God that all your dreams come true. love you mama liam
        </p>
        <button @click="showEasterEgg = false" class="retro-btn">CLOSE</button>
      </div>
    </div>
  </div>

  <!-- Vault Screen -->
  <div v-if="showVault" class="p-3 sm:p-4 flex flex-col items-center justify-center gap-4 galaxy-bg min-h-screen relative overflow-hidden">
    <h1 class="font-press-start text-xl sm:text-2xl text-yellow-300 mb-4 z-10">THE VAULT</h1>
    
    <!-- PIN Input Section -->
    <div v-if="!letterRead" class="flex flex-col items-center gap-4 z-10">
      <p class="font-press-start text-[10px] sm:text-xs text-pink-200">Enter the 6-digit PIN to unlock:</p>
      <div class="flex gap-1 sm:gap-2 justify-center">
        <input
          v-for="(digit, index) in pinDigits"
          :key="index"
          v-model="pinDigits[index]"
          type="text"
          maxlength="1"
          class="retro-select w-6 sm:w-8 text-center text-lg"
          @input="moveToNext($event, index)"
          @keydown.backspace="moveToPrevious($event, index)"
        />
      </div>
      <button @click="checkVaultPassword" class="retro-btn mt-4">UNLOCK</button>
      <div class="mt-4 text-center">
        <p class="font-press-start text-[8px] sm:text-[10px] text-pink-200">Hint 1: Starts with the day of birth (two digits)</p>
        <p class="font-press-start text-[8px] sm:text-[10px] text-pink-200">Hint 2: Followed by the month (two digits)</p>
        <p class="font-press-start text-[8px] sm:text-[10px] text-pink-200">Hint 3: Ends with the last two digits of the year of birth</p>
      </div>
    </div>

    <!-- Reveal Bouquet Button (Appears after reading the letter) -->
    <div v-if="letterRead && !showBouquet" class="z-20 mt-8 animate-bounce">
      <button @click="revealBouquet" class="retro-btn text-sm !bg-gradient-to-r !from-pink-500 !to-rose-500 !border-yellow-300 !shadow-[0_0_15px_#ff4da6]">
        💐 REVEAL BOUQUET 💐
      </button>
    </div>

    <!-- Love Letter Modal -->
    <div v-if="showLoveLetter" class="fixed inset-0 bg-black/90 flex items-center justify-center z-50 p-4">
      <div class="old-paper-modal text-center max-w-lg max-h-[80vh] overflow-y-auto">
        <h2 class="text-2xl sm:text-3xl text-rose-700 mb-4 font-old-script">My wife</h2>
        <p class="text-[14px] sm:text-base text-brown-800 mb-6 leading-relaxed font-serif">
          i will love you forever. you are my the mother of children, my wife and partner in crime.
          <br><br>
          sometimes life doesnt go as planned we all get scared but i promise you i will always be there and always love you my love
          <br><br>
          i want us to grow old together live on our farm and never be scared of anything baby daddy loves you so much
          <span class="text-rose-600 font-bold">Happy Birthday, mama liam. May your day be as beautiful and special as you are.</span>
        </p>
        <button @click="closeLoveLetter" class="retro-btn">CLOSE LETTER</button>
      </div>
    </div>

    <!-- Bouquet Animation -->
    <div v-if="showBouquet" class="bouquet-container" @click="hideBouquet">
      <div class="bouquet-wrapper">
        <div class="bouquet-glow"></div>
        <div class="bouquet-flowers">
          <!-- Central Rose -->
          <div class="flower rose center">
            <div class="petal"></div>
            <div class="petal"></div>
            <div class="petal"></div>
            <div class="center-pistil"></div>
          </div>
          <!-- Surrounding Flowers -->
          <div class="flower rose left">
            <div class="petal"></div>
            <div class="petal"></div>
            <div class="petal"></div>
          </div>
          <div class="flower rose right">
            <div class="petal"></div>
            <div class="petal"></div>
            <div class="petal"></div>
          </div>
          <div class="flower daisy top-left">
            <div class="daisy-petal"></div>
            <div class="daisy-petal"></div>
            <div class="daisy-petal"></div>
            <div class="daisy-petal"></div>
            <div class="daisy-petal"></div>
            <div class="daisy-center"></div>
          </div>
          <div class="flower daisy top-right">
            <div class="daisy-petal"></div>
            <div class="daisy-petal"></div>
            <div class="daisy-petal"></div>
            <div class="daisy-petal"></div>
            <div class="daisy-petal"></div>
            <div class="daisy-center"></div>
          </div>
          <!-- Baby's Breath -->
          <div class="babys-breath bb-1"></div>
          <div class="babys-breath bb-2"></div>
          <div class="babys-breath bb-3"></div>
          <div class="babys-breath bb-4"></div>
          <div class="babys-breath bb-5"></div>
          <!-- Leaves -->
          <div class="leaf leaf-1"></div>
          <div class="leaf leaf-2"></div>
          <div class="leaf leaf-3"></div>
          <div class="leaf leaf-4"></div>
          <!-- Ribbon -->
          <div class="ribbon">
            <div class="ribbon-tail left"></div>
            <div class="ribbon-tail right"></div>
            <div class="ribbon-knot"></div>
          </div>
        </div>
        <div class="bouquet-text font-press-start text-pink-300 text-xs mt-4 text-center drop-shadow-[0_0_10px_#ff4da6]">
          For You, my love 🌹
        </div>
        <p class="text-[10px] text-pink-200 mt-2 animate-pulse">Click anywhere to hide</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, nextTick, onMounted, onUnmounted, watch } from 'vue'

const board = ref(null)
const gridSize = ref(3)
const defaultImage = '/patricia.jpg'
const boardSize = ref(window.innerWidth < 640 ? Math.min(window.innerWidth - 32, 320) : 360)
const tiles = reactive([])
const selected = ref(null)
const solved = ref(false)
const timeLeft = ref(60)
let timer = null
const showWinModal = ref(false)
const showLoseModal = ref(false)
const showGlitch = ref(false)
const showUniverseHub = ref(false)
const showBirthdayScreen = ref(true)
const showNote = ref(false)
const currentNote = ref('')
const secretUnlocked = ref(false)
const isSmallScreen = ref(window.innerWidth < 360)
const showAsteroidGame = ref(false)
const gameCanvas = ref(null)
const gameWidth = ref(window.innerWidth < 640 ? Math.min(window.innerWidth - 32, 320) : 360)
const gameHeight = ref(240)
const score = ref(0)
const asteroidLevel = ref(1)
const showAsteroidLose = ref(false)
const showAsteroidWin = ref(false)
const showMazeGame = ref(false)
const mazeCanvas = ref(null)
const mazeWidth = ref(window.innerWidth < 640 ? Math.min(window.innerWidth - 32, 320) : 360)
const mazeHeight = ref(240)
const mazeScore = ref(0)
const mazeLevel = ref(1)
const mazeTime = ref(60)
let mazeTimer = null
const showMazeLose = ref(false)
const showMazeWin = ref(false)
const showEasterEgg = ref(false)
const showVault = ref(false)
const pinDigits = ref(['', '', '', '', '', ''])
const showLoveLetter = ref(false)
const letterRead = ref(false)
const showBouquet = ref(false)

const loveNotes = [
  'Your laugh is my favorite 8-bit melody! 🎵',
  'You make every level of life an adventure! 🌟',
  'My heart glitches every time you\'re near! ❤️'
]

// Puzzle Game Logic
function uid(x = '') {
  return Math.random().toString(36).slice(2, 9) + x
}

function buildTiles() {
  tiles.splice(0, tiles.length)
  const size = gridSize.value
  for (let row = 0; row < size; row++) {
    for (let col = 0; col < size; col++) {
      const index = row * size + col
      tiles.push({
        id: uid(index),
        index,
        correctIndex: index,
        row,
        col,
        url: defaultImage,
      })
    }
  }
}

function shuffleTiles() {
  const size = gridSize.value
  const indices = Array.from({ length: size * size }, (_, i) => i)
  for (let i = indices.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[indices[i], indices[j]] = [indices[j], indices[i]]
  }
  tiles.forEach((t, idx) => {
    t.index = indices[idx]
    t.row = Math.floor(t.index / size)
    t.col = t.index % size
  })
}

function tileStyle(tile) {
  const size = gridSize.value
  const tileSize = boardSize.value / size
  const correctRow = Math.floor(tile.correctIndex / size)
  const correctCol = tile.correctIndex % size
  const bgX = -(correctCol * tileSize)
  const bgY = -(correctRow * tileSize)
  const left = tile.col * tileSize
  const top = tile.row * tileSize

  return {
    width: tileSize + 'px',
    height: tileSize + 'px',
    left: left + 'px',
    top: top + 'px',
    backgroundImage: `url("${tile.url}")`,
    backgroundSize: `${boardSize.value}px ${boardSize.value}px`,
    backgroundPosition: `${bgX}px ${bgY}px`,
    transition: 'left .22s ease, top .22s ease',
    border: '1px solid #FF4DA6',
    imageRendering: 'pixelated',
  }
}

function onTileClick(idx) {
  if (solved.value || showWinModal.value || showLoseModal.value) return
  if (selected.value === null) {
    selected.value = idx
    return
  }
  if (selected.value === idx) {
    selected.value = null
    return
  }

  const a = tiles.find((t) => t.index === selected.value)
  const b = tiles.find((t) => t.index === idx)
  if (!a || !b) {
    selected.value = null
    return
  }

  ;[a.index, b.index] = [b.index, a.index]
  a.row = Math.floor(a.index / gridSize.value)
  a.col = a.index % gridSize.value
  b.row = Math.floor(b.index / gridSize.value)
  b.col = b.index % gridSize.value

  selected.value = null
  checkSolved()
}

function checkSolved() {
  const ok = tiles.every((t) => t.index === t.correctIndex)
  if (ok) {
    solved.value = true
    clearInterval(timer)
    setTimeout(() => {
      showWinModal.value = true
    }, 2000)
  }
}

function startTimer() {
  clearInterval(timer)
  timeLeft.value = gridSize.value === 2 ? 15 : 60
  timer = setInterval(() => {
    if (timeLeft.value > 0) {
      timeLeft.value--
    } else {
      clearInterval(timer)
      showLoseModal.value = true
    }
  }, 1000)
}

function initGame() {
  solved.value = false
  selected.value = null
  showWinModal.value = false
  showLoseModal.value = false
  buildTiles()
  nextTick(() => shuffleTiles())
  startTimer()
}

// Universe Navigation
function unlockUniverse() {
  showWinModal.value = false
  showVault.value = true
}

function checkVaultPassword() {
  const pin = pinDigits.value.join('')
  if (pin === '140202') {
    showLoveLetter.value = true
  } else {
    alert('Incorrect PIN! Try again.')
  }
}

function closeLoveLetter() {
  showLoveLetter.value = false
  letterRead.value = true
}

function revealBouquet() {
  showBouquet.value = true
  setTimeout(() => {
    if (showBouquet.value) showBouquet.value = false
  }, 10000)
}

function hideBouquet() {
  showBouquet.value = false
}

function moveToNext(event, index) {
  if (event.target.value && index < 5) {
    const nextInput = event.target.nextSibling
    if (nextInput) nextInput.focus()
  }
}

function moveToPrevious(event, index) {
  if (event.target.value === '' && index > 0) {
    const prevInput = event.target.previousSibling
    if (prevInput) prevInput.focus()
  }
}

function returnToPuzzle() {
  showUniverseHub.value = false
  showAsteroidGame.value = false
  showMazeGame.value = false
  showVault.value = false
  showBirthdayScreen.value = false
  initGame()
}

function returnToHub() {
  showAsteroidGame.value = false
  showMazeGame.value = false
  showVault.value = false
  showBirthdayScreen.value = false
  showUniverseHub.value = true
}

// Asteroid Love Game Logic
const player = reactive({ x: gameWidth.value / 2 - 15, y: gameHeight.value - 30, width: 30, height: 20 })
const hearts = reactive([])
const asteroids = reactive([])
let gameLoop = null
let lastShot = 0

function startAsteroidGame() {
  showUniverseHub.value = false
  showAsteroidGame.value = true
  showMazeGame.value = false
  showVault.value = false
  score.value = 0
  asteroidLevel.value = 1
  showAsteroidLose.value = false
  showAsteroidWin.value = false
  player.x = gameWidth.value / 2 - 15
  player.y = gameHeight.value - 30
  hearts.splice(0)
  asteroids.splice(0)
  spawnAsteroids()
  if (gameLoop) cancelAnimationFrame(gameLoop)
  gameLoop = requestAnimationFrame(updateGame)
}

function spawnAsteroids() {
  if (asteroids.length < 5 + asteroidLevel.value) {
    asteroids.push({
      x: Math.random() * (gameWidth.value - 30),
      y: -20,
      width: 20,
      height: 20,
      speed: 1 + Math.random() * asteroidLevel.value
    })
  }
}

function updateGame() {
  const ctx = gameCanvas.value.getContext('2d')
  ctx.clearRect(0, 0, gameWidth.value, gameHeight.value)

  // Draw player
  ctx.fillStyle = '#00ffea'
  ctx.beginPath()
  ctx.moveTo(player.x + player.width / 2, player.y)
  ctx.lineTo(player.x, player.y + player.height)
  ctx.lineTo(player.x + player.width, player.y + player.height)
  ctx.closePath()
  ctx.fill()

  // Update and draw hearts
  hearts.forEach((heart, index) => {
    heart.y -= 3
    ctx.fillStyle = '#ff4da6'
    ctx.beginPath()
    ctx.moveTo(heart.x + 5, heart.y + 2)
    ctx.quadraticCurveTo(heart.x + 5, heart.y, heart.x + 2.5, heart.y)
    ctx.quadraticCurveTo(heart.x, heart.y, heart.x, heart.y + 2.5)
    ctx.quadraticCurveTo(heart.x, heart.y + 5, heart.x + 2.5, heart.y + 7)
    ctx.quadraticCurveTo(heart.x + 5, heart.y + 5, heart.x + 7.5, heart.y + 7)
    ctx.quadraticCurveTo(heart.x + 10, heart.y + 5, heart.x + 10, heart.y + 2.5)
    ctx.quadraticCurveTo(heart.x + 10, heart.y, heart.x + 7.5, heart.y)
    ctx.closePath()
    ctx.fill()
    if (heart.y < 0) hearts.splice(index, 1)
  })

  // Update and draw asteroids, check collision with player
  let gameOver = false
  asteroids.forEach((asteroid, index) => {
    asteroid.y += asteroid.speed
    ctx.fillStyle = '#ffffff'
    ctx.beginPath()
    ctx.arc(asteroid.x + asteroid.width / 2, asteroid.y + asteroid.height / 2, asteroid.width / 2, 0, Math.PI * 2)
    ctx.fill()

    // Check player collision
    if (
      asteroid.x < player.x + player.width &&
      asteroid.x + asteroid.width > player.x &&
      asteroid.y < player.y + player.height &&
      asteroid.y + asteroid.height > player.y
    ) {
      gameOver = true
    }

    if (asteroid.y > gameHeight.value) asteroids.splice(index, 1)
  })

  if (gameOver) {
    cancelAnimationFrame(gameLoop)
    showAsteroidLose.value = true
    return
  }

  // Collision detection hearts and asteroids
  hearts.forEach((heart, hIndex) => {
    asteroids.forEach((asteroid, aIndex) => {
      if (
        heart.x < asteroid.x + asteroid.width &&
        heart.x + 10 > asteroid.x &&
        heart.y < asteroid.y + asteroid.height &&
        heart.y + 7 > asteroid.y
      ) {
        asteroids.splice(aIndex, 1)
        hearts.splice(hIndex, 1)
        score.value += 10
      }
    })
  })

  // Level up
  if (score.value >= asteroidLevel.value * 50) {
    asteroidLevel.value++
    if (asteroidLevel.value > 5) {
      cancelAnimationFrame(gameLoop)
      showAsteroidWin.value = true
      return
    }
  }

  // Spawn new asteroids
  if (Math.random() < 0.02 * asteroidLevel.value) spawnAsteroids()

  gameLoop = requestAnimationFrame(updateGame)
}

// Star Maze Game Logic
const mazePlayer = reactive({ x: 1, y: 1, width: 10, height: 10 })
const mazeHearts = reactive([])
const cellSize = 20
const mazeRows = Math.floor(mazeHeight.value / cellSize)
const mazeCols = Math.floor(mazeWidth.value / cellSize)
let maze = []
let mazeLoop = null
let easterEggPosition = { x: 0, y: 0 }

function generateMaze() {
  maze = Array(mazeRows).fill().map(() => Array(mazeCols).fill(1))
  function carve(x, y) {
    maze[y][x] = 0
    const directions = [[0, -1], [1, 0], [0, 1], [-1, 0]]
    directions.sort(() => Math.random() - 0.5)
    for (const [dx, dy] of directions) {
      const nx = x + dx * 2
      const ny = y + dy * 2
      if (nx >= 0 && nx < mazeCols && ny >= 0 && ny < mazeRows && maze[ny][nx] === 1) {
        maze[y + dy][x + dx] = 0
        carve(nx, ny)
      }
    }
  }
  carve(1, 1)
  // Place player in a valid path cell
  let px, py
  do {
    px = Math.floor(Math.random() * mazeCols)
    py = Math.floor(Math.random() * mazeRows)
  } while (maze[py][px] !== 0)
  mazePlayer.x = px * cellSize + (cellSize - mazePlayer.width) / 2
  mazePlayer.y = py * cellSize + (cellSize - mazePlayer.height) / 2
  // Place hearts
  mazeHearts.splice(0)
  for (let i = 0; i < 5 * mazeLevel.value; i++) {
    let x, y
    do {
      x = Math.floor(Math.random() * mazeCols)
      y = Math.floor(Math.random() * mazeRows)
    } while (maze[y][x] !== 0 || (x === px && y === py))
    mazeHearts.push({ x: x * cellSize + 5, y: y * cellSize + 5 })
  }
  // Place easter egg only in level 4
  if (mazeLevel.value === 4) {
    let ex, ey
    do {
      ex = Math.floor(Math.random() * mazeCols)
      ey = Math.floor(Math.random() * mazeRows)
    } while (maze[ey][ex] !== 0 || (ex === px && ey === py))
    easterEggPosition = { x: ex * cellSize, y: ey * cellSize }
  } else {
    easterEggPosition = { x: -100, y: -100 }
  }
}

function startMazeGame() {
  showUniverseHub.value = false
  showAsteroidGame.value = false
  showMazeGame.value = true
  showVault.value = false
  mazeScore.value = 0
  mazeLevel.value = 1
  mazeTime.value = 60
  showMazeLose.value = false
  showMazeWin.value = false
  showEasterEgg.value = false
  generateMaze()
  clearInterval(mazeTimer)
  mazeTimer = setInterval(() => {
    if (mazeTime.value > 0) {
      mazeTime.value--
    } else {
      clearInterval(mazeTimer)
      cancelAnimationFrame(mazeLoop)
      showMazeLose.value = true
    }
  }, 1000)
  if (mazeLoop) cancelAnimationFrame(mazeLoop)
  mazeLoop = requestAnimationFrame(updateMaze)
}

function updateMaze() {
  const ctx = mazeCanvas.value.getContext('2d')
  ctx.clearRect(0, 0, mazeWidth.value, mazeHeight.value)

  // Draw maze
  for (let y = 0; y < mazeRows; y++) {
    for (let x = 0; x < mazeCols; x++) {
      ctx.fillStyle = maze[y][x] === 1 ? '#ffffff' : '#000000'
      ctx.fillRect(x * cellSize, y * cellSize, cellSize, cellSize)
    }
  }

  // Draw hearts
  mazeHearts.forEach((heart) => {
    ctx.fillStyle = '#ff4da6'
    ctx.beginPath()
    ctx.moveTo(heart.x + 5, heart.y + 2)
    ctx.quadraticCurveTo(heart.x + 5, heart.y, heart.x + 2.5, heart.y)
    ctx.quadraticCurveTo(heart.x, heart.y, heart.x, heart.y + 2.5)
    ctx.quadraticCurveTo(heart.x, heart.y + 5, heart.x + 2.5, heart.y + 7)
    ctx.quadraticCurveTo(heart.x + 5, heart.y + 5, heart.x + 7.5, heart.y + 7)
    ctx.quadraticCurveTo(heart.x + 10, heart.y + 5, heart.x + 10, heart.y + 2.5)
    ctx.quadraticCurveTo(heart.x + 10, heart.y, heart.x + 7.5, heart.y)
    ctx.closePath()
    ctx.fill()
  })

  // Draw player
  ctx.fillStyle = '#00ffea'
  ctx.fillRect(mazePlayer.x, mazePlayer.y, mazePlayer.width, mazePlayer.height)

  // Check for easter egg in level 4
  if (mazeLevel.value === 4 && !showEasterEgg.value) {
    if (
      mazePlayer.x < easterEggPosition.x + cellSize &&
      mazePlayer.x + mazePlayer.width > easterEggPosition.x &&
      mazePlayer.y < easterEggPosition.y + cellSize &&
      mazePlayer.y + mazePlayer.height > easterEggPosition.y
    ) {
      showEasterEgg.value = true
    }
  }

  // Collision detection for hearts
  mazeHearts.forEach((heart, index) => {
    if (
      mazePlayer.x < heart.x + 10 &&
      mazePlayer.x + mazePlayer.width > heart.x &&
      mazePlayer.y < heart.y + 7 &&
      mazePlayer.y + mazePlayer.height > heart.y
    ) {
      mazeHearts.splice(index, 1)
      mazeScore.value += 10
    }
  })

  // Check win level
  if (mazeHearts.length === 0) {
    mazeLevel.value++
    if (mazeLevel.value > 5) {
      clearInterval(mazeTimer)
      cancelAnimationFrame(mazeLoop)
      showMazeWin.value = true
      return
    }
    mazeTime.value = 60
    generateMaze()
  }

  mazeLoop = requestAnimationFrame(updateMaze)
}

function handleKeydown(event) {
  if (showAsteroidGame.value) {
    if (event.key === 'ArrowLeft' && player.x > 0) {
      player.x -= 5
    } else if (event.key === 'ArrowRight' && player.x < gameWidth.value - player.width) {
      player.x += 5
    } else if (event.key === ' ' && Date.now() - lastShot > 300) {
      hearts.push({ x: player.x + player.width / 2 - 5, y: player.y })
      lastShot = Date.now()
    }
  } else if (showMazeGame.value) {
    const speed = 5
    let newX = mazePlayer.x
    let newY = mazePlayer.y
    if (event.key === 'ArrowLeft') newX -= speed
    if (event.key === 'ArrowRight') newX += speed
    if (event.key === 'ArrowUp') newY -= speed
    if (event.key === 'ArrowDown') newY += speed

    const cellX = Math.floor(newX / cellSize)
    const cellY = Math.floor(newY / cellSize)
    const cellX2 = Math.floor((newX + mazePlayer.width - 1) / cellSize)
    const cellY2 = Math.floor((newY + mazePlayer.height - 1) / cellSize)

    if (
      newX >= 0 && newX + mazePlayer.width <= mazeWidth.value &&
      newY >= 0 && newY + mazePlayer.height <= mazeHeight.value &&
      maze[cellY][cellX] === 0 && maze[cellY][cellX2] === 0 &&
      maze[cellY2][cellX] === 0 && maze[cellY2][cellX2] === 0
    ) {
      mazePlayer.x = newX
      mazePlayer.y = newY
    }
  }
}

// Easter Eggs
function heartStyle(index) {
  const positions = [
    { top: '10%', left: '10%' },
    { top: '20%', right: '10%' },
    { bottom: '10%', left: '15%' }
  ]
  return {
    position: 'absolute',
    ...positions[index],
    width: '20px',
    height: '20px',
    background: 'url("data:image/svg+xml,%3Csvg xmlns=\'http://www.w3.org/2000/svg\' viewBox=\'0 0 24 24\' fill=\'%23FF4DA6\'%3E%3Cpath d=\'M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z\'/%3E%3C/svg%3E")',
    backgroundSize: 'contain',
    cursor: 'pointer'
  }
}

function showLoveNote(index) {
  currentNote.value = loveNotes[index]
  showNote.value = true
}

// Secret Code
let konamiCode = ['ArrowUp', 'ArrowUp', 'ArrowDown', 'ArrowDown', 'ArrowLeft', 'ArrowRight', 'ArrowLeft', 'ArrowRight', 'b', 'a']
let konamiProgress = 0

function checkKonamiCode(event) {
  if (event.key === konamiCode[konamiProgress]) {
    konamiProgress++
    if (konamiProgress === konamiCode.length) {
      secretUnlocked.value = true
      konamiProgress = 0
    }
  } else {
    konamiProgress = 0
  }
}

watch([gridSize], () => {
  initGame()
})

onMounted(() => {
  initGame()
  window.addEventListener('resize', () => {
    boardSize.value = window.innerWidth < 640 ? Math.min(window.innerWidth - 32, 320) : 360
    gameWidth.value = window.innerWidth < 640 ? Math.min(window.innerWidth - 32, 320) : 360
    mazeWidth.value = window.innerWidth < 640 ? Math.min(window.innerWidth - 32, 320) : 360
    isSmallScreen.value = window.innerWidth < 360
  })
  window.addEventListener('keydown', checkKonamiCode)
  window.addEventListener('keydown', handleKeydown)
  const audio = new Audio('https://cdn.pixabay.com/download/audio/2023/12/14/audio_0c0e1b3c92.mp3?filename=chiptune-167267.mp3')
  audio.loop = true
  audio.volume = 0.3
  audio.play()
})

onUnmounted(() => {
  window.removeEventListener('keydown', checkKonamiCode)
  window.removeEventListener('keydown', handleKeydown)
  if (gameLoop) cancelAnimationFrame(gameLoop)
  if (mazeLoop) cancelAnimationFrame(mazeLoop)
  clearInterval(mazeTimer)
})
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Great+Vibes&display=swap');

.font-press-start {
  font-family: 'Press Start 2P', cursive;
}

.font-old-script {
  font-family: 'Great Vibes', cursive;
}

/* Retro Background */
.retro-bg {
  background: radial-gradient(circle at top, #2a0044, #000);
  color: #fff;
  position: relative;
  overflow: hidden;
}
.retro-bg::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: repeating-linear-gradient(
    0deg,
    rgba(0, 0, 0, 0.1),
    rgba(0, 0, 0, 0.1) 2px,
    transparent 2px,
    transparent 4px
  );
  pointer-events: none;
}

/* Neon Galaxy Background */
.galaxy-bg {
  background: linear-gradient(180deg, #0a0015, #000022);
  color: #fff;
  position: relative;
  overflow: hidden;
}
.galaxy-bg::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: url('data:image/svg+xml,%3Csvg xmlns="http://www.w3.org/2000/svg"%3E%3Cdefs%3E%3Cfilter id="glow"%3E%3CfeGaussianBlur stdDeviation="2.5" result="coloredBlur"/%3E%3CfeMerge%3E%3CfeMergeNode in="coloredBlur"/%3E%3CfeMergeNode in="SourceGraphic"/%3E%3C/feMerge%3E%3C/filter%3E%3C/defs%3E%3Cg filter="url(%23glow)"%3E%3Ccircle cx="10%" cy="10%" r="2" fill="%23fff"/%3E%3Ccircle cx="80%" cy="20%" r="1.5" fill="%23fff"/%3E%3Ccircle cx="50%" cy="70%" r="1" fill="%23fff"/%3E%3C/g%3E%3C/svg%3E'),
    repeating-linear-gradient(
      0deg,
      rgba(255, 255, 255, 0.05),
      rgba(255, 255, 255, 0.05) 1px,
      transparent 1px,
      transparent 3px
    );
  pointer-events: none;
}

/* CRT Effect */
.crt-border {
  box-shadow: 0 0 20px #ff4da6, inset 0 0 10px #ff4da6;
  border-radius: 6px;
  position: relative;
  overflow: hidden;
}
.crt-border::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: radial-gradient(circle, rgba(255, 255, 255, 0.05), transparent);
  pointer-events: none;
}

/* Glitch Transition */
.glitch {
  animation: glitch 0.3s infinite;
}
.glitch-text {
  position: relative;
}
.glitch-text::before,
.glitch-text::after {
  content: 'ENTERING NEW UNIVERSE...';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  color: #ff4da6;
}
.glitch-text::before {
  color: #00ffea;
  animation: glitch-top 0.4s infinite;
}
.glitch-text::after {
  color: #ff00ff;
  animation: glitch-bottom 0.4s infinite;
}
@keyframes glitch {
  0% { transform: translate(0); }
  20% { transform: translate(-2px, 2px); }
  40% { transform: translate(-2px, -2px); }
  60% { transform: translate(2px, 2px); }
  80% { transform: translate(2px, -2px); }
  100% { transform: translate(0); }
}
@keyframes glitch-top {
  0% { clip-path: polygon(0 0, 100% 0, 100% 5%, 0 5%); transform: translate(-2px, -1px); }
  100% { clip-path: polygon(0 0, 100% 0, 100% 5%, 0 5%); transform: translate(2px, 1px); }
}
@keyframes glitch-bottom {
  0% { clip-path: polygon(0 95%, 100% 95%, 100% 100%, 0 100%); transform: translate(2px, 1px); }
  100% { clip-path: polygon(0 95%, 100% 95%, 100% 100%, 0 100%); transform: translate(-2px, -1px); }
}

/* Fireworks Animation */
.fireworks {
  width: 100%;
  height: 120px;
  position: relative;
  overflow: hidden;
}
.fireworks::before,
.fireworks::after {
  content: '';
  position: absolute;
  width: 100%;
  height: 100%;
  background: url('data:image/svg+xml,%3Csvg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"%3E%3Cg%3E%3Ccircle cx="20" cy="50" r="3" fill="%23ff4da6" opacity="0" style="animation: firework 1.5s infinite; animation-delay: 0s;"/%3E%3Ccircle cx="50" cy="30" r="4" fill="%23ffff00" opacity="0" style="animation: firework 1.5s infinite; animation-delay: 0.5s;"/%3E%3Ccircle cx="80" cy="60" r="3" fill="%2300ffea" opacity="0" style="animation: firework 1.5s infinite; animation-delay: 1s;"/%3E%3C/g%3E%3C/svg%3E');
  background-size: cover;
}
.fireworks::before {
  animation: sparkle 2s infinite;
}
.fireworks::after {
  animation: sparkle 2s infinite 0.5s;
}
@keyframes firework {
  0% { opacity: 0; transform: scale(0); }
  50% { opacity: 1; transform: scale(1); }
  100% { opacity: 0; transform: scale(1.5); }
}
@keyframes sparkle {
  0% { transform: translateX(0); opacity: 0.7; }
  100% { transform: translateX(20%); opacity: 0.7; }
}

/* Scrolling Text */
@keyframes scroll {
  0% { transform: translateY(20px); opacity: 0; }
  10% { opacity: 1; }
  90% { opacity: 1; }
  100% { transform: translateY(-20px); opacity: 0; }
}
.animate-scroll {
  animation: scroll 5s infinite;
}

/* Retro Buttons */
.retro-btn {
  font-family: 'Press Start 2P', cursive;
  background: linear-gradient(145deg, #ff4da6, #ff1a6d);
  color: #fff;
  padding: 0.4rem 0.8rem;
  font-size: 0.55rem;
  border: 2px solid #fff;
  border-radius: 4px;
  text-shadow: 1px 1px #000;
  box-shadow: 0 4px #660033, inset 0 0 8px rgba(255, 255, 255, 0.3);
  transition: transform 0.1s ease, box-shadow 0.1s ease, background 0.2s ease;
}
.retro-btn:hover:not(.disabled) {
  background: linear-gradient(145deg, #ff6db6, #ff4da6);
}
.retro-btn:active:not(.disabled) {
  transform: translateY(2px);
  box-shadow: 0 2px #660033;
}
.retro-btn.disabled {
  background: #555;
  cursor: not-allowed;
}

/* Retro Select */
.retro-select {
  background: #111;
  border: 2px solid #ff4da6;
  color: #ffb6e6;
  padding: 0.3rem 0.5rem;
  font-size: 0.55rem;
  font-family: 'Press Start 2P', cursive;
  border-radius: 4px;
  box-shadow: 0 0 8px #ff4da6;
}

/* Retro Modal */
.retro-modal {
  background: linear-gradient(145deg, #111, #222);
  border: 3px solid #ff4da6;
  padding: 1rem;
  border-radius: 8px;
  box-shadow: 0 0 20px #ff4da6, inset 0 0 8px rgba(255, 77, 166, 0.5);
  max-width: 95%;
  font-size: clamp(0.6rem, 2vw, 1rem);
}

/* Old Paper Modal */
.old-paper-modal {
  background-color: #f4e8d1;
  border: 3px solid #d2b48c;
  padding: 1.5rem;
  border-radius: 8px;
  box-shadow: 0 0 20px #d2b48c, inset 0 0 10px rgba(210, 180, 140, 0.5);
  max-width: 95%;
  color: #4b3a2a;
  text-align: left;
  font-style: italic;
}

/* Tile Styles */
.tile {
  transition: transform 0.1s ease;
}
.tile:active {
  transform: scale(0.95);
}
.tile-selected {
  outline: 2px dashed #ffff00;
  transform: scale(1.05);
  z-index: 20;
  box-shadow: 0 0 12px #ffff00;
}

/* Blinking Animation */
@keyframes blink {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.6; }
}
.animate-blink {
  animation: blink 1.2s infinite;
}

/* Bouquet Animation Styles */
.bouquet-container {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  top: 0;
  display: flex;
  align-items: flex-end;
  justify-content: center;
  z-index: 100;
  background: radial-gradient(circle at center bottom, rgba(255, 77, 166, 0.2), transparent 70%);
  animation: fadeIn 0.5s ease-out;
  cursor: pointer;
}

.bouquet-wrapper {
  position: relative;
  width: 300px;
  height: 500px;
  animation: riseUp 1.5s cubic-bezier(0.34, 1.56, 0.64, 1) forwards;
  transform-origin: bottom center;
}

@keyframes riseUp {
  0% {
    transform: translateY(100%) scale(0.5);
    opacity: 0;
  }
  60% {
    transform: translateY(-10%) scale(1.05);
  }
  100% {
    transform: translateY(0) scale(1);
    opacity: 1;
  }
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.bouquet-glow {
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 200px;
  height: 200px;
  background: radial-gradient(circle, rgba(255, 77, 166, 0.4), transparent 70%);
  filter: blur(20px);
  animation: pulseGlow 2s ease-in-out infinite;
}

@keyframes pulseGlow {
  0%, 100% { transform: translateX(-50%) scale(1); opacity: 0.6; }
  50% { transform: translateX(-50%) scale(1.2); opacity: 1; }
}

.bouquet-flowers {
  position: relative;
  width: 100%;
  height: 100%;
  filter: drop-shadow(0 10px 20px rgba(0,0,0,0.5));
}

/* Rose Styles */
.flower.rose {
  position: absolute;
  width: 80px;
  height: 80px;
}

.flower.rose.center {
  bottom: 100px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 10;
}

.flower.rose.left {
  bottom: 120px;
  left: 20%;
  transform: rotate(-20deg) scale(0.8);
  z-index: 5;
}

.flower.rose.right {
  bottom: 120px;
  right: 20%;
  transform: rotate(20deg) scale(0.8);
  z-index: 5;
}

.petal {
  position: absolute;
  width: 40px;
  height: 40px;
  background: linear-gradient(135deg, #ff1a6d, #ff4da6);
  border-radius: 50% 50% 50% 0;
  transform-origin: bottom left;
  box-shadow: inset -2px -2px 4px rgba(0,0,0,0.2);
}

.flower.rose .petal:nth-child(1) {
  transform: rotate(0deg) translate(10px, 10px);
  background: linear-gradient(135deg, #ff0066, #ff3388);
}

.flower.rose .petal:nth-child(2) {
  transform: rotate(72deg) translate(10px, 10px);
  background: linear-gradient(135deg, #ff1a75, #ff4d94);
}

.flower.rose .petal:nth-child(3) {
  transform: rotate(144deg) translate(10px, 10px);
  background: linear-gradient(135deg, #ff3388, #ff66aa);
}

.center-pistil {
  position: absolute;
  width: 20px;
  height: 20px;
  background: radial-gradient(circle, #ffd700, #ffaa00);
  border-radius: 50%;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  box-shadow: 0 0 10px rgba(255, 215, 0, 0.8);
}

/* Daisy Styles */
.flower.daisy {
  position: absolute;
  width: 60px;
  height: 60px;
}

.flower.daisy.top-left {
  bottom: 160px;
  left: 15%;
  transform: rotate(-30deg) scale(0.7);
}

.flower.daisy.top-right {
  bottom: 160px;
  right: 15%;
  transform: rotate(30deg) scale(0.7);
}

.daisy-petal {
  position: absolute;
  width: 25px;
  height: 8px;
  background: linear-gradient(90deg, #fff5e6, #ffffff);
  border-radius: 50%;
  top: 50%;
  left: 50%;
  transform-origin: center;
  box-shadow: 0 1px 2px rgba(0,0,0,0.1);
}

.daisy-petal:nth-child(1) { transform: translate(-50%, -50%) rotate(0deg) translateX(15px); }
.daisy-petal:nth-child(2) { transform: translate(-50%, -50%) rotate(72deg) translateX(15px); }
.daisy-petal:nth-child(3) { transform: translate(-50%, -50%) rotate(144deg) translateX(15px); }
.daisy-petal:nth-child(4) { transform: translate(-50%, -50%) rotate(216deg) translateX(15px); }
.daisy-petal:nth-child(5) { transform: translate(-50%, -50%) rotate(288deg) translateX(15px); }

.daisy-center {
  position: absolute;
  width: 15px;
  height: 15px;
  background: radial-gradient(circle, #ffcc00, #ff9900);
  border-radius: 50%;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

/* Baby's Breath */
.babys-breath {
  position: absolute;
  width: 8px;
  height: 8px;
  background: radial-gradient(circle, #ffffff, #e6e6fa);
  border-radius: 50%;
  box-shadow: 0 0 5px rgba(255,255,255,0.8);
}

.bb-1 { bottom: 140px; left: 35%; animation: float 3s ease-in-out infinite; }
.bb-2 { bottom: 180px; left: 45%; animation: float 3s ease-in-out infinite 0.5s; }
.bb-3 { bottom: 150px; right: 35%; animation: float 3s ease-in-out infinite 1s; }
.bb-4 { bottom: 190px; left: 25%; animation: float 3s ease-in-out infinite 1.5s; }
.bb-5 { bottom: 170px; right: 25%; animation: float 3s ease-in-out infinite 2s; }

@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-5px); }
}

/* Leaves */
.leaf {
  position: absolute;
  width: 40px;
  height: 60px;
  background: linear-gradient(135deg, #228b22, #006400);
  border-radius: 0 100% 0 100%;
  opacity: 0.9;
}

.leaf-1 {
  bottom: 80px;
  left: 30%;
  transform: rotate(-45deg);
}

.leaf-2 {
  bottom: 80px;
  right: 30%;
  transform: rotate(45deg) scaleX(-1);
}

.leaf-3 {
  bottom: 60px;
  left: 40%;
  transform: rotate(-30deg) scale(0.8);
}

.leaf-4 {
  bottom: 60px;
  right: 40%;
  transform: rotate(30deg) scaleX(-1) scale(0.8);
}

/* Ribbon */
.ribbon {
  position: absolute;
  bottom: 40px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 20;
}

.ribbon-knot {
  width: 30px;
  height: 20px;
  background: linear-gradient(135deg, #ff1493, #ff69b4);
  border-radius: 50%;
  position: relative;
  box-shadow: 0 2px 4px rgba(0,0,0,0.3);
}

.ribbon-tail {
  position: absolute;
  width: 20px;
  height: 60px;
  background: linear-gradient(135deg, #ff1493, #ff69b4);
  top: 10px;
  border-radius: 0 0 50% 50%;
}

.ribbon-tail.left {
  left: -15px;
  transform: rotate(-20deg);
  animation: ribbonWaveLeft 2s ease-in-out infinite;
}

.ribbon-tail.right {
  right: -15px;
  transform: rotate(20deg);
  animation: ribbonWaveRight 2s ease-in-out infinite;
}

@keyframes ribbonWaveLeft {
  0%, 100% { transform: rotate(-20deg); }
  50% { transform: rotate(-30deg); }
}

@keyframes ribbonWaveRight {
  0%, 100% { transform: rotate(20deg); }
  50% { transform: rotate(30deg); }
}

.bouquet-text {
  position: absolute;
  bottom: 10px;
  left: 50%;
  transform: translateX(-50%);
  white-space: nowrap;
  text-shadow: 0 0 20px #ff4da6, 0 0 40px #ff4da6;
  animation: textGlow 2s ease-in-out infinite;
}

@keyframes textGlow {
  0%, 100% { opacity: 1; transform: translateX(-50%) scale(1); }
  50% { opacity: 0.8; transform: translateX(-50%) scale(1.05); }
}

/* Responsive Adjustments */
@media (max-width: 360px) {
  .retro-btn {
    padding: 0.4rem 0.6rem;
    font-size: 0.5rem;
    margin-bottom: 0.5rem;
  }
  .retro-select {
    padding: 0.2rem 0.4rem;
    font-size: 0.5rem;
  }
  .retro-modal {
    padding: 0.8rem;
  }
}

@media (max-width: 640px) {
  .bouquet-wrapper {
    transform: scale(0.8);
    transform-origin: bottom center;
  }
}
</style>