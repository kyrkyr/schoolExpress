<template>
	<view class="forget">
		
		<view class="content">
			<view style="margin-top: 256upx;"></view>
			<!-- 主体 -->
			<view class="main">
				<view class="tips">若你忘记了密码，可在此重置新密码。</view>
				<wInput
					v-model="phoneData"
					type="text"
					maxlength="11"
					placeholder="请输入手机号码"
				></wInput>
				<wInput
					v-model="passData"
					type="password"
					maxlength="11"
					placeholder="请输入新密码"
					isShowPass
				></wInput>
				
				<wInput
					v-model="verCode"
					type="number"
					maxlength="4"
					placeholder="验证码"
					
					isShowCode
					codeText="获取重置码"
					setTime="60"
					ref="runCode"
					@setCode="getVerCode()"
				></wInput>
			</view>
			
			<wButton 
				text="重置密码"
				bgColor="#4191ea"
				:rotate="isRotate" 
				@click.native="startRePass()"
			></wButton>

		</view>
	</view>
</template>

<script>
	const app = getApp()
	var _this;
	import wInput from '../../components/watch-login/watch-input.vue' //input
	import wButton from '../../components/watch-login/watch-button.vue' //button
	export default {
		data() {
			return {
				phoneData: "", //电话
				passData: "", //密码
				verCode:"", //验证码
				code:"",
				phone:"",
				isRotate: false, //是否加载旋转
			}
		},
		components:{
			wInput,
			wButton
		},
		mounted() {
			_this= this;
		},
		methods: {
			getVerCode(){
				//获取验证码
				if (_this.phoneData.length != 11) {
				     uni.showToast({
				        icon: 'none',
						position: 'bottom',
				        title: '手机号不正确'
				    });
				    return false;
				}
				console.log("获取验证码")
				//this.$refs.runCode.$emit('runCode'); //触发倒计时（一般用于请求成功验证码后调用）
				uni.request({
					method: "POST",
					url: app.globalData.mainUrl + 'auth/sendSms',
					data: {
						phone: this.phoneData,
					},
					success: res =>{
						console.log(res);
						if (res.data.code == '000000'){
							this.$refs.runCode.$emit('runCode');
							_this.phone = res.data.data.phone;
							_this.code = res.data.data.code;
						}
						else{
							uni.showToast({
								icon: "none",
								title: res.data.message
							});
						}
					}
				});
			},
			startRePass() {
				//重置密码
				if(this.isRotate){
					//判断是否加载中，避免重复点击请求
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
				console.log(this.code);
				if (this.verCode.length != 4 || this.verCode != this.code) {
				    uni.showToast({
				        icon: 'none',
						position: 'bottom',
				        title: '验证码不正确'
				    });
				    return false;
				}
				uni.request({
					method: "POST",
					url: app.globalData.mainUrl + 'auth/resetPsw',
					data: {
						phone: this.phoneData,
						password: this.passData,
						code: this.verCode
					},
					success: res =>{
						console.log(res);
						if (res.data.code == '000000'){
							console.log("重置密码成功");
							uni.showToast({
								title:'成功'
							});
							_this.isRotate=true;
							setTimeout(function(){
								_this.isRotate=false;
								uni.reLaunch({
									url:'login'
								})
							},500);
						}
						else{
							uni.showToast({
								icon: "none",
								title: res.data.message
							});
						}
					}
				});
			}
		}
	}
</script>

<style>
	@import url("../../components/watch-login/css/icon.css");
	@import url("./css/main.css");
</style>

