<template>
	<view class="refresh" :style="{color: color}">
		<view class="refresh-icon" :style="{'border-top': '5rpx solid' + color, 'border-left': '5rpx solid' + color, 'border-bottom': '5rpx solid' + color}"></view>
		<slot>下拉刷新</slot>
	</view>
</template>

<script>
	export default {
		props: {
			//字体颜色
			color: {
				type: String,
				default: '#333333'
			}
		}
	}
</script>

<style scoped>
	@keyframes animationRotate  {
	  0% {
	    transform: rotateZ(0);
	  }
	  100% {
	    transform: rotateZ(360deg);
	  }
	}
	.refresh {
		width: 100%;
		height: 100rpx;
		display: flex;
		align-items: center;
		justify-content: center;
	}
	.refresh-icon {
		width: 30rpx;
		height: 30rpx;
		border-radius: 30rpx;
		border-right: 5rpx solid transparent;
		margin-right: 10rpx;
		animation: animationRotate 1s linear infinite;
	}
</style>
