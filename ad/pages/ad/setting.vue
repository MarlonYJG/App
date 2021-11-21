<template>
	<view class="settings">
		<!-- 协议中心 -->
		<view class="xyzx" @click="jump(1)">
			<text>个人身份认证协议</text>
			<uni-icons class="arrowr-rt" type="arrowright"></uni-icons>
		</view>
		<view class="xyzx" @click="jump(2)">
			<text>个人信息保护协议</text>
			<uni-icons class="arrowr-rt" type="arrowright"></uni-icons>
		</view>
		<view class="xyzx" @click="jump(3)">
			<text>用户注册协议</text>
			<uni-icons class="arrowr-rt" type="arrowright"></uni-icons>
		</view>
		<!-- 退出 -->
		<button type="default" class="out" @click="layout">退出</button>
	</view>
</template>

<script>
	import {
		BaseUrl,
		BaseTag
	} from "@/config.js"

	export default {
		data() {
			return {
				key: 'value'
			}
		},
		methods: {
			// 退出
			layout() {
				// 清空数据
				uni.removeStorageSync('userInfo');
				uni.request({
					url: `${BaseUrl}/tfa/logout`,
					method: 'POST',
					header: {
						Authorization: JSON.parse(uni.getStorageSync('Authorization'))
					},
					data: {
						tag: BaseTag
					},
					complete: () => {
						uni.removeStorageSync('Authorization');
						uni.navigateTo({
							url: '/pages/ad/index'
						})
					}
				})
			},
			// 协议跳转
			jump(type) {
				switch (type) {
					case 1: {
						uni.navigateTo({
							url: '/pages/ad/proinfo_1'
						})
						return;
					}
					case 2: {
						uni.navigateTo({
							url: '/pages/ad/proinfo_2'
						})
						return;
					}
					case 3: {
						uni.navigateTo({
							url: '/pages/ad/proinfo_3'
						})
						return;
					}
					default:
						break;
				}

			}
		},
	}
</script>

<style scoped lang="scss">
	.settings {
		width: 100vw;
		height: 100%;

		.xyzx {
			height: 100rpx;
			padding: 16rpx 30rpx;
			line-height: 100rpx;
			font-size: 30rpx;
			background: #fff;
			margin-top: 10rpx;
			color: #393939;

			.arrowr-rt {
				float: right;
			}
		}

		.out {
			margin-top: 20rpx;
			font-size: 35rpx;

		}
	}
</style>
