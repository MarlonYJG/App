<template>
	<view class="loginContainer">
		<view class="loginInner">
			<view class="login_header">
				<view class="login_header_title">
					<text>欢迎使用信用分期</text>
				</view>
			</view>
			<!-- 内容部分 -->
			<view class="login_content">
				<view>
					<!-- 短信登录 -->
					<view class="loginform">
						<!-- 手机号 -->
						<view class="login_message">
							<uni-easyinput type="number" class="input" v-model="phone" placeholder="手机号"
								:clearable="false" trim>
							</uni-easyinput>
							<button type="default" :disabled="disabled" class="code"
								@click="getCode">{{codeMsg}}</button>
						</view>
						<!-- 验证码 -->
						<view class="login_verification">
							<uni-easyinput type="text" :maxlength="4" class="input" v-model="code" placeholder="验证码"
								:clearable="false" trim />
						</view>
						<view class="login_hint">
							温馨提示：登录即表示您已阅读、理解并同意
							<text class="xy" @click="jump">《用户服务协议》</text>
						</view>
						<button type="primary" :loading="loading" :disabled="loadDis" @click="login">登录</button>

					</view>

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
				loading: false,
				timer: null,
				computeTime: 0,
				phone: '', //手机号,
				code: '', //验证码
				disabled: false,
				codeMsg: '获取验证码',
				loadDis: false
			};
		},
		computed: {
			rightPhone() {
				//利用正则对手机号匹配
				return /^1[3456789]\d{9}$/.test(this.phone);
			}
		},
		methods: {
			getCode() {
				if (this.rightPhone) {
					this.disabled = true;
					if (!this.computeTime) {
						this.computeTime = 600;
						this.codeMsg = this.computeTime + 's';
						this.timer = setInterval(() => {
							this.computeTime--;
							this.codeMsg = this.computeTime + 's';
							if (this.computeTime <= 0) {
								this.codeMsg = '获取验证码';
								this.disabled = false;
								clearInterval(this.timer)
							}
						}, 1000);
						// 短信倒计开始时即请求服务器
						this.sedCode();
					}
				} else {
					uni.showToast({
						image: "/static/err.png",
						title: "手机号错误"
					})
				}

			},
			// 用户协议
			jump() {
				uni.navigateTo({
					url: '/pages/ad/proinfo_3'
				})
			},

			// 发送验证码
			sedCode() {
				uni.request({
					url: `${BaseUrl}/tfa/util-sms-record/sms`,
					method: 'GET',
					data: {
						phone: this.phone,
						"tag": BaseTag
					},
					success(res) {
						if (res.data.code == 200) {
							uni.showToast({
								title: '短信已发送'
							})
						}
					},
					fail(err) {
						uni.showToast({
							image: "/static/err.png",
							title: "发送失败"
						})
					}
				})

			},
			// 登录
			login() {
				if (this.rightPhone) {
					let {
						brand,
						model,
						platform,
						deviceId
					} = uni.getSystemInfoSync()
					this.loading = true;
					this.loadDis = true;
					uni.request({
						url: `${BaseUrl}/tfa/login`,
						method: 'POST',
						data: {
							"isApprove": 1,
							"smsCode": this.code,
							"username": this.phone,
							"tag": BaseTag,
							"idfa": 'NULL',
							"imeiMd5": model || 'NULL',
							"oaid": deviceId,
							"oaidMd5": deviceId,
							"os": 2
						},
						success: (res) => {
							if (res.data.code == 200) {
								console.log(res)
								uni.setStorageSync('userInfo', JSON.stringify(res.data.data));
								uni.setStorageSync('Authorization', JSON.stringify(res.data.data.token || ''));

								uni.switchTab({
									url: '/pages/ad/home'
								})
							} else {
								uni.showToast({
									image: "/static/err.png",
									title: res.data.msg
								})
							}
						},
						fail: (err) => {
							uni.showToast({
								image: "/static/err.png",
								title: '登录失败'
							})
						},
						complete: () => {
							this.loadDis = false;
							this.loading = false;
						}
					})
				} else {
					uni.showToast({
						image: "/static/err.png",
						title: "手机号错误"
					})
				}
			},
			// 获取设备信息
			getSystem() {
				let {
					brand,
					model,
					platform,
					deviceId
				} = uni.getSystemInfoSync();
				uni.request({
					url: `${BaseUrl}/tfa/app-user-behavior/activate`,
					method: 'POST',
					data: {
						"tag": BaseTag,
						"idfa": 'NULL',
						"imeiMd5": model || 'NULL',
						"oaid": deviceId,
						"oaidMd5": deviceId,
						"os": 2
					}
				})
			}
		},
		onLoad() {
			this.getSystem()
		}
	}
</script>

<style scoped lang="scss">
	.loginContainer {
		width: 100vw;
		height: 100vh;
		background: #fff;
	}

	.loginInner {
		position: absolute;
		top: 40%;
		width: 80%;
		right: 10%;
		transform: translateY(-40%);
		margin: 0 auto;
	}

	.loginInner .login_header .login_logo {
		color: #0faeff;
		font-weight: bolder;
		font-size: 80rpx;
		text-align: center;
	}

	.login_header .login_header_title {
		text-align: center;
		font-size: 55rpx;
		color: #262626;
		font-weight: bolder;
	}

	.login_header_title a {
		text-decoration: none;
		font-size: 20px;
		color: #0b0b0b;
		padding-bottom: 8rpx;
		font-weight: bolder;
	}

	.login_header_title a.on {
		color: #0faeff;
		font-weight: bolder;
		border-bottom: 2px solid #0faeff;
	}

	.loginform {
		.input {
			overflow: hidden;
		}

		/deep/ .uni-easyinput__content {
			input {
				height: 92rpx;
				line-height: 92rpx;
				font-size: 32rpx;
			}
		}
	}

	.login_content form input {
		width: 100%;
		height: 100%;
		border: 1px solid #ddd;
		border-radius: 8rpx;
		outline: none;
		padding-left: 20rpx;
		box-sizing: border-box;

	}

	.login_content form input:focus {
		border: 1px solid #0faeff;
	}

	.login_message {
		position: relative;
		margin-top: 32rpx;
		height: 96rpx;
		font-size: 28rpx;
		background: #fff;
		display: flex;

		.input {
			float: left;
			width: 70%;
			margin-right: 12rpx;
		}

		.code {
			float: right;
			width: 186rpx;
			height: 74rpx;
			line-height: 74rpx;
			margin: 12rpx 0px;
			display: inline-block;
			font-size: 26rpx;
			color: #313131;
		}
	}

	.login_message .get_verification {
		position: absolute;
		top: 50%;
		right: 20rpx;
		transform: translateY(-50%);
		border: none;
		color: #ccc;
		background: transparent;
		font-size: 14px;
	}

	.login_message .get_verification.right_phone {
		color: #000;
	}

	.login_hint {
		text-align: center;
		color: #999;
		margin-top: 24rpx;
		font-size: 14px;
		line-height: 40rpx;
		margin-bottom: 100rpx;

		.xy {
			color: #0faeff;
			cursor: pointer;
		}
	}

	.login_verification {
		position: relative;
		margin-top: 32rpx;
		height: 96rpx;
		font-size: 14px;
		background: #fff;
	}

	.login_verification .switch_button {
		border: 1px solid #ddd;
		width: 60rpx;
		height: 32rpx;
		position: absolute;
		top: 50%;
		right: 20rpx;
		transform: translateY(-50%);
		border-radius: 16rpx;
		padding: 0 12rpx;
		line-height: 32rpx;
		font-size: 12px;
		transition: background-color 0.3s;

	}

	.login_verification .switch_button.on {
		background: #0faeff;
	}

	.login_verification .switch_button.off {
		background: #fff;
	}

	.switch_button .switch_circle {
		background: #fff;
		border: 1px solid #ddd;
		border-radius: 50%;
		width: 32rpx;
		height: 32rpx;
		position: absolute;
		left: -2rpx;
		top: -2rpx;
		box-shadow: 0 2px 4px 0 rgba(0, 0, 0, 0.1);
	}

	.switch_button .switch_circle.right {
		transform: translateX(60rpx);
	}

	.switch_button .switch_text {
		color: #ddd;
		float: right;
	}

	.login_submit {
		display: block;
		width: 100%;
		height: 84rpx;
		margin-top: 60rpx;
		background: #0faeff;
		border-radius: 8rpx;
		font-size: 16px;
		line-height: 84rpx;
		color: #fff;
		text-align: center;
		border: none;
	}

	.register {
		font-size: 14px;
		color: #999;
		position: fixed;
		bottom: 40rpx;
		width: 100%;
		text-align: center;
	}

	.register a {
		text-decoration: none;
		color: #0faeff;
	}

	.forgetPwd {
		text-align: right;
		font-size: 14px;
		padding-top: 30rpx;
	}

	.forgetPwd a {
		text-decoration: none;
		color: #0faeff;
	}
</style>
