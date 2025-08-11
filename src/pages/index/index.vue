<script setup lang="ts">
interface listItem {
	title: string,
	desc: string,
	url: string
}
const list: listItem[] = [
	{
		title: '小程序名字查询',
		desc: '通过 AppId 查询',
		url: '/pages/tools/search-miniapp-name'
	},
	// {
	// 	title: '照片添加水印',
	// 	desc: '光圈、焦距、ISO等',
	// 	url: '/pages/tools/add-photo-mask'
	// },
	// {
	// 	title: '随机做饭挑战',
	// 	desc: '还在纠结今天吃啥吗？快来随机一个吧',
	// 	url: '/pages/tools/random-cook'
	// },
	// {
	// 	title: '随机旅游挑战',
	// 	desc: '还在纠结去哪玩吗？快来随机一个吧',
	// 	url: '/pages/tools/random-travel'
	// }
]

function handleClick(item: listItem) {
	uni.navigateTo({
		url: item.url
	})
}
</script>

<template>
  <view class="page-wrapper index-page">
    <view
			v-for="(item, index) in list"
			:key="index"
			class="list-item"
			@click="handleClick(item)"
		>
			<view class="icon-wrapper">
				<img class="icon-img" src="/static/index/tool.png" alt="" />
			</view>
			<view class="item-info">
				<view class="item-title">{{ item.title }}</view>
				<view class="item-desc">{{ item.desc }}</view>
			</view>
		</view>
		<view class="list-item">
			<view class="icon-wrapper">
				<img class="icon-img" src="/static/index/tool.png" alt="" />
			</view>
			<view class="item-info-single">更多工具，敬请期待...</view>
		</view>
  </view>
</template>

<style scoped lang="less">
.index-page {
  padding: 20rpx 32rpx;
	overflow: auto;
	.list-item {
		display: flex;
		padding: 20rpx;
		border-radius: 16rpx;
		background-color: #fff;
		&:not(:last-child) {
			margin-bottom: 16rpx;
		}
		.icon-wrapper {
			display: flex;
			justify-content: center;
			align-items: center;
			width: 80rpx;
			height: 80rpx;
			margin-right: 20rpx;
			overflow: hidden;
			.icon-img {
				width: 100%;
				height: 100%;
			}
		}
		.item-info-single {
			line-height: 80rpx;
			color: #666;
		}
		.item-title {
			font-size: 32rpx;
			margin-bottom: 10rpx;
		}
		.item-desc {
			font-size: 20rpx;
			color: #666;
		}
	}
}
</style>
