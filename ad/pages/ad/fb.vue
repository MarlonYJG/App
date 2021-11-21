<template>
	<view class="fb">
		<textarea class="text" v-model="val" placeholder="请告诉我们,您的感受和建议(必填)" />
		<button class="btn" type="primary" @click="submit">提交</button>
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
				val: ''
			}
		},
		methods: {
			submit() {
				uni.request({
					url: `${BaseUrl}/tfa/appfeedbackrecord/save`,
					method: 'POST',
					header: {
						Authorization: JSON.parse(uni.getStorageSync('Authorization'))
					},
					data: {
						appUserId: '',
						feedback: this.val,
						phoneNo: JSON.parse(uni.getStorageSync('userInfo')).principal
					},
					success: (res) => {
						if (res.data.code === 9998) {
							uni.reLaunch({
								url: '/pages/ad/index'
							})
							return
						}
						if (res.data.code == 200) {
							uni.showToast({
								title: '提交成功'
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
							title: '提交失败,请稍后重试',
							icon: 'none'
						})
					}
				})
			}
		},
	}
</script>

<style scoped lang="scss">
	.fb {
		position: relative;
		width: 98%;
		height: 100vh;
		margin: 0 auto;
		background-color: #fff;

		.text {
			width: 95%;
			height: 500rpx;
			background-color: #eeeeee;
			padding: 20rpx 0;
			margin: 0 auto;
			color: #353535;
			font-size: 30rpx;
			border-radius: 10rpx;
			padding: 10rpx;
		}

		.btn {
			width: 90%;
			margin-top: 50rpx;
			background: #4aa0ff;
			// position: absolute;
			// bottom: 20rpx;
		}
	}
</style>
