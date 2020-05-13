<template>
	<view class="login">
		<view class="content">
			<!-- 头部logo -->
			<!--<view class="header">
				<image :src="logoImage"></image>
			</view>-->
			<view style="margin-top: 356upx;"></view>
			<!-- 主体表单 -->
			<view class="main">
				<wInput
					v-model="phoneData"
					type="text"
					maxlength="11"
					placeholder="请输入手机号"
				></wInput>
				<wInput
					v-model="passData"
					type="password"
					maxlength="11"
					placeholder="请输入密码"
				></wInput>
			</view>
			<wButton 
				text="登 录"
				bgColor="#4191ea"
				:rotate="isRotate"
				@click.native="startLogin()"
			></wButton>
			
			<!-- 其他登录 -->
			<!--<view class="other_login cuIcon">
				<view class="login_icon">
					<view class="cuIcon-weixin" @tap="login_weixin"></view>
				</view>
				<view class="login_icon">
					<view class="cuIcon-weibo" @tap="login_weibo"></view>
				</view>
				<view class="login_icon">
					<view class="cuIcon-github" @tap="login_github"></view>
				</view>
			</view>-->
			
			<!-- 底部信息 -->
			<view class="footer">
				<navigator url="forget" open-type="navigate">找回密码</navigator>
				<text>|</text>
				<navigator url="register" open-type="navigate">注册账号</navigator>
			</view>
		</view>
	</view>
</template>

<script>
	const app = getApp()
	import wInput from '../../components/watch-login/watch-input.vue' //input
	import wButton from '../../components/watch-login/watch-button.vue' //button
	
	export default {
		data() {
			return {
				//logo图片 base64
				logoImage: '',
				phoneData:'', //用户/电话
				passData:'', //密码
				isRotate: false, //是否加载旋转
			}
		},
		components:{
			wInput,
			wButton,
		},
		methods: {
			startLogin(){
				//登录
				if(this.isRotate){
					//判断是否加载中，避免重复点击请求
					return false;
				}
				if (this.phoneData.length != 11) {
					 uni.showToast({
						icon: 'none',
						position: 'bottom',
						title: '手机号不合法'
					});
					return;
				}
				if (this.passData.length < 6) {
					uni.showToast({
						icon: 'none',
						position: 'bottom',
						title: '密码不正确'
					});
					return;
				}
				uni.request({
					method: "POST",
					url: app.globalData.mainUrl + 'auth/login',
					data: {
						password: this.passData,
						phone: this.phoneData,
						uuid: uni.getStorageSync('uuid'),
						brand: uni.getSystemInfoSync().brand,
						model: uni.getSystemInfoSync().model
					},
					success: res =>{
						console.log(res);
						if (res.data.code == '000000'){
							uni.setStorage({
								key: 'token',
								data: res.data.data.token,
								success: function () {
	                                uni.getStorage({
	                                    key: 'token',
	                                    success: function (res) {
	                                        console.log(res.data);
	                                    }
	                                });
	                            }
							});
							console.log("登录成功");
							this.isRotate=true
							setTimeout(function(){
								this.isRotate=false
							},500)
							uni.reLaunch({
								url: '../index/index'
							});
						}
						else{
							uni.showToast({
								icon: "none",
								title: res.data.message
							});
						}
					},
					complete: () => {
					}
				});
			},
			login_weixin() {
				//微信登录
				uni.showToast({
					icon: 'none',
					position: 'bottom',
					title: '...'
				});
			},
			login_weibo() {
				//微博登录
				uni.showToast({
					icon: 'none',
					position: 'bottom',
					title: '...'
				});
			},
			login_github() {
				//github登录
				uni.showToast({
					icon: 'none',
					position: 'bottom',
					title: '...'
				});
			},
			getDeviceUuid(){
				//同步获取设备唯一标识
				var mainActivity = plus.android.runtimeMainActivity();
				var Settings = plus.android.importClass("android.provider.Settings");
				var uuid = Settings.Secure.getString(mainActivity.getContentResolver(), Settings.Secure.ANDROID_ID);
				console.log("设备唯一标识为：" + uuid);
				uni.setStorage({
					key: 'uuid',
					data: uuid,
				});
			},
			getToken(){
				uni.request({
					method: "POST",
					url: app.globalData.mainUrl + 'auth/getToken?uuid=' + uni.getStorageSync('uuid')
					 + '&brand=' + uni.getSystemInfoSync().brand
					 + '&model=' + uni.getSystemInfoSync().model,
					success: res =>{
						console.log(res);
						if (res.data.code == '000000'){
							uni.setStorage({
								key: 'token',
								data: res.data.data,
								success: function () {
	                                uni.getStorage({
	                                    key: 'token',
	                                    success: function (res) {
	                                        console.log(res.data);
	                                    }
	                                });
	                            }
							});
							console.log("获取token成功");
							uni.reLaunch({
								url: '../index/index'
							});
						}
					},
					complete: () => {
						
					}
				});
			}	
		},
		created() {
			this.getDeviceUuid();
			this.getToken();
		}
	}
</script>

<style>
	@import url("../../components/watch-login/css/icon.css");
	@import url("./css/main.css");
</style>
