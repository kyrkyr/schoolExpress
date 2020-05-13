<template>
	<view class="register">
	
		<view class="content">
			<!-- 头部logo -->
			<!--<view class="header">
				<image :src="logoImage"></image>
			</view>-->
			<view style="margin-top: 256upx;"></view>
			<!-- 主体 -->
			<view class="main">
				<wInput
					v-model="nameData"
					type="text"
					maxlength="20"
					placeholder="用户名"
				></wInput>
				<wInput
					v-model="phoneData"
					type="text"
					maxlength="11"
					placeholder="手机号"
				></wInput>
				<wInput
					v-model="passData"
					type="password"
					maxlength="11"
					placeholder="登录密码"
					isShowPass
				></wInput>
				<!--<wInput
					v-model="verCode"
					type="number"
					maxlength="4"
					placeholder="验证码"
					
					isShowCode
					ref="runCode"
					@setCode="getVerCode()"
				></wInput>-->
					
			</view>
				
			<wButton 
				text="注 册"
				bgColor="#4191ea"
				:rotate="isRotate" 
				@click.native="startReg()"
			></wButton>
			
			<!-- 底部信息 -->
			<view class="footer">
				<text 
					@tap="isShowAgree" 
					class="cuIcon"
					:class="showAgree?'cuIcon-radiobox':'cuIcon-round'"
				>同意</text>
				<!-- 协议地址 -->
				<navigator url="" open-type="navigate">《协议》</navigator>
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
				nameData:'',
				phoneData:'', // 用户/电话
				passData:'', //密码
				//verCode:"", //验证码
				showAgree:true, //协议是否选择
				isRotate: false, //是否加载旋转
			}
		},
		components:{
			wInput,
			wButton,
		},
		methods: {
			isShowAgree(){
				//是否选择协议
				this.showAgree = !this.showAgree;
			},
		    startReg() {
				//注册
				if(this.isRotate){
					//判断是否加载中，避免重复点击请求
					return false;
				}
				if (this.showAgree == false) {
				    uni.showToast({
				        icon: 'none',
						position: 'bottom',
				        title: '请先同意《协议》'
				    });
				    return false;
				}
				if (this.phoneData.length != 11) {
				    uni.showToast({
				        icon: 'none',
						position: 'bottom',
				        title: '手机号不正确'
				    });
				    return false;
				}
		        if (this.passData.length < 6) {
		            uni.showToast({
		                icon: 'none',
						position: 'bottom',
		                title: '密码不正确'
		            });
		            return false;
		        }
				uni.request({
					method: "POST",
					url: app.globalData.mainUrl + 'auth/register',
					data: {
						name: this.nameData,
						password: this.passData,
						phone: this.phoneData
					},
					success: res =>{
						console.log(res);
						if (res.data.code == '000000'){
							console.log("注册成功");
							//提示框
							uni.showToast({
								title: '注册成功'
							});
						}
						else{
							uni.showToast({
								title: res.data.message
							});
						}
					},
					complete: () => {
						
					}
				});
				this.isRotate=true;
				setTimeout(function(){
					this.isRotate=false;
					//uni.navigateTo跳转的返回，默认1为返回上一级
					uni.navigateBack({
						delta: 1
					});
				},3000);
		    }
		}
	}
</script>

<style>
	@import url("../../components/watch-login/css/icon.css");
	@import url("./css/main.css");
</style>