<template>
  <view class="wrapper" @touchstart="onTouchStart" @touchmove="onTouchMove" @touchend="onTouchEnd">
    <view class="topbar">
      <view class="brand">
        <view class="logo">2048</view>
        <view class="subtitle">益智解压</view>
      </view>
      <view class="scores">
        <view class="score">
          <view class="label">得分</view>
          <view class="value">{{ score }}</view>
        </view>
        <view class="score">
          <view class="label">最高分</view>
          <view class="value">{{ bestScore }}</view>
        </view>
      </view>
    </view>

    <view class="toolbar">
      <button class="btn btn-primary" @click="newGame">重置</button>
      <button class="btn btn-primary" :class="{ disabled: board.filter(v => v > 0).length <= 2 }" @click="startRemove">任意消除</button>
      <button class="btn btn-primary" @click="startReplace">任意替换</button>
    </view>

    <view class="board">
      <view class="grid">
        <view v-for="n in 16" :key="n" class="grid-cell" />
      </view>

      <view class="tiles">
        <view
          v-for="(v, i) in board"
          :key="i + '-' + v + '-' + animKey"
          class="tile"
          :class="['v' + v, { 
            empty: v === 0, 
            pop: poppedIndex===i || mergedIndices.includes(i),
            selected: mode === 'replace' && selectedIndex === i
          }]"
          @click="onTileClick(i)"
        >
          <text v-if="v !== 0">{{ v }}</text>
        </view>
      </view>

      <view v-if="isGameOver" class="mask">
        <view class="game-over-modal">
          <view class="modal-header">
            <view class="modal-title">游戏结束</view>
            <view class="modal-subtitle">本局得分</view>
            <view class="score-display">{{ score }}</view>
          </view>
          <view class="modal-actions">
            <button class="btn btn-outline" @click.stop="isGameOver=false">留在当前</button>
            <button class="btn btn-primary" @click.stop="newGame">再来一局</button>
          </view>
        </view>
      </view>

    </view>
    <view v-if="mode==='remove'" class="mask mask-remove" @click="mode = 'none'">
      <view class="hint-text">点击任意数字块进行消除</view>
    </view>

    <!-- 任意替换蒙层 + 选择数字按钮 -->
    <view v-else-if="mode==='replace'" class="mask" @click="mode = 'none'">
      <view class="replace-panel" @click.stop>
        <view v-for="val in [2,4,8,16,32]" 
              :key="val" 
              :class="['mini-tile', 'v' + val]" 
              @click.stop="confirmReplace(val)">
          {{val}}
        </view>
      </view>
    </view>
  </view>
</template>

<script setup lang="ts">
import { onMounted, onUnmounted, reactive, ref, watch } from 'vue'

type Dir = 'left' | 'right' | 'up' | 'down'

const SIZE = 4
const board = reactive<number[]>(Array(SIZE * SIZE).fill(0))
const score = ref(0)
const bestScore = ref<number>(uni.getStorageSync('v2048_best') || 0)
const isGameOver = ref(false)
const animKey = ref(0)
const poppedIndex = ref<number|null>(null)
const mergedIndices = ref<number[]>([])

const mode = ref<'none'|'remove'|'replace'>('none')
const selectedIndex = ref<number|null>(null)

// 生成游戏结束的测试数据
function mockGameOverState() {
  // 清空当前棋盘
  reset()
  
  // 创建一个无法继续移动的棋盘状态
  const mockBoard = [
    2, 4, 2, 4,
    4, 2, 4, 2,
    2, 4, 2, 4,
    4, 2, 4, 2
  ]
  
  // 更新棋盘
  for (let i = 0; i < board.length; i++) {
    board[i] = mockBoard[i]
  }
  
  // 设置一个高分
  score.value = 1000
  
  // 强制触发游戏结束检查
  if (!canMove()) {
    isGameOver.value = true
  }
}

function idx(r: number, c: number) {
  return r * SIZE + c
}

function bump() {
  animKey.value++
}

function reset() {
  for (let i = 0; i < board.length; i++) board[i] = 0
  score.value = 0
  isGameOver.value = false
  spawn()
  spawn()
  bump()
}

const newGame = () => {
  // Clear saved game state when starting a new game
  uni.removeStorageSync('v2048_game_state')
  reset()
}

function emptyCells() {
  return board.map((v, i) => v===0?i:-1).filter(i=>i>=0)
}

function spawn() {
  const empties = emptyCells()
  if (!empties.length) return false
  const spot = empties[Math.floor(Math.random() * empties.length)]
  board[spot] = Math.random() < 0.9 ? 2 : 4
  poppedIndex.value = spot
  setTimeout(()=> poppedIndex.value=null,200)
  return true
}

function canMove() {
  if (emptyCells().length) return true
  for (let r = 0; r < SIZE; r++) {
    for (let c = 0; c < SIZE; c++) {
      const v = board[idx(r, c)]
      if (r + 1 < SIZE && board[idx(r + 1, c)] === v) return true
      if (c + 1 < SIZE && board[idx(r, c + 1)] === v) return true
    }
  }
  return false
}

function lineIndices(dir: Dir, line: number): number[] {
  const ids: number[] = []
  if (dir === 'left') {
    for (let c = 0; c < SIZE; c++) ids.push(idx(line, c))
  } else if (dir === 'right') {
    for (let c = SIZE - 1; c >= 0; c--) ids.push(idx(line, c))
  } else if (dir === 'up') {
    for (let r = 0; r < SIZE; r++) ids.push(idx(r, line))
  } else if (dir === 'down') {
    for (let r = SIZE - 1; r >= 0; r--) ids.push(idx(r, line))
  }
  return ids
}

function collapse(values: number[]) {
  const nonZero = values.filter(v => v !== 0)
  const out: number[] = []
  let gained = 0
  const mergedAt: number[] = [] // indices in `out` where a merge resulted
  
  for (let i = 0; i < nonZero.length; i++) {
    if (i + 1 < nonZero.length && nonZero[i] === nonZero[i + 1]) {
      const merged = nonZero[i] * 2
      out.push(merged)
      gained += merged
      mergedAt.push(out.length - 1)
      i++
    } else {
      out.push(nonZero[i])
    }
  }
  while (out.length < SIZE) out.push(0)
  return { out, gained, mergedAt }
}

function move(dir: Dir) {
  if (isGameOver.value) return
  
  let moved = false
  const mergedGlobal: number[] = []
  
  for (let l = 0; l < SIZE; l++) {
    const ids = lineIndices(dir, l)
    const before = ids.map(i => board[i])
    const { out, gained, mergedAt } = collapse(before)
    
    out.forEach((v, k) => board[ids[k]] = v)
    if (!arraysEqual(before, out)) moved = true
    if (gained) score.value += gained
    
    // Record merged indices on the board
    mergedAt.forEach(k => {
      mergedGlobal.push(ids[k])
    })
  }
  
  if (moved) {
    spawn()
    bump()
    
    // Trigger pop animation for merged tiles
    mergedIndices.value = mergedGlobal
    if (mergedGlobal.length) {
      setTimeout(() => { 
        mergedIndices.value = [] 
      }, 220)
    }
    
    if (!canMove()) isGameOver.value = true
  }
}

function arraysEqual(a: number[], b: number[]): boolean {
  if (a.length !== b.length) return false
  
  for (let i = 0; i < a.length; i++) {
    if (a[i] !== b[i]) return false
  }
  
  return true
}

watch(score,()=>{
  // Only update best score if the game is not over or if it's a new best score from a completed game
  if(score.value > bestScore.value && !isGameOver.value) {
    bestScore.value = score.value
    uni.setStorageSync('v2048_best', bestScore.value)
  }
})

// 手势
let startX=0,startY=0
function onTouchStart(e:TouchEvent){
  const t=e.touches[0]
  startX=t.clientX;startY=t.clientY
}
function onTouchMove(_e:TouchEvent){}
function onTouchEnd(e:TouchEvent){
  const t=e.changedTouches[0]
  handleSwipe(t.clientX,t.clientY)
}
function handleSwipe(endX:number,endY:number){
  if(mode.value!=='none') return
  const dx=endX-startX, dy=endY-startY
  const absX=Math.abs(dx), absY=Math.abs(dy)
  const min=30
  if(absX<min && absY<min) return
  const dir:Dir=absX>absY?(dx>0?'right':'left'):(dy>0?'down':'up')
  move(dir)
}

// 任意消除/替换逻辑
function startRemove() {
  if(board.filter(v => v > 0).length <= 2) return
  mode.value = 'remove' 
}

function startReplace() { 
  mode.value = 'replace'
  selectedIndex.value = null
}

const onTileClick = (i: number) => {
  if (mode.value === 'remove' && board[i] > 0) {
    board[i] = 0
    mode.value = 'none'
  } else if (mode.value === 'replace') {
    // Only allow selecting non-empty tiles for replacement
    if (board[i] > 0) {
      selectedIndex.value = i
    }
  } 
}

const confirmReplace = (val: number) => {
  if (selectedIndex.value !== null) {
    board[selectedIndex.value] = val
  }
  mode.value = 'none'
  selectedIndex.value = null
}

onMounted(()=>{
  // Try to load saved game state
  const savedState = uni.getStorageSync('v2048_game_state')
  if (savedState) {
    const { savedBoard, savedScore } = JSON.parse(savedState)
    board.splice(0, board.length, ...savedBoard)
    score.value = savedScore
  } else {
    reset()
  }
})

onUnmounted(() => {
  // Save current game state when component is unmounted
  if (!isGameOver.value) {  // Only save if game is not over
    const gameState = {
      savedBoard: [...board],
      savedScore: score.value
    }
    uni.setStorageSync('v2048_game_state', JSON.stringify(gameState))
  }
})
</script>

<style scoped lang="less">
// variables
@font: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
@board-bg: #bbada0;
@cell-bg: #cdc1b4;
@btn-bg: #8f7a66;
@brand-bg: #edc22e;

.wrapper {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  font-family: @font;

  .topbar {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin: 10px;

    .brand {
      .logo {
        background: @brand-bg;
        color: #fff;
        font-weight: 900;
        border-radius: 8px;
        padding: 6px 10px;
      }
      .subtitle {
        font-size: 20px;
        font-weight: 700;
      }
    }

    .scores {
      display: flex;
      gap: 10px;

      .score {
        background: @board-bg;
        color: #fff;
        border-radius: 8px;
        padding: 6px 10px;
        text-align: center;
        min-width: 70px;

        .value {
          font-size: 18px;
          font-weight: 800;
        }
      }
    }
  }

  .toolbar {
    display: flex;
    margin: 10px 0;
    padding: 0 10px;
  }

  .board {
    position: relative;
    background: @board-bg;
    padding: 10px;
    border-radius: 10px;
    z-index: 101;
    margin: 0 10px;

    .grid {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      grid-gap: 10px;

      .grid-cell {
        width: 78px;
        height: 78px;
        background: @cell-bg;
        border-radius: 6px;
      }
    }

    .tiles {
      position: absolute;
      inset: 10px;
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      grid-gap: 10px;
      z-index: 20; /* 确保tiles在mask上方 */

      .tile {
        width: 78px;
        height: 78px;
        border-radius: 6px;
        display: flex;
        align-items: center;
        justify-content: center;
        font-weight: 900;
        font-size: 28px;
        transition: all .15s ease-out;
        transform: scale(1);
        
        &.pop {
          animation: popIn .15s ease-out forwards;
        }
        &.empty {
          background: transparent;
        }
      }
    }

    .overlay {
      position: absolute;
      inset: 0;
      background: rgba(0, 0, 0, .4);
      display: flex;
      justify-content: center;
      align-items: center;
      border-radius: 10px;

      &-card {
        background: #fff;
        color: #333;
        padding: 16px 20px;
        border-radius: 10px;
        width: 70%;
        text-align: center;
      }
    }

  }

  
}

.mini-tile {
  width: 48px;
  height: 48px;
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 900;
  font-size: 20px;
  color: #fff;
  cursor: pointer;
  transition: transform 0.1s;

  &:not(:last-child){
    margin-right: 10px;
  }
  
  &:active {
    transform: scale(0.95);
  }
}

.hint-text {
  color: white;
  font-size: 18px;
  font-weight: bold;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.5);
  padding: 15px 25px;
  background: rgba(0, 0, 0, 0.5);
  border-radius: 20px;
  pointer-events: none;
}

// number colors
.tile,
.mini-tile {
  &.v2 { background: #eee4da; color: #776e65; }
  &.v4 { background: #ede0c8; color: #776e65; }
  &.v8 { background: #f2b179; color: #f9f6f2; }
  &.v16 { background: #f59563; color: #f9f6f2; }
  &.v32 { background: #f67c5f; color: #f9f6f2; }
}

.tile {
  transition: all 0.1s ease;
  
  &.selected {
    box-shadow: 0 0 0 4px rgba(255, 255, 0, 0.8) inset;
    transform: scale(0.95);
    z-index: 2;
    position: relative;
  }

  &.v64 { background: #f65e3b; color: #f9f6f2; }
  &.v128 { background: #edcf72; color: #f9f6f2; }
  &.v256 { background: #edcc61; color: #f9f6f2; }
  &.v512 { background: #edc850; color: #f9f6f2; }
  &.v1024 { background: #edc53f; color: #f9f6f2; }
  &.v2048 { background: #edc22e; color: #f9f6f2; }
}

// animations should be at root to avoid scoping issues
@keyframes popIn {
  0% { transform: scale(0); }
  100% { transform: scale(1); }
}

@keyframes pop {
  0% { transform: scale(1); }
  50% { transform: scale(1.15); }
  100% { transform: scale(1); }
}

/* 蒙层 - 全屏 */
.mask {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  backdrop-filter: blur(4px);
  z-index: 100;
  display: flex;
  justify-content: center;
  align-items: center;
  touch-action: none;
  animation: fadeIn 0.2s ease-out;
  &.mask-remove {
    padding-top: 30px;
    align-items: flex-start;
  }

  .game-over-modal {
    background: #fff;
    border-radius: 16px;
    width: 80%;
    max-width: 300px;
    overflow: hidden;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);
    transform: translateY(0);
    animation: slideUp 0.3s ease-out;
    
    .modal-header {
      padding: 24px 20px;
      text-align: center;
      background: linear-gradient(135deg, #ff6b6b, #ff8e8e);
      color: white;
    }
    
    .modal-title {
      font-size: 24px;
      font-weight: bold;
      margin-bottom: 8px;
    }
    
    .modal-subtitle {
      font-size: 14px;
      opacity: 0.9;
      margin-bottom: 4px;
    }
    
    .score-display {
      font-size: 42px;
      font-weight: bold;
      margin: 10px 0 0;
      text-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    }
    
    .modal-actions {
      display: flex;
      padding: 20px;
      background: #f8f8f8;
      justify-content: space-between;
      gap: 12px;
    }
  }
  
  .replace-panel {
    position: absolute;
    display: flex;
    bottom: 50px;
    z-index: 101;
    background: rgba(255, 255, 255, 0.9);
    padding: 10px;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
    pointer-events: auto;
  }
  
  /* 添加点击蒙层关闭功能 */
  &::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    z-index: -1;
  }
}
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slideUp {
  from { 
    opacity: 0;
    transform: translateY(20px);
  }
  to { 
    opacity: 1;
    transform: translateY(0);
  }
}

.btn {
  position: relative;
  overflow: hidden;
  border: none;
  border-radius: 8px;
  margin: 0;
  padding: 4px;
  font-size: 15px;
  font-weight: 500;
  transition: all 0.2s ease;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-width: 100px;
  
  &-primary {
    background: @btn-bg;
    color: white;
    
    &:active {
      transform: translateY(1px);
    }
  }
  
  &-outline {
    background: white;
    color: #666;
    border: 1px solid #ddd;
    
    &:active {
      background: #f5f5f5;
      transform: translateY(1px);
    }
  }
  
  .btn-icon {
    margin-right: 6px;
    font-size: 16px;
  }

  &:not(:last-child){
    margin-right: 10px;
  }

  &.disabled {
    opacity: 0.6;
    &::after {
      content: '';
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      background: rgba(255, 255, 255, 0.3);
      z-index: 1;
    }
    &:active {
      transform: none;
    }
  }

  &.big {
    padding: 10px 16px;
    font-size: 16px;
  }
}
</style>
