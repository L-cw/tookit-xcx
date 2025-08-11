<template>
	<view class="page-wrapper mine-page">
		<view class="banner">
			<button open-type="contact" class="gradient-bg banner-btn">联系作者</button>
			<button open-type="contact" class="gradient-bg banner-btn">意见反馈</button>
		</view>
		<view class="banner-text">触摸想象，为思维插上翅膀</view>
		<image class="mine-bg" src="../../static/mine/mine-bg.png"></image>
	</view>
</template>

<script setup>
	
</script>

<style scoped lang="less">
.mine-page {
	display: flex;
	flex-direction: column;
	.banner {
		display: flex;
		justify-content: space-around;
		align-items: center;
		width: 100%;
		height: 320rpx;
		border-bottom-left-radius: 40rpx;
		border-bottom-right-radius: 40rpx;
		overflow: hidden;
		background-color: #fff;
		.banner-btn {
			width: 200rpx;
			height: 80rpx;
			line-height: 80rpx;
			font-size: 32rpx;
			border-radius: 16rpx;
			color: #fff;
			text-align: center;
		}
	}
	.banner-text {
		font-size: 28rpx;
		height: 80rpx;
		line-height: 80rpx;
		color: #666;
		text-align: center;
	}
	.mine-bg {
		flex: 1;
		width: 100%;
		border-top-right-radius: 40rpx;
		border-top-left-radius: 40rpx;
		overflow: hidden;
	}
}   
</style>
