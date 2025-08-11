<script setup lang="ts">
	import { ref } from 'vue'
	
	const showTips = ref(false)
	const appId = ref('')
	const pathStr = ref('')
	
	function handleShowTips() {
		showTips.value = !showTips.value
	}
	
	function handleSearch() {
		uni.navigateToMiniProgram({
			appId: appId.value,
			path: pathStr.value
		})
	}
</script>

<template>
	<view class="page-wrapper search-miniapp-name-page">
		<view class="ipt-box">
			<input class="uni-input" placeholder="请输入 AppId" v-model="appId" />
		</view>
		<view class="ipt-box">
			<input class="uni-input" placeholder="请输入 path（选填）" v-model="pathStr" />
		</view>
		<button type="primary" class="search-btn" @click="handleSearch">查询</button>
		<view class="tips-box">
			<view class="get-method" @click="handleShowTips">
				怎样获取 AppId ？点击{{ showTips ? '收起' : '展开' }}
			</view>
			<view v-show="showTips" class="faq-box">
				<view class="faq-item">1. 点击小程序右上角的三个点</view>
				<view class="faq-item">2. 在弹出的窗口中点击小程序的名字，查看小程序介绍</view>
				<view class="faq-item">3. 点击更多资料按钮</view>
				<view class="faq-item">4. 在页面中就能看到我们要找的 AppId 啦</view>
			</view>
			<view class="step-box">
				<view class="step-item">1. 输入 AppId ，点击查询</view>
				<view class="step-item">2. 弹窗中显示的就是查询的小程序的名称</view>
				<view class="step-item">3. 点击允许可以跳转至该小程序</view>
			</view>
		</view>
	</view>
</template>

<style scoped lang="less">
.search-miniapp-name-page {
	padding: 20rpx 32rpx;
	.ipt-box {
		padding: 20rpx 32rpx;
		margin-bottom: 24rpx;
		border-radius: 5px;
		background-color: #fff;
	}
	.search-btn {
		margin-bottom: 24rpx;
	}
	.tips-box {
		padding: 20rpx 32rpx;
		font-size: 28rpx;
		border-radius: 5px;
		background-color: #fff;
		.get-method {
			height: 30rpx;
			line-height: 30rpx;
			margin-bottom: 24rpx;
			color: #42b983;
		}
		.faq-box {
			border: 1rpx solid #eee;
			padding: 20rpx;
			margin-bottom: 24rpx;
			border-radius: 5px;
			color: #999;
		}
		.step-box {
			color: #666;
		}
	}
}    
</style>
