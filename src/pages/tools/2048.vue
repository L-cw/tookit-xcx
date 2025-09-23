<!--
 * @Author: luchengwei
 * @Date: 2025-09-23 21:00:00
 * @LastEditTime: 2025-09-23 21:00:00
 * @Description: 
 * @LastEditors: luchengwei
-->
<template>
  <div class="wrapper">
    <header class="topbar">
      <div class="brand">
        <div class="logo">2048</div>
        <div class="subtitle">新2048</div>
      </div>
      <div class="scores">
        <div class="score">
          <div class="label">得分</div>
          <div class="value">{{ score }}</div>
        </div>
        <div class="score">
          <div class="label">最高分</div>
          <div class="value">{{ bestScore }}</div>
        </div>
      </div>
    </header>

    <div
      class="board"
      ref="boardEl"
      @touchstart.passive="onTouchStart"
      @touchmove.prevent="onTouchMove"
      @touchend.passive="onTouchEnd"
    >
      <div class="grid">
        <div v-for="n in 16" :key="n" class="grid-cell" />
      </div>

      <div class="tiles">
        <div
          v-for="(v, i) in board"
          :key="i + '-' + v + '-' + animKey"
          class="tile"
          :class="['v' + v, { empty: v === 0 }]"
        >
          <span v-if="v !== 0">{{ v }}</span>
        </div>
      </div>

      <div v-if="isGameOver" class="overlay">
        <div class="overlay-card">
          <h3>游戏结束</h3>
          <p>本局得分：{{ score }}</p>
          <button class="btn big" @click="newGame">再来一局</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, reactive, ref, watch } from 'vue'

type Dir = 'left' | 'right' | 'up' | 'down'

const SIZE = 4
const board = reactive<number[]>(Array(SIZE * SIZE).fill(0))
const score = ref(0)
const bestScore = ref<number>(parseInt(localStorage.getItem('v2048_best') || '0'))
const undoStack = reactive<{ board: number[]; score: number }[]>([])
const isGameOver = ref(false)
const animKey = ref(0)
const boardEl = ref<HTMLDivElement | null>(null)

function idx(r: number, c: number) {
  return r * SIZE + c
}

function cloneBoard(b = board) {
  return b.slice()
}

function bump() {
  animKey.value++
}

function reset() {
  for (let i = 0; i < board.length; i++) board[i] = 0
  score.value = 0
  isGameOver.value = false
  undoStack.length = 0
  spawn()
  spawn()
  bump()
}

function newGame() {
  reset()
}

function emptyCells() {
  const res: number[] = []
  board.forEach((v, i) => v === 0 && res.push(i))
  return res
}

function spawn() {
  const empties = emptyCells()
  if (empties.length === 0) return false
  const spot = empties[Math.floor(Math.random() * empties.length)]
  board[spot] = Math.random() < 0.9 ? 2 : 4
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
  const nonZero = values.filter((v) => v !== 0)
  const out: number[] = []
  let gained = 0
  for (let i = 0; i < nonZero.length; i++) {
    if (i + 1 < nonZero.length && nonZero[i] === nonZero[i + 1]) {
      const merged = nonZero[i] * 2
      out.push(merged)
      gained += merged
      i++
    } else {
      out.push(nonZero[i])
    }
  }
  while (out.length < SIZE) out.push(0)
  return { out, gained }
}

function move(dir: Dir) {
  if (isGameOver.value) return
  let moved = false
  for (let l = 0; l < SIZE; l++) {
    const ids = lineIndices(dir, l)
    const before = ids.map((i) => board[i])
    const { out, gained } = collapse(before)
    out.forEach((v, k) => (board[ids[k]] = v))
    if (!arraysEqual(before, out)) moved = true
    if (gained) score.value += gained
  }
  if (moved) {
    spawn()
    bump()
    if (!canMove()) isGameOver.value = true
  }
}

function arraysEqual(a: number[], b: number[]) {
  if (a.length !== b.length) return false
  for (let i = 0; i < a.length; i++) if (a[i] !== b[i]) return false
  return true
}

watch(score, () => {
  if (score.value > bestScore.value) {
    bestScore.value = score.value
    localStorage.setItem('v2048_best', String(bestScore.value))
  }
})

let startX = 0, startY = 0

function onTouchStart(ev: TouchEvent) {
  const t = ev.touches[0]
  startX = t.clientX
  startY = t.clientY
}
function onTouchMove(_ev: TouchEvent) {}
function onTouchEnd(ev: TouchEvent) {
  const t = ev.changedTouches[0]
  handleSwipe(t.clientX, t.clientY)
}

function handleSwipe(endX: number, endY: number) {
  const dx = endX - startX
  const dy = endY - startY
  const absX = Math.abs(dx)
  const absY = Math.abs(dy)
  const min = 30 // 提高滑动触发阈值，防止误触
  if (absX < min && absY < min) return
  const dir: Dir = absX > absY ? (dx > 0 ? 'right' : 'left') : (dy > 0 ? 'down' : 'up')
  move(dir)
}

onMounted(() => {
  reset()
})
</script>

<style scoped>
.wrapper { max-width: 420px; margin: 20px auto; padding: 0 12px; font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, 'Noto Sans', 'PingFang SC', 'Hiragino Sans GB', sans-serif; color: #776e65; }
.topbar { display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px; }
.brand { display: flex; align-items: center; gap: 10px; }
.logo { background: #edc22e; color: #fff; font-weight: 900; border-radius: 8px; padding: 6px 10px; }
.subtitle { font-size: 20px; font-weight: 700; }
.scores { display: flex; gap: 10px; }
.score { background: #bbada0; color: #fff; border-radius: 8px; padding: 6px 10px; text-align: center; min-width: 84px; }
.score .label { font-size: 12px; opacity: .9; }
.score .value { font-weight: 800; font-size: 18px; }
.board { position: relative; background: #bbada0; padding: 10px; border-radius: 10px; user-select: none; touch-action: none; }
.grid { display: grid; grid-template-columns: repeat(4, 1fr); grid-gap: 10px; }
.grid-cell { width: 78px; height: 78px; background: #cdc1b4; border-radius: 6px; }
.tiles { position: absolute; inset: 10px; display: grid; grid-template-columns: repeat(4, 1fr); grid-gap: 10px; pointer-events: none; }
.tile { width: 78px; height: 78px; border-radius: 6px; display: flex; align-items: center; justify-content: center; font-weight: 900; font-size: 24px; transition: transform .08s ease, background-color .15s ease; }
.tile.empty { background: transparent; }
.overlay { position: absolute; inset: 0; background: rgba(0,0,0,.4); display: grid; place-items: center; border-radius: 10px; }
.overlay-card { background: #fff; color: #333; padding: 16px 20px; border-radius: 10px; width: 70%; text-align: center; }
.btn.big { background: #8f7a66; color: #fff; border: none; border-radius: 8px; padding: 10px 16px; font-size: 16px; cursor: pointer; }
.tile.v2 { background: #eee4da; color: #776e65; }
.tile.v4 { background: #ede0c8; color: #776e65; }
.tile.v8 { background: #f2b179; color: #f9f6f2; }
.tile.v16 { background: #f59563; color: #f9f6f2; }
.tile.v32 { background: #f67c5f; color: #f9f6f2; }
.tile.v64 { background: #f65e3b; color: #f9f6f2; }
.tile.v128 { background: #edcf72; color: #f9f6f2; font-size: 22px; }
.tile.v256 { background: #edcc61; color: #f9f6f2; font-size: 22px; }
.tile.v512 { background: #edc850; color: #f9f6f2; font-size: 22px; }
.tile.v1024 { background: #edc53f; color: #f9f6f2; font-size: 18px; }
.tile.v2048 { background: #edc22e; color: #f9f6f2; font-size: 18px; }
.tile[class*='v4096'], .tile[class*='v8192'] { background: #3c3a32; color: #f9f6f2; font-size: 18px; }
</style>