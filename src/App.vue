<script setup lang="ts">
import { onLaunch, onShow, onHide } from "@dcloudio/uni-app";
onLaunch(() => {
  console.log("App Launch");
});
onShow(() => {
  console.log("App Show");
});
onHide(() => {
  console.log("App Hide");
});
</script>
<style>
	view {
		padding: 0;
		margin: 0;
		box-sizing: border-box;
	}
	uni-page-body {
		height: 100%;
	}
	.page-wrapper {
		height: 100vh;
		background-color: #f0f0f0;
	}
	.page-title {
		width: 100%;
		height: 80rpx;
		font-size: 32rpx;
		line-height: 80rpx;
		font-weight: 500;
		border-bottom: 1rpx solid #eee;
		text-align: center;
		background-color: #fff;
	}
	
	
	.gradient-bg {
		background: linear-gradient(45deg, #ffc107, deeppink, #9c27b0);
		animation: hueRotate 10s infinite alternate;
	}
	@keyframes hueRotate {
		100% {
			filter: hue-rotate(360deg);
		}
	}
</style>
