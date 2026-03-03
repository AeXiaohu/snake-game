<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

// 游戏配置
const config = {
  gridSize: 20,
  cellSize: 20,
  speed: 100
}

// 游戏状态
const snake = ref([])
const food = ref(null)
const direction = ref('right')
const nextDirection = ref('right')
const gameRunning = ref(false)
const score = ref(0)
const highScore = ref(0)
const gameOver = ref(false)
const gameStarted = ref(false)

// 触摸控制变量
let touchStartX = 0
let touchStartY = 0

// 初始化游戏
function initGame() {
  // 重置蛇的位置
  snake.value = [
    { x: 10, y: 10 },
    { x: 9, y: 10 },
    { x: 8, y: 10 }
  ]
  
  // 重置方向
  direction.value = 'right'
  nextDirection.value = 'right'
  
  // 生成食物
  generateFood()
  
  // 重置游戏状态
  score.value = 0
  gameRunning.value = true
  gameOver.value = false
  gameStarted.value = true
}

// 开始游戏
function startGame() {
  initGame()
}

// 生成食物
function generateFood() {
  const emptyCells = []
  
  // 遍历所有格子，找出空位置
  for (let y = 0; y < config.gridSize; y++) {
    for (let x = 0; x < config.gridSize; x++) {
      const isOccupied = snake.value.some(cell => cell.x === x && cell.y === y)
      if (!isOccupied) {
        emptyCells.push({ x, y })
      }
    }
  }
  
  // 随机选择一个空位置
  if (emptyCells.length > 0) {
    const randomIndex = Math.floor(Math.random() * emptyCells.length)
    food.value = emptyCells[randomIndex]
  }
}

// 移动蛇
function moveSnake() {
  if (!gameRunning.value) return
  
  // 更新方向
  direction.value = nextDirection.value
  
  // 计算新头部位置
  const head = { ...snake.value[0] }
  
  switch (direction.value) {
    case 'up':
      head.y--
      break
    case 'down':
      head.y++
      break
    case 'left':
      head.x--
      break
    case 'right':
      head.x++
      break
  }
  
  // 检查边界碰撞
  if (head.x < 0 || head.x >= config.gridSize || head.y < 0 || head.y >= config.gridSize) {
    gameOver.value = true
    gameRunning.value = false
    updateHighScore()
    return
  }
  
  // 检查自身碰撞
  if (snake.value.some(cell => cell.x === head.x && cell.y === head.y)) {
    gameOver.value = true
    gameRunning.value = false
    updateHighScore()
    return
  }
  
  // 检查是否吃到食物
  if (food.value && head.x === food.value.x && head.y === food.value.y) {
    // 吃到食物，增加分数，生成新食物
    score.value++
    generateFood()
  } else {
    // 没吃到食物，移除尾部
    snake.value.pop()
  }
  
  // 添加新头部
  snake.value.unshift(head)
}

// 更新最高分
function updateHighScore() {
  if (score.value > highScore.value) {
    highScore.value = score.value
    localStorage.setItem('snakeHighScore', highScore.value)
  }
}

// 处理键盘事件
function handleKeyDown(e) {
  switch (e.key) {
    case 'ArrowUp':
    case 'w':
    case 'W':
      if (direction.value !== 'down') {
        nextDirection.value = 'up'
      }
      break
    case 'ArrowDown':
    case 's':
    case 'S':
      if (direction.value !== 'up') {
        nextDirection.value = 'down'
      }
      break
    case 'ArrowLeft':
    case 'a':
    case 'A':
      if (direction.value !== 'right') {
        nextDirection.value = 'left'
      }
      break
    case 'ArrowRight':
    case 'd':
    case 'D':
      if (direction.value !== 'left') {
        nextDirection.value = 'right'
      }
      break
    case ' ': // 空格键开始/暂停
      if (gameOver.value) {
        initGame()
      } else if (!gameStarted.value) {
        startGame()
      } else {
        gameRunning.value = !gameRunning.value
      }
      break
  }
}

// 处理手机端控制按钮点击
function handleMobileControl(dir) {
  if (!gameRunning.value) return
  
  switch (dir) {
    case 'up':
      if (direction.value !== 'down') {
        nextDirection.value = 'up'
      }
      break
    case 'down':
      if (direction.value !== 'up') {
        nextDirection.value = 'down'
      }
      break
    case 'left':
      if (direction.value !== 'right') {
        nextDirection.value = 'left'
      }
      break
    case 'right':
      if (direction.value !== 'left') {
        nextDirection.value = 'right'
      }
      break
  }
}

// 游戏循环
let gameLoop = null

function startGameLoop() {
  if (gameLoop) clearInterval(gameLoop)
  gameLoop = setInterval(moveSnake, config.speed)
}

// 生命周期钩子
onMounted(() => {
  // 加载最高分
  const savedHighScore = localStorage.getItem('snakeHighScore')
  if (savedHighScore) {
    highScore.value = parseInt(savedHighScore)
  }
  
  // 初始化游戏
  snake.value = [
    { x: 10, y: 10 },
    { x: 9, y: 10 },
    { x: 8, y: 10 }
  ]
  generateFood()
  
  // 开始游戏循环
  startGameLoop()
  
  // 添加键盘事件监听
  window.addEventListener('keydown', handleKeyDown)
})

onUnmounted(() => {
  // 清除游戏循环
  if (gameLoop) clearInterval(gameLoop)
  
  // 移除键盘事件监听
  window.removeEventListener('keydown', handleKeyDown)
})
</script>

<template>
  <div class="game-container">
    <h1 class="game-title">贪吃蛇</h1>
    
    <div class="score-board">
      <div class="score-item">
        <span class="score-label">分数</span>
        <span class="score-value">{{ score }}</span>
      </div>
      <div class="score-item">
        <span class="score-label">最高分</span>
        <span class="score-value">{{ highScore }}</span>
      </div>
    </div>
    
    <div class="game-grid" :style="{
      width: config.gridSize * config.cellSize + 'px',
      height: config.gridSize * config.cellSize + 'px'
    }">
      <!-- 蛇身 -->
      <div 
        v-for="(cell, index) in snake" 
        :key="index" 
        class="snake-cell"
        :style="{
          left: cell.x * config.cellSize + 'px',
          top: cell.y * config.cellSize + 'px',
          backgroundColor: index === 0 ? '#4CAF50' : '#8BC34A'
        }"
      ></div>
      
      <!-- 食物 -->
      <div 
        v-if="food" 
        class="food-cell"
        :style="{
          left: food.x * config.cellSize + 'px',
          top: food.y * config.cellSize + 'px'
        }"
      ></div>
    </div>

    <!-- 手机端控制按钮 -->
    <div class="mobile-controls">
      <div class="control-row">
        <button class="mobile-control-btn" @touchstart="handleMobileControl('up')">↑</button>
      </div>
      <div class="control-row">
        <button class="mobile-control-btn" @touchstart="handleMobileControl('left')">←</button>
        <button class="mobile-control-btn" @touchstart="handleMobileControl('down')">↓</button>
        <button class="mobile-control-btn" @touchstart="handleMobileControl('right')">→</button>
      </div>
    </div>

    <!-- 开始按钮 -->
    <div v-if="!gameStarted" class="start-screen">
      <button class="start-btn" @click="startGame">开始游戏</button>
    </div>
    
    <!-- 游戏控制按钮 -->
    <div v-else class="game-controls">
      <button 
        class="control-btn" 
        @click="initGame"
      >
        重新开始
      </button>
      <button 
        class="control-btn" 
        @click="gameRunning = !gameRunning"
      >
        {{ gameRunning ? '暂停' : '继续' }}
      </button>
    </div>
    
    <!-- 游戏结束提示 -->
    <div v-if="gameOver" class="game-over-overlay">
      <div class="game-over-modal">
        <h2>游戏结束</h2>
        <p>最终得分: {{ score }}</p>
        <button class="control-btn" @click="initGame">重新开始</button>
      </div>
    </div>
    
    <div class="game-instructions">
      <p>使用方向键或WSAD控制蛇的移动</p>
      <p>空格键开始/暂停游戏</p>
    </div>
  </div>
</template>

<style scoped>
.game-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 20px;
  font-family: 'Arial', sans-serif;
}

.game-title {
  color: white;
  font-size: 2.5rem;
  margin-bottom: 20px;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
}

.score-board {
  display: flex;
  gap: 40px;
  margin-bottom: 20px;
  background: rgba(255, 255, 255, 0.2);
  padding: 10px 20px;
  border-radius: 10px;
  backdrop-filter: blur(10px);
}

.score-item {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.score-label {
  color: rgba(255, 255, 255, 0.8);
  font-size: 0.9rem;
}

.score-value {
  color: white;
  font-size: 1.5rem;
  font-weight: bold;
}

.game-grid {
  position: relative;
  background: rgba(255, 255, 255, 0.1);
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 10px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.2);
  backdrop-filter: blur(10px);
  overflow: hidden;
}

.snake-cell {
  position: absolute;
  width: 18px;
  height: 18px;
  border-radius: 4px;
  transition: all 0.1s ease;
}

.food-cell {
  position: absolute;
  width: 18px;
  height: 18px;
  background: #FF5722;
  border-radius: 50%;
  box-shadow: 0 0 10px rgba(255, 87, 34, 0.8);
  animation: pulse 1s infinite alternate;
}

@keyframes pulse {
  0% {
    transform: scale(1);
  }
  100% {
    transform: scale(1.1);
  }
}

.game-controls {
  display: flex;
  gap: 10px;
  margin-top: 20px;
}

.control-btn {
  background: rgba(255, 255, 255, 0.2);
  color: white;
  border: 1px solid rgba(255, 255, 255, 0.3);
  padding: 10px 20px;
  border-radius: 25px;
  cursor: pointer;
  font-size: 1rem;
  transition: all 0.3s ease;
  backdrop-filter: blur(10px);
}

.control-btn:hover {
  background: rgba(255, 255, 255, 0.3);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.game-over-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.game-over-modal {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 40px;
  border-radius: 20px;
  text-align: center;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.3);
  backdrop-filter: blur(10px);
}

.game-over-modal h2 {
  color: white;
  font-size: 2rem;
  margin-bottom: 20px;
}

.game-over-modal p {
  color: white;
  font-size: 1.2rem;
  margin-bottom: 30px;
}

.game-instructions {
  margin-top: 20px;
  color: rgba(255, 255, 255, 0.8);
  text-align: center;
  font-size: 0.9rem;
}

.game-instructions p {
  margin: 5px 0;
}

/* 开始屏幕 */
.start-screen {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 100;
}

.start-btn {
  background: linear-gradient(135deg, #4CAF50 0%, #45a049 100%);
  color: white;
  border: none;
  padding: 15px 30px;
  border-radius: 50px;
  font-size: 1.2rem;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(76, 175, 80, 0.4);
}

.start-btn:hover {
  transform: scale(1.1);
  box-shadow: 0 6px 20px rgba(76, 175, 80, 0.6);
}

/* 响应式设计 */
@media (max-width: 768px) {
  .game-title {
    font-size: 2rem;
  }
  
  .score-board {
    gap: 20px;
  }
  
  .game-grid {
    transform: scale(0.8);
  }
  
  .start-btn {
    padding: 12px 24px;
    font-size: 1rem;
  }
  
  .game-instructions {
    font-size: 0.8rem;
  }
}

/* 手机端控制按钮 */
.mobile-controls {
  display: none;
  margin-top: 10px;
  align-items: center;
  justify-content: center;
}

.control-row {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-bottom: 10px;
}

.mobile-control-btn {
  width: 60px;
  height: 60px;
  border-radius: 15px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  background: rgba(255, 255, 255, 0.2);
  color: white;
  font-size: 1.5rem;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease;
  backdrop-filter: blur(10px);
}

.mobile-control-btn:hover {
  background: rgba(255, 255, 255, 0.3);
  transform: scale(1.1);
}

/* 触摸设备优化 */
@media (hover: none) and (pointer: coarse) {
  .game-grid {
    transform: scale(0.9);
  }
  
  .control-btn {
    padding: 12px 24px;
    font-size: 1rem;
  }
  
  /* 显示手机端控制按钮 */
  .mobile-controls {
    display: flex;
    flex-direction: column;
  }
  
  .mobile-control-btn {
    width: 70px;
    height: 70px;
    font-size: 1.8rem;
  }
}
</style>
