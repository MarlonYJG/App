<template>
	<view class="ads">
		<uni-nav-bar class="navbar" title="首页"></uni-nav-bar>
		<view class="ads-banner">
			<swiper :indicator-dots="false" :autoplay="true" :interval="3000" :duration="1000" :circular="true">
				<swiper-item v-for="(item,index) in banner" :key="index">
					<view class="swiper-item">
						<view class="banner">
							<view class="banner-b">
								<text class="banner-top">最高可借额度(元)</text>
								<view class="banner-mid">
									<text class="banner-mid-num">{{item.loanLimitMax}}</text>
									<text class="banner-mid-text" @click="getReq(item)">立即申请</text>
								</view>
								<view class="banner-bot">
									<image :src="item.logoUrl" mode="" class="banner-icon"></image>
									<text class="banner-text">{{item.productName}}</text>
								</view>
							</view>
						</view>
					</view>
				</swiper-item>
			</swiper>
		</view>
		<view class="ads-b">
			<view class="ads-list">
				<view class="ads-list-title-b">
					<view class="title-icon"></view>
					<view class="list-title">精准推荐</view>
				</view>
				<!-- 列表 -->
				<view class="ads-list-b">
					<template v-for="(item,index) in list">
						<view class="adItem" :key="index">
							<view class="adItem-top">
								<image :src="item.logoUrl" mode=""></image>
								<view class="adItem-top-rt">
									<view class="adItem-name">{{item.productName}}</view>
									<template v-for="(item_,i) in item.tagVoList">
										<view :key="i" class="adItem-tag adItem-tag_1" v-if="item_.tagName">
											{{item_.tagName}}
										</view>
									</template>
								</view>
							</view>
							<view class="adItem-bot">
								<view class="adItem-bot-info">
									<view class="info info_1_jg">
										{{item.loanLimitMin}}{{item.loanLimitMax?'~':''}}{{item.loanLimitMax}}
									</view>
									<view class="info info_1_sd">放贷速度：<text
											class="h">{{item.lendingRate}}{{item.lendingUnit}}小时</text></view>
									<view class="info info_1_sq">
										<text class="info_1_num">{{item.applicationNum ||0}}</text>
										<text>人申请</text>
									</view>
								</view>
								<view class="adItem-bot-info">
									<view class="info info_2_jg">额度范围(元)</view>
									<view class="info info_2_sd">月利率：<text class="h">{{item.interestRate}}%</text>
									</view>
									<view class="info info_2_sq" @click="getReq(item)"><text
											class="info_2_sq_text">立即申请</text></view>
								</view>
							</view>
						</view>
					</template>
					<template v-if="isNoLoad">
						<view v-show="isLoadMore">
							<uni-load-more :status="loadStatus"></uni-load-more>
						</view>
					</template>
				</view>
			</view>
		</view>
	</view>
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
				list: [],
				banner: [],
				pageSize: 10,
				pageNo: 1,
				loadStatus: 'loading', //加载样式：more-加载前样式，loading-加载中样式，nomore-没有数据样式
				isLoadMore: false, //是否加载中
				isNoLoad: true
			}
		},
		methods: {
			// 推荐列表
			getList() {
				uni.request({
					url: `${BaseUrl}/tfa/main/list?tag=${BaseTag}`,
					method: 'POST',
					header: {
						Authorization: JSON.parse(uni.getStorageSync('Authorization'))
					},
					data: {
						tag: BaseTag
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
								// if (Res.length < this.pageSize) {
								// 	this.isLoadMore = true
								// 	this.loadStatus = 'nomore'
								// } else {
								// 	this.isLoadMore = false
								// }
							} else {
								// this.isLoadMore = true
								// this.loadStatus = 'nomore'
							}
						} else {
							uni.showToast({
								title: res.data.msg,
								icon: 'none'
							})
							// this.isLoadMore = false
							// if (this.pageNo > 1) {
							// 	this.pageNo -= 1
							// }
						}
					},
					fail: () => {
						uni.showToast({
							title: '服务器开小差了呢，请您稍后再试',
							icon: 'none'
						})
						this.isLoadMore = false
						// if (this.pageNo > 1) {
						// 	this.pageNo -= 1
						// }
					},
					complete: () => {
						this.isNoLoad = true;
						uni.stopPullDownRefresh()
					}
				})
			},
			// 推荐位(banner)
			getTop() {
				uni.request({
					url: `${BaseUrl}/tfa/main/page/recommend?tag=${BaseTag}`,
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
								this.banner = Res;
							}
						}
					},
					complete: () => {
						uni.stopPullDownRefresh()
					}
				})
			},
			// 立即申请
			getReq(item) {
				if (item && item.id) {
					uni.request({
						url: `${BaseUrl}/tfa/full/application?tag=${BaseTag}&customerProductId=${item.id}`,
						method: 'GET',
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
							if (res.data.code == 200) {
								uni.navigateTo({
									url: `/pages/ad/load?htmlUrl=${item.htmlUrl}`,
								})
							} else {
								uni.showToast({
									title: res.data.msg,
									icon: 'none'
								})
							}
						},
						fail: (err) => {
							uni.showToast({
								title: '申请失败,请稍后重试',
								icon: 'none'
							})
						}
					})
				}

			}
		},
		onLoad() {
			this.getList()
			this.getTop()
		},
		onPullDownRefresh() {
			this.isNoLoad = false;
			this.list = [];
			this.getList()
		},
		onReachBottom() {
			// if (!this.isLoadMore) { //此处判断，上锁，防止重复请求
			// 	this.isLoadMore = true
			// 	this.pageNo += 1
			// 	this.getList()
			// }
		},
		onTabItemTap() {
			this.getList()
			this.getTop()
		}
	}
</script>

<style scoped lang="scss">
	.ads {
		.navbar {
			box-sizing: border-box;
			background-color: #e80000 !important;
			padding-top: 40px;

			/deep/.uni-navbar__header {
				height: 44px;
				background-color: #e80000 !important;
				border-color: #e80000;
				box-sizing: border-box;
			}

			/deep/ .uni-nav-bar-text {
				color: #ffffff !important;
				font-size: 40rpx;
				font-weight: bolder;
			}
		}
	}

	.ads-banner {
		position: relative;
		top: -1px;
		box-sizing: border-box;
		height: 300rpx;
		width: 100vw;
	}

	.swiper-item {
		padding-top: 10rpx;
		width: 100%;
		height: 100%;
		background: -webkit-linear-gradient(#e80000, #ffefef);
		/* Safari 5.1 - 6.0 */
		background: -o-linear-gradient(#e80000, #ffefef);
		/* Opera 11.1 - 12.0 */
		background: -moz-linear-gradient(#e80000, #ffefef);
		/* Firefox 3.6 - 15 */
		background: linear-gradient(#e80000, #ffefef);
		/* 标准的语法 */
	}

	.banner {
		width: 100%;
		height: 248rpx;
		width: 94%;
		margin: 0 auto;
		background-color: #FFFFFF;
		overflow: hidden;
		border-radius: 10rpx;
	}

	.banner-b {
		padding: 25rpx 25rpx;
		height: 100%;
		overflow: hidden;
	}

	.banner-top {
		color: #919191;
		font-size: 34rpx;
	}

	.banner-mid {
		height: 89rpx;
		line-height: 89rpx;
		overflow: hidden;

	}

	.banner-mid-num {
		display: inline-block;
		font-size: 60rpx;
		color: #006cd8;
		font-weight: bolder;
		max-width: 450rpx;
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;
	}

	.banner-mid-text {
		float: right;
		height: 60rpx;
		line-height: 60rpx;
		font-size: 34rpx;
		background-color: #006cd8;
		border-radius: 35rpx;
		padding: 0 30rpx;
		color: #FFFFFF;
		margin-top: 15rpx;
	}

	.banner-bot {
		font-size: 30rpx;
		height: 45rpx;
		line-height: 45rpx;
		padding-top: 10rpx;
		border-top: 1rpx solid #f1f1f1;
	}

	.banner-text {
		position: relative;
		top: -8rpx;
		font-size: 30rpx;
		margin-left: 20rpx;
		color: #000000;

	}

	.banner-icon {
		width: 40rpx;
		height: 40rpx;
		background-size: 100% 100%;
	}

	.ads-b {
		padding: 0rpx 20rpx;
	}

	.ads-list {
		margin-top: 25rpx;
	}


	.ads-list-title-b {
		height: 45rpx;
		line-height: 45rpx;
		overflow: hidden;
		box-sizing: border-box;
	}

	.title-icon {
		float: left;
		display: inline-block;
		width: 8rpx;
		height: 100%;
		background-color: #007AFF;
		margin-right: 15rpx;
	}

	.list-title {
		font-weight: bolder;
		color: #000000;
		font-size: 34rpx;
	}

	.ads-list-b {
		padding: 0 15rpx;
		overflow: hidden;
		padding-bottom: 25rpx;
	}

	/* 广告Item */
	.adItem {
		overflow: hidden;
		margin-bottom: 30rpx;
		box-sizing: border-box;
		border-bottom: 1rpx solid #dadada;
	}

	.adItem:first-child {
		margin-top: 50rpx;
	}

	.adItem:last-child {
		margin-bottom: 0;
		border-bottom: 0;
	}

	.adItem-top {
		overflow: hidden;
		box-sizing: border-box;

	}

	.adItem-top>image {
		float: left;
		width: 75rpx;
		height: 75rpx;
		box-sizing: border-box;
		background-size: 100% 100%;
	}

	.adItem-top-rt {
		overflow: hidden;
		height: 46rpx;
		line-height: 46rpx;
		margin-top: 24rpx;
	}

	.adItem-name {
		position: relative;
		bottom: 0;
		float: left;
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;
		max-width: 300rpx;
		margin: 0 20rpx;
		font-weight: 600;
		font-size: 30rpx;

	}

	.adItem-tag {
		float: left;
		padding: 0 10rpx;
		background: #c58300 linear-gradient(to right, rgba(255, 179, 0, 0.0), rgba(255, 247, 0, 0.5));
		transition: background-color .5s;
		color: #FFFFFF;
		font-size: 24rpx;
		border-radius: 4rpx;
		height: 36rpx;
		line-height: 36rpx;
		margin-top: 7rpx;
	}

	.adItem-tag_1 {
		margin-right: 15rpx;
	}

	.adItem-bot {
		box-sizing: border-box;
		margin-top: 20rpx;
	}

	.adItem-bot-info {
		display: flex;
		color: #919191;
		height: 50rpx;
		box-sizing: border-box;
	}

	.info {
		flex: 1;
		font-size: 22rpx;
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;
	}

	.info_1_jg {
		text-align: left;
		font-size: 30rpx;
		font-weight: bolder;
		color: #d10000;
	}

	.info_2_jg {
		text-align: left;
	}

	.info_1_sd,
	.info_2_sd {
		text-align: left;
	}

	.info_1_sq {
		text-align: right;
	}

	.info_2_sq {
		text-align: right;
	}

	.info_2_sq_text {
		display: inline-block;
		background-color: #006cd8;
		color: #FFFFFF;
		width: 120rpx;
		height: 40rpx;
		line-height: 40rpx;
		border-radius: 20rpx;
		text-align: center;
	}

	.h {
		color: #000000;
	}

	.info_1_num {
		color: #d10000;
		font-weight: bolder;

	}
</style>
