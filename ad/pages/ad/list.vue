<template>
	<view class="ad-list">
		<!-- 列表 -->
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
						<view class="info info_2_sd">月利率：<text class="h">{{item.interestRate}}%</text></view>
						<view class="info info_2_sq" @click="getReq(item)"><text class="info_2_sq_text">立即申请</text>
						</view>
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
				pageSize: 10,
				pageNo: 1,
				loadStatus: 'loading', //加载样式：more-加载前样式，loading-加载中样式，nomore-没有数据样式
				isLoadMore: false, //是否加载中
				isNoLoad: true
			}
		},
		methods: {
			getList(val) {
				let productName = val || ''
				uni.request({
					url: `${BaseUrl}/tfa/full/page/list?tag=${BaseTag}&productName=${productName}`,
					method: 'POST',
					header: {
						Authorization: JSON.parse(uni.getStorageSync('Authorization'))
					},
					data: {
						pageNo: this.pageNo,
						pageSize: this.pageSize
					},
					success: (res) => {
						if (res.data.code === 9998) {
							uni.reLaunch({
								url: '/pages/ad/index'
							})
							return
						}
						if (res.data && res.data.code == 200) {
							let Res = [];
							if (res.data.data && res.data.data && res.data.data.data) {
								Res = res.data.data.data;
							}

							if (Res && Res.length) {
								this.list = this.list.concat(Res);
								if (Res.length < this.pageSize) {
									this.isLoadMore = true
									this.loadStatus = 'nomore'
								} else {
									this.isLoadMore = false
								}
							} else {
								this.isLoadMore = true
								this.loadStatus = 'nomore'
							}
						} else {
							uni.showToast({
								title: res.data.msg,
								icon: 'none'
							})
							this.isLoadMore = false
							if (this.pageNo > 1) {
								this.pageNo -= 1
							}
						}
					},
					fail: () => {
						uni.showToast({
							title: '服务器开小差了呢，请您稍后再试',
							icon: 'none'
						})
						this.isLoadMore = false
						if (this.pageNo > 1) {
							this.pageNo -= 1
						}
					},
					complete: () => {
						this.isNoLoad = true;
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
									url: `/pages/ad/load?htmlUrl=${item.htmlUrl}`
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
		mounted() {
			this.pageNo = 1;
			this.getList()
		},
		onPullDownRefresh() {
			this.pageNo = 1;
			this.isNoLoad = false;
			this.list = [];
			this.getList()
		},
		onReachBottom() {
			if (!this.isLoadMore) { //此处判断，上锁，防止重复请求
				this.isLoadMore = true
				this.pageNo += 1
				this.getList()
			}
		},
		onNavigationBarSearchInputConfirmed(val) {
			this.pageNo = 1;
			this.isNoLoad = false
			this.list = [];
			this.getList(val.text)
		},
		onTabItemTap() {
			this.pageNo = 1;
			this.getList()
		}

	}
</script>

<style scoped lang="scss">
	.ad-list {
		height: 100vh;
		background-color: #ececec;
		padding: 10rpx 30rpx 20px 30rpx;
		overflow: auto;
	}

	/* 广告Item */
	.adItem {
		overflow: hidden;
		margin-bottom: 30rpx;
		padding: 25rpx 25rpx;
		background-color: #FFFFFF;
		border-radius: 6rpx;
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
