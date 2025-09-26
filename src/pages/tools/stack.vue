<template>
  <view class="stack-game">
    <!-- 分数显示 -->
    <view class="score">得分：{{ score }}</view>
    <!-- 游戏画布 -->
    <canvas
      type="2d"
      canvas-id="stackCanvas"
      class="game-canvas"
      @click="handleClick"
    ></canvas>
    <!-- 结束提示 -->
    <view v-if="gameOver" class="overlay">
      <text>游戏结束</text>
      <button @click="restart">重新开始</button>
    </view>
  </view>
</template>

<script setup lang="ts">
import { ref, onMounted, nextTick, getCurrentInstance } from 'vue'

const canvasId = 'stackCanvas'
const ctx = ref<UniApp.CanvasContext>()
const screenInfo = uni.getSystemInfoSync()
const screenWidth = screenInfo.windowWidth
const screenHeight = Math.floor(screenInfo.windowHeight * 0.8) // 80% of screen height
const blockHeight = 30 // 方块高度
const baseSpeed = 1 // 基础速度
const themeColors = ['#ff595e', '#ffca3a', '#8ac926', '#1982c4', '#6a4c93']
const viewportOffset = ref(0) // 视口偏移量，用于实现滚动效果

// 游戏状态
const tower = ref<any[]>([])
const currentBlock = ref<any>(null)
const score = ref(0)
const gameOver = ref(false)
let animationFrame: number

// 计算新方块的宽度
function calculateNewWidth() {
  if (tower.value.length <= 1) return screenWidth * 0.8
  
  // 获取最近两个方块的重叠部分
  const last = tower.value[tower.value.length - 1]
  const secondLast = tower.value[tower.value.length - 2]
  
  // 计算重叠部分的宽度
  const overlapStart = Math.max(last.x, secondLast.x)
  const overlapEnd = Math.min(last.x + last.width, secondLast.x + secondLast.width)
  const overlapWidth = Math.max(overlapEnd - overlapStart, 0)
  
  // 新方块的宽度基于重叠部分，但不要太窄
  return Math.max(overlapWidth * 0.9, screenWidth * 0.2)
}

// 初始化游戏
function initGame() {
  score.value = 0
  gameOver.value = false
  viewportOffset.value = 0
  
  // 初始方块
  const initialBlockWidth = screenWidth * 0.8
  const initialX = (screenWidth - initialBlockWidth) / 2
  
  tower.value = [
    {
      x: initialX,
      y: screenHeight - blockHeight,
      width: initialBlockWidth,
    },
  ]
  
  // 确保当前方块被正确初始化
  spawnBlock()
  
  // 如果已经有动画帧，先取消
  if (animationFrame) {
    cancelAnimationFrame(animationFrame)
  }
  
  console.log('Game initialized with screen size:', screenWidth, screenHeight)
  console.log('Initial block position:', tower.value[0])
  
  // 开始游戏循环
  draw()
}

// 生成新方块
async function spawnBlock() {
  const last = tower.value[tower.value.length - 1]
  
  // 计算新方块的宽度
  const newWidth = calculateNewWidth()
  
  // 新方块的初始位置（从左侧进入）
  currentBlock.value = {
    x: 0,
    y: last.y - blockHeight,
    width: newWidth,
    speed: baseSpeed + (tower.value.length * 0.1),
    direction: 1,
  }
  
  // 检查是否需要滚动视口
  checkViewportScroll()
  console.log('New block spawned at:', currentBlock.value)
}

// 添加下落动画
async function animateBlockFall(block: any, targetY: number) {
  return new Promise<void>((resolve) => {
    const startY = block.y
    const distance = targetY - startY
    const duration = 150 // 缩短动画时间
    const startTime = performance.now()
    
    function step(currentTime: number) {
      const elapsed = currentTime - startTime
      const progress = Math.min(elapsed / duration, 1)
      
      // 使用线性缓动，更自然
      block.y = startY + distance * progress
      
      if (progress < 1) {
        requestAnimationFrame(step)
      } else {
        block.y = targetY // 确保最终位置准确
        resolve()
      }
    }
    
    requestAnimationFrame(step)
  })
}

// 点击下落
async function handleClick() {
  if (gameOver.value || !currentBlock.value) return
  
  const last = tower.value[tower.value.length - 1]
  if (!last) {
    console.error('No last block found')
    return
  }
  
  // 确保当前方块不会超出画布
  currentBlock.value.x = Math.max(0, Math.min(currentBlock.value.x, screenWidth - currentBlock.value.width))
  
  // 添加下落动画
  const targetY = last.y - blockHeight
  await animateBlockFall(currentBlock.value, targetY)
  
  const overlapStart = Math.max(currentBlock.value.x, last.x)
  const overlapEnd = Math.min(
    currentBlock.value.x + currentBlock.value.width,
    last.x + last.width
  )
  const overlapWidth = overlapEnd - overlapStart

  if (overlapWidth <= 0) {
    // 游戏结束
    gameOver.value = true
    if (animationFrame) {
      cancelAnimationFrame(animationFrame)
    }
    return
  }

  // 保留重叠部分
  tower.value.push({
    x: overlapStart,
    y: currentBlock.value.y,
    width: overlapWidth,
  })

  score.value++
  
  // 立即更新视图
  if (ctx.value) {
    ctx.value.draw(true)
  }
  
  // 短暂延迟后生成新方块
  await new Promise(resolve => setTimeout(resolve, 50))
  await spawnBlock()
  checkViewportScroll()
}

// 平滑滚动视口到目标位置
let targetOffset = 0
let currentOffset = 0
const scrollSpeed = 0.1

// 检查并更新视口滚动
function checkViewportScroll() {
  if (tower.value.length === 0) return
  
  const last = tower.value[tower.value.length - 1]
  // 目标视口顶部位置（让新方块显示在屏幕上半部分）
  targetOffset = Math.max(0, last.y - screenHeight * 0.7)
  
  // 平滑滚动到目标位置
  if (Math.abs(currentOffset - targetOffset) > 0.5) {
    currentOffset += (targetOffset - currentOffset) * scrollSpeed
    viewportOffset.value = Math.floor(currentOffset)
    
    // 确保继续更新直到到达目标
    requestAnimationFrame(() => {
      if (ctx.value) ctx.value.draw()
    })
  }
}

// 转换坐标到视口坐标
function toViewportY(y: number) {
  return y - viewportOffset.value
}

// 绘制循环
function draw() {
  if (!ctx.value) {
    console.log('Canvas context not ready')
    return
  }
  
  // 游戏结束处理
  if (gameOver.value) {
    // 首先绘制当前游戏状态
    if (ctx.value) {
      // 清除画布
      ctx.value.clearRect(0, 0, screenWidth, screenHeight)
      
      // 绘制背景
      ctx.value.setFillStyle('#f5f5f5')
      ctx.value.fillRect(0, 0, screenWidth, screenHeight)
      
      // 计算视口偏移，使塔在屏幕中居中显示
      if (tower.value.length > 0) {
        const totalHeight = Math.abs(tower.value[0].y - tower.value[tower.value.length - 1].y) + blockHeight
        const targetY = Math.max(0, (screenHeight - totalHeight) / 2)
        viewportOffset.value = Math.max(0, -targetY)
      }
      
      // 绘制所有方块
      tower.value.forEach((block, i) => {
        if (!block) return
        const color = themeColors[i % themeColors.length]
        ctx.value.setFillStyle(color)
        const padding = 0
        
        // 计算方块在视口中的位置
        const blockY = toViewportY(block.y)
        
        // 只绘制视口内的方块
        if (blockY + blockHeight >= 0 && blockY <= screenHeight) {
          ctx.value.fillRect(
            Math.floor(block.x) + padding,
            Math.floor(blockY) + padding,
            Math.ceil(block.width) - padding * 2,
            blockHeight - padding * 2
          )
        }
      })
      
      // 绘制半透明遮罩
      ctx.value.setFillStyle('rgba(0, 0, 0, 0.6)')
      ctx.value.fillRect(0, 0, screenWidth, screenHeight)
      
      // 绘制游戏结束面板
      const panelWidth = Math.min(screenWidth * 0.8, 400)
      const panelHeight = 200
      const panelX = (screenWidth - panelWidth) / 2
      const panelY = (screenHeight - panelHeight) / 2
      
      // 绘制面板背景
      ctx.value.setFillStyle('rgba(255, 255, 255, 0.9)')
      ctx.value.fillRect(panelX, panelY, panelWidth, panelHeight)
      
      // 绘制边框
      ctx.value.setStrokeStyle('#333')
      ctx.value.setLineWidth(2)
      ctx.value.strokeRect(panelX, panelY, panelWidth, panelHeight)
      
      // 设置文字样式
      ctx.value.setFillStyle('#333')
      ctx.value.setTextBaseline('middle')
      
      // 游戏结束标题
      ctx.value.setFontSize(28)
      const title = '游戏结束'
      const titleWidth = ctx.value.measureText(title).width
      ctx.value.fillText(title, (screenWidth - titleWidth) / 2, panelY + 50)
      
      // 分数
      ctx.value.setFontSize(24)
      const scoreText = `得分: ${score.value}`
      const scoreWidth = ctx.value.measureText(scoreText).width
      ctx.value.fillText(scoreText, (screenWidth - scoreWidth) / 2, panelY + 100)
      
      // 重新开始提示
      ctx.value.setFontSize(18)
      const restartText = '点击重新开始'
      const restartWidth = ctx.value.measureText(restartText).width
      ctx.value.fillText(restartText, (screenWidth - restartWidth) / 2, panelY + 150)
      
      ctx.value.draw(true)
    }
    return
  }
  
  // 清除画布
  ctx.value.clearRect(0, 0, screenWidth, screenHeight)
  
  // 绘制背景
  ctx.value.setFillStyle('#f5f5f5')
  ctx.value.fillRect(0, 0, screenWidth, screenHeight)

  // 绘制 tower
  tower.value.forEach((block, i) => {
    if (!block) return
    
    // 只绘制在视口内的方块
    const blockViewportY = toViewportY(block.y)
    if (blockViewportY + blockHeight < 0 || blockViewportY > screenHeight) return
    
    const color = themeColors[i % themeColors.length]
    ctx.value.setFillStyle(color)
    
    // 绘制方块主体，稍微放大一点以消除白边
    const padding = 0 // 微小的内边距来消除白边
    ctx.value.fillRect(
      Math.floor(block.x) + padding,
      Math.floor(blockViewportY) + padding,
      Math.ceil(block.width) - padding * 2,
      blockHeight - padding * 2
    )
  })

  // 绘制当前方块
  if (currentBlock.value) {
    currentBlock.value.x +=
      currentBlock.value.speed * currentBlock.value.direction
    if (
      currentBlock.value.x < 0 ||
      currentBlock.value.x + currentBlock.value.width > screenWidth
    ) {
      currentBlock.value.direction *= -1
    }
    
    const currentY = toViewportY(currentBlock.value.y)
    if (currentY + blockHeight >= 0 && currentY <= screenHeight) {
      // 使用下一个方块应该有的颜色
      const colorIndex = tower.value.length % themeColors.length
      ctx.value.setFillStyle(themeColors[colorIndex])
      
      // 添加内边距以保持与其他方块一致的样式
      const padding = 0
      ctx.value.fillRect(
        Math.floor(currentBlock.value.x) + padding,
        Math.floor(currentY) + padding,
        Math.ceil(currentBlock.value.width) - padding * 2,
        blockHeight - padding * 2
      )
    }
  }

  // 提交绘制
  ctx.value.draw()
  
  // 继续动画循环
  animationFrame = requestAnimationFrame(draw)
}

// 重新开始
function restart() {
  initGame()
}

// 获取组件实例
const instance = getCurrentInstance()

onMounted(() => {
  console.log('Component mounted, initializing canvas...')
  
  // 使用 nextTick 确保 DOM 已更新
  nextTick(() => {
    console.log('DOM updated, creating canvas context...')
    
    // 创建 canvas 上下文
    ctx.value = uni.createCanvasContext(canvasId, instance)
    
    // 设置 canvas 尺寸
    const query = uni.createSelectorQuery().in(instance)
    query.select(`#${canvasId}`)
      .boundingClientRect()
      .exec(res => {
        if (res && res[0]) {
          const { width, height } = res[0]
          console.log('Canvas dimensions:', width, height)
          // 初始化游戏
          initGame()
        } else {
          console.error('Failed to get canvas dimensions', res)
          // 如果获取尺寸失败，使用默认值继续
          console.log('Using default dimensions')
          initGame()
        }
      })
  })
})
</script>

<style scoped>
.stack-game {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.score {
  margin: 10px;
  font-size: 18px;
}
.game-canvas {
  width: 100%;
  height: 80vh;
  background: #f5f5f5;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}
.overlay {
  position: absolute;
  top: 200rpx;
  text-align: center;
}
</style>