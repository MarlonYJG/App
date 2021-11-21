<template>
	<view class="mine">
		<view class="mine-info">
			<view class="mine-info-b">
				<image class="avatar" src="../../static/logo/wuyoudai.png" mode=""></image>
				<text class="name">{{userName}}</text>
				<text class="iconfont settings" @click="jump">&#xe63a;</text>
			</view>
		</view>
		<view class="mine-bg">
			<view class="mine-b">
				<view class="kf">
					<text class="iconfont icon">&#xe740;</text>
					<view class="zxkf">在线客服</view>
				</view>
				<view class="kf" @click="fb">
					<text class="iconfont icon">&#xe61f;</text>
					<view class="zxkf">投诉反馈</view>
				</view>
			</view>
		</view>
		<!-- 更多推荐 -->
		<view class="mine-more">
			<view class="mine-more-bg">
				<view class="ads-list-title-b">
					<view class="title-icon"></view>
					<view class="list-title">更多推荐</view>
				</view>
				<view class="mine-more-list">
					<view class="listItem" v-for="(item,index) in list">
						<image class="logo" :src="item.logoUrl" mode=""></image>
						<view class="logo-rt">
							<view class="logo-name">
								{{item.productName}}
							</view>
							<view class="logo-info">
								额度{{item.loanLimitMin}}{{item.loanLimitMax?'~':''}}{{item.loanLimitMax}}万
							</view>
						</view>
					</view>
				</view>

			</view>
		</view>
	</view>
</template>

<script>
	import uIcons from "@/components/uni-icons/uni-icons.vue"
	import {
		BaseUrl,
		BaseTag
	} from "@/config.js"

	export default {
		components: {
			uIcons
		},
		data() {
			return {
				list: []
			}
		},
		computed: {
			userName() {
				if (uni.getStorageSync('userInfo')) {
					return JSON.parse(uni.getStorageSync('userInfo')).name
				}
				return '无忧贷'
			}
		},
		methods: {
			jump() {
				uni.navigateTo({
					url: '/pages/ad/setting'
				})
			},
			// 更多推荐
			getList() {
				this.list = [];
				uni.request({
					url: `${BaseUrl}/tfa/mine/list?tag=${BaseTag}`,
					method: 'POST',
					header: {
						Authorization: JSON.parse(uni.getStorageSync('Authorization'))
					},
					success: (res) => {
						if (res.data.code === 9998) {
							uni.reLaunch({
								url: '/pages/ad/index'
							})
							return
						}
						if (res.data && res.data.code == 200) {
							let Res = res.data.data;
							if (Res && Res.length) {
								this.list = Res;
							}
						} else {
							uni.showToast({
								title: res.data.msg,
								icon: 'none'
							})
						}
					},
					fail: () => {
						uni.showToast({
							title: '服务器开小差了呢，请您稍后再试',
							icon: 'none'
						})
					}
				})
			},
			fb() {
				uni.navigateTo({
					url: '/pages/ad/fb'
				})
			}
		},
		onLoad() {
			this.getList()
		},
		onTabItemTap() {
			this.getList()
		}
	}
</script>

<style scoped lang="scss">
	.mine {

		overflow: hidden;

		.mine-info {
			overflow: hidden;
			background-color: rgb(25, 137, 245);
			padding-top: 100rpx;

			.mine-info-b {
				width: 93%;
				height: 100rpx;
				margin: 0 auto;
				margin-bottom: 20rpx;
				overflow: hidden;

				.avatar {
					float: left;
					width: 100rpx;
					height: 100rpx;
					border-radius: 50%;
				}

				.name {
					display: inline-block;
					margin-left: 30rpx;
					height: 100rpx;
					line-height: 100rpx;
					font-weight: bolder;
					color: #fff;
					font-size: 44rpx;

				}

				.settings {
					float: right;
					height: 100rpx;
					line-height: 100rpx;
					font-size: 44rpx;
					color: #fff;
				}
			}
		}

		.mine-bg {
			box-sizing: border-box;
			height: 250rpx;
			width: 100vw;
			padding-top: 4rpx;
			background: -webkit-linear-gradient(#1989fa, #dbeaf5);
			/* Safari 5.1 - 6.0 */
			background: -o-linear-gradient(#1989fa, #dbeaf5);
			/* Opera 11.1 - 12.0 */
			background: -moz-linear-gradient(#1989fa, #dbeaf5);
			/* Firefox 3.6 - 15 */
			background: linear-gradient(#1989fa, #dbeaf5);
			/* 标准的语法 */

			.mine-b {
				position: relative;
				height: 200rpx;
				width: 94%;
				background-color: #FFFFFF;
				margin: 0 auto;
				border-radius: 10rpx;
			}

			.kf {
				box-sizing: border-box;
				width: 120rpx;
				overflow: hidden;
				text-align: center;
				float: left;
				margin-left: 50rpx;
				margin-right: 50rpx;
				margin-top: 20rpx;

				.icon {
					font-size: 60rpx;
				}

				.zxkf {
					font-size: 28rpx;
				}
			}

			.kf_icon {
				width: 50rpx;
				height: 50rpx;
			}
		}

		.mine-more {
			position: relative;
			top: -40rpx;
			overflow: hidden;
			width: 100vw;
			margin: 20rpx 0;

			.mine-more-bg {
				overflow: hidden;
				width: 94%;
				background-color: #fff;
				margin: 0 auto;
				box-sizing: border-box;
				border-top-left-radius: 10rpx;
				border-top-right-radius: 10rpx;

				.ads-list-title-b {

					height: 45rpx;
					line-height: 45rpx;
					overflow: hidden;
					box-sizing: border-box;
					margin: 15rpx auto;


					.title-icon {
						float: left;
						display: inline-block;
						width: 8rpx;
						height: 100%;
						background-color: #007AFF;
						margin-right: 15rpx;
						margin-left: 16rpx;
					}

					.list-title {
						font-weight: bolder;
						color: #000000;
						font-size: 34rpx;
					}
				}

				.mine-more-list {
					width: 94%;
					overflow: hidden;
					display: flex;
					flex-direction: row;
					justify-content: space-between;
					flex-wrap: wrap;
					padding: 0 24rpx;

					.listItem {
						width: 48%;
						height: 160rpx;
						border-radius: 10rpx;
						text-align: center;
						line-height: 120rpx;
						border: 1rpx solid #dadada;
						margin-bottom: 15rpx;
						box-sizing: border-box;
						overflow: hidden;

						.logo {
							float: left;
							width: 90rpx;
							height: 90rpx;
							margin: 34rpx 10rpx;

						}

						.logo-rt {
							float: left;
							overflow: hidden;
							width: calc(100% - 120rpx);
							height: 110rpx;
							padding: 24rpx 0;
							padding-right: 10rpx;

							.logo-name {
								height: 40rpx;
								line-height: 40rpx;
								overflow: hidden;
								text-align: left;
								box-sizing: border-box;
								margin-top: 14rpx;
								font-size: 28rpx;
								font-weight: bolder;
								color: #242424;
								white-space: nowrap;
								text-overflow: ellipsis;
							}

							.logo-info {
								height: 50rpx;
								line-height: 50rpx;
								overflow: hidden;
								text-align: left;
								box-sizing: border-box;
								font-size: 22rpx;
								color: #919191;
								white-space: nowrap;
								text-overflow: ellipsis;
							}
						}

					}
				}

			}


		}

	}
</style>
