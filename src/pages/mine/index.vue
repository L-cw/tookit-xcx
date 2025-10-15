<template>
  <view class="my-page">
    <!-- 顶部：用户区域（无需登录） -->
    <view class="user-section">
      <view class="avatar">{{ initials }}</view>
      <view class="user-text">
        <text class="title">无需登录即可使用</text>
        <text class="subtitle">当前数据仅存在本地，清空缓存后无法找回</text>
      </view>
    </view>

    <!-- 功能区：扁平风列表 -->
    <view class="list">

			<!-- 应用介绍 -->
			<view class="cell" @click="showIntro">
        <text class="cell-left">应用介绍</text>
        <text class="cell-right">查看</text>
      </view>

      <!-- 意见反馈（客服） -->
      <button class="cell reset-btn" open-type="contact">
        <text class="cell-left">意见反馈</text>
        <text class="cell-right">联系客服</text>
      </button>

      <!-- 常见问题 -->
      <view class="cell" @click="showFAQ">
        <text class="cell-left">常见问题</text>
        <text class="cell-right">FAQ</text>
      </view>

      <!-- 清除缓存 -->
      <!-- <view class="cell" @click="clearCache">
        <text class="cell-left">清除缓存</text>
        <text class="cell-right">一键清理</text>
      </view> -->

      <!-- 关于本应用 -->
      <view class="cell">
        <text class="cell-left">关于本应用</text>
        <text class="cell-right">{{ version }}</text>
      </view>
    </view>

  </view>
</template>

<script setup lang="ts">
import { ref } from 'vue'

/** 版本可从配置处注入，这里先写死 */
const version = ref('Beta')
/** 简洁头像：用首字母占位（也可换成本地图片） */
const initials = ref('U')

const showIntro = () => {
  uni.showModal({
    title: '应用介绍',
    content:
      '这是一款简洁的小程序，无需登录即可使用核心功能。未来可按需开启登录以同步设置、收藏、历史等。',
    showCancel: false
  })
}

const showFAQ = () => {
  uni.showModal({
    title: '常见问题',
    content:
`Q1：需要登录吗
A1：目前不需要，后续按需开放。

Q2：数据会丢失吗？
A2：本地数据保存在缓存中，手动清理或重装小程序可能会清空。`,
    showCancel: false
  })
}

const clearCache = () => {
  try {
    uni.clearStorageSync()
    uni.showToast({ title: '缓存已清除', icon: 'success' })
  } catch (e) {
    uni.showToast({ title: '清除失败', icon: 'none' })
  }
}
</script>

<style scoped>
/* 页面骨架 */
.my-page {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  background: #f6f7fb;
  color: #111;
}

/* 用户区域（扁平、留白） */
.user-section {
  display: flex;
  align-items: center;
  padding: 20px 16px;
  background: #ffffff;
}
.avatar {
  width: 56px;
  height: 56px;
  border-radius: 12px;
  background: #eceff5;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  font-size: 20px;
  color: #555;
  margin-right: 12px;
}
.user-text .title {
  display: block;
  font-size: 18px;
  font-weight: 600;
}
.user-text .subtitle {
  display: block;
  font-size: 13px;
  color: #888;
  margin-top: 4px;
}

/* 列表（卡片式容器，扁平分隔） */
.list {
  margin: 12px 12px 0;
  background: #ffffff;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 1px 2px rgba(0,0,0,0.04);
}

/* 单元行：左右排布，轻分隔线 */
.cell {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 14px 16px;
  font-size: 16px;
  line-height: 24px;
  background: #fff;
}
.cell + .cell {
  border-top: 1px solid #f1f2f6;
}
.cell-left { color: #222; }
.cell-right { color: #9aa0a6; font-size: 14px; }

/* 去除小程序 button 默认样式，保持扁平一致 */
.reset-btn {
  text-align: left;
  padding: 0;
  border: none;
  background: transparent;
  border-radius: 0;
}
.reset-btn .cell-left,
.reset-btn .cell-right {
  pointer-events: none; /* 保证整行点击仍触发按钮 */
}
button.reset-btn {
  /* 覆盖小程序button默认内边距与背景 */
  margin: 0;
  padding: 14px 16px;
  background: #fff;
  color: inherit;
  font-size: 16px;
  line-height: 24px;
}
button.reset-btn + .cell,
button.reset-btn + button.reset-btn {
  border-top: 1px solid #f1f2f6;
}
/* 去除 button ::after 的默认边框（微信小程序） */
button.reset-btn::after {
  border: none;
}

/* 底部版权 */
.footer {
  margin-top: auto;
  text-align: center;
  padding: 16px 0 24px;
  font-size: 12px;
  color: #9aa0a6;
}
</style>