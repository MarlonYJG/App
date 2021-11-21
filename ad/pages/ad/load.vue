<template>
	<view class="load">
		<uni-nav-bar v-if="!webView" class="navbar" left-icon="back" title="正在跳转至第三方平台..." @clickLeft="back">
		</uni-nav-bar>
		<web-view v-if="webView" :src="webView"></web-view>
		<view class="tz" v-else>
			<view class="tz-title">
				正在跳转至第三方平台
			</view>
			<text class="tz-info">
				您即将进入第三方页面，后续服务均由外部机构提供，请您在后续流程中独立作出判断并承担风险。
			</text>
		</view>

	</view>
</template>

<script>
	let timer = null;
	export default {
		data() {
			return {
				webView: ''
			}
		},
		methods: {
			back() {
				uni.navigateBack({
					delta: 1
				})
			}
		},
		onLoad(urlParams) {
			if (urlParams && urlParams.htmlUrl) {
				timer = setTimeout(() => {
					this.webView = urlParams.htmlUrl
				}, 1000)
			}
		},
		onUnload() {
			timer && clearTimeout(timer)
		}
	}
</script>

<style scoped lang="scss">
	.navbar {
		height: 60px;
		font-weight: bolder;
	}

	.load {
		position: relative;
		width: 100vw;
		height: 100vh;

		.tz {
			position: absolute;
			overflow: hidden;
			width: 440rpx;
			left: 50%;
			top: 50%;
			margin: -220rpx 0 0 -220rpx;

			.tz-title {
				width: 440rpx;
				text-align: center;
				color: #2c2c2c;
				font-weight: bolder;
				font-size: 28rpx;
				margin-bottom: 25rpx;
			}

			.tz-info {
				display: inline-block;
				width: 440rpx;
				text-align: left;
				color: #969799;
			}
		}

	}
</style>
