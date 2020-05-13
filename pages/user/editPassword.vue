<template>
	<view class="list-content">
		<view class="cu-bar bg-white margin-top solid-bottom">
			<view class="action" style="width: 100%;">
				<image src="../../static/cate/fanhui.png" style="width: 50upx; height: 40upx;" @click="fanhui"></image>
				<text style="margin-left: 33%; width: 80%; font-size: large;">修改密码</text> 
			</view>
		</view>
		
		<view class="list">
			
			<view class="li" >
				<view class="text">原密码</view>
				<view class="right-text">
					<input type="text" v-model="oldPsw" style="color: #000;" />
				</view>
			</view>
			
			<view class="li" >
				<view class="text">新密码</view>
				<view class="right-text">
					<input type="text" v-model="newPsw" style="color: #000;" />
				</view>
			</view>
			
			<view class="li" >
				<view class="text">再次输入</view>
				<view class="right-text">
					<input type="text" v-model="reNewPsw" style="color: #000;" />
				</view>
			</view>
			
		</view>
		
		<view>
			<button type="default" @tap="save" style="color: #007AFF;">确定</button>
		</view>
	</view>

	
</template>

<script>
	const app = getApp()
	export default {
		data() {
			return {
				oldPsw: '',
				newPsw: '',
				reNewPsw: ''
			}
		},
		methods:{	
			fanhui(){
				uni.navigateBack({
					delta: 1
				});
			},
			save(){
				if (this.newPsw != this.reNewPsw) {
					 uni.showToast({
						icon: 'none',
						position: 'bottom',
						title: '两次输入密码不同'
					});
					return;
				}
				uni.request({
					url: app.globalData.mainUrl + "user/edit",
	                method:'POST',
					header:{
						'Authorization' : uni.getStorageSync('token')
					},
					data:{
						oldPassword: this.oldPsw,
						password: this.newPsw
					},
	                success: function (res) {
	                    console.log(res.data);
						if (res.data.code == '000000'){
							uni.showToast({
								title:'修改成功'
							});
							setTimeout(function(){
								uni.navigateBack({
									delta: 1
								})
							},500);
						}else{
							uni.showToast({
								icon: "none",
								title: res.data.message
							});
						}
	                }
				});
			},
		},
		created() {
			
		}
	}
	
</script>

<style>
	@import "../../colorui/main.css";
	@import "../../colorui/icon.css";
	
	.list-content{
		background: #fff;
		margin-top: 30upx;
	}
	
	.list{
		width:100%;
		border-bottom:15upx solid  #f1f1f1;
		background: #fff;
	}
	
	.li{
		width:100%;
		height:120upx;
		padding:0 4%;
		border-bottom:1px solid rgb(243,243,243);
		display:flex;
		align-items:center;
	}
	
	.text{
		padding-left:20upx;
		width:30%;
		color:#666;
		font-size: medium;
	}
	.right-text{
		width: 70%; 
		padding-right: 20upx; 
		text-align: right;
		font-size: medium;
		color:#666;
	}
	
	
</style>
