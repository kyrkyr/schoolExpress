<template>
	<view>
		<view class="header">
			<view class="bg">
				<view class="box">
					<view class="box-hd">
						<view class="avator">
							<!--<img src="../../static/face.jpg"></img>-->
							<image style="width: 160upx;height: 160upx;" :src="userInfo.url" mode="" v-bind:id="userInfo.url"></image>
						</view>
						<view class="phone-number">
							<button v-if="!userInfo.sign" style="width: 180upx; height: 65upx;font-size: small; border-radius: 1000upx; background-color: #CCE6FF;" @click="sign">点击签到</button>
							<button v-if="userInfo.sign" style="width: 150upx; height: 65upx;font-size: small; border-radius: 1000upx;">已签到</button>
						</view>
					</view>
					<view class="box-bd">
						<view class="item">
							<view class="icon">记账天数</view>
							<view class="text">{{userInfo.day}}</view>
						</view>
						<view class="item">
							<view class="icon">积分</view>
							<view class="text">{{userInfo.score}}</view>
						</view>
						<view class="item">
							<view class="icon">虚拟币</view>
							<view class="text">{{userInfo.money}}</view>
						</view>
					</view>
				</view>
			</view>
		</view>
		<view class="list-content">
			<view class="list">
				<view class="li noborder" @click="toMyMesg">
					<view class="icon"><image src="../../static/card.png"></image></view>
					<view class="text">我的信息</view>
					<image class="to" src="../../static/to.png"></image>
				</view>
			</view>
			<view class="list">
				<view class="li " @click="toAnalyze">
					<view class="icon"><image src="../../static/cate/choujiang.png"></image></view>
					<view class="text">收支分析</view>
					<image class="to" src="../../static/to.png"></image>
				</view>
				<!-- 跳转到微信app -->
				<view class="li " @click="toWeiXin">
					<view class="icon"><image src="../../static/cate/choujiang.png"></image></view>
					<view class="text">微信账单</view>
					<image class="to" src="../../static/to.png"></image>
				</view>
				<!-- 跳转到支付宝app -->
				<view class="li " @click="toZFB">
					<view class="icon"><image src="../../static/cate/choujiang.png"></image></view>
					<view class="text">支付宝账单</view>
					<image class="to" src="../../static/to.png"></image>
				</view>
				<view class="li " @click="toLuckyDraw">
					<view class="icon"><image src="../../static/cate/choujiang.png"></image></view>
					<view class="text">幸运抽奖</view>
					<image class="to" src="../../static/to.png"></image>
				</view>
			</view>
			<view class="list">
				<view class="li noborder" @click="toEditPassword">
					<view class="icon"><image src="../../static/about.png"></image></view>
					<view class="text">修改密码</view>
					<image class="to" src="../../static/to.png"></image>
				</view>
			</view>
		</view>
		<view class="btn-row">
			<button type="default" @tap="bindLogout">退出登录</button>
		</view>
	</view>
</template>

<script>
	const app = getApp()
	export default {
		data() {
			return {
				userInfo: {
					url:'face.jpg',
					name:'',
					phone:'',
					gender:'',
					description:'',
					day:0,
					score:0,
					money:0,
					current: 2,
					sign: false,
				},
			}
		},
		onShow() {
			this.getUserInfo(this.userInfo);
		},
		methods: {
			toEditPassword(){
				uni.navigateTo({
					url: './editPassword'
				})
			},
			//签到
			sign(){
				uni.request({
					url: app.globalData.mainUrl + "user/edit",
	                method:'POST',
					header:{
						'Authorization' : uni.getStorageSync('token')
					},
					data:{
						isSign: true,
						score: this.userInfo.score + 1,
						day: this.userInfo.day + 1
					},
	                success: function (res) {
	                    console.log(res.data);
						if (res.data.code == '000000'){
							uni.showToast({
								title:'已签，积分+1'
							});
							setTimeout(function(){
								uni.reLaunch({
									url: 'user'
								})
							},500)
						}
	                }
				});
			},
			toLuckyDraw(){
				uni.navigateTo({
					url: '../luckyDraw/luckyDraw'
				})
			},
			toAnalyze(){
				uni.navigateTo({
					url: './analyze'
				})
			},
			toZFB(){
				if ( plus.os.name == "Android" ) {
					plus.runtime.launchApplication( {pname:"com.eg.android.AlipayGphone"
						}, function ( e ) {
							alert( "Open system default browser failed: " + e.message );
					} );
				}
			},
			toWeiXin(){
				if ( plus.os.name == "Android" ) {
					plus.runtime.launchApplication( {pname:"com.tencent.mm"
						}, function ( e ) {
							alert( "Open system default browser failed: " + e.message );
					} );
				}
			},
			toMyMesg(){
				uni.navigateTo({
					url:'./mesg'
				})
				
			},
			bindLogout(){
				uni.request({
					url: app.globalData.mainUrl + "user/logout",
	                method:'POST',
					header:{
						'Authorization' : uni.getStorageSync('token')
					},
	                success: function (res) {
	                    console.log(res.data);
						if (res.data.code == '000000'){
							uni.removeStorageSync('token');
							uni.reLaunch({
								url: '../login/login'
							});
						}
	                }
				});
			},
			getUserInfo(e){
				uni.request({
					url: app.globalData.mainUrl + "user/detail",
	                method:'GET',
					header:{
						'Authorization' : uni.getStorageSync('token')
					},
	                success: function (res) {
	                    console.log(res.data);
						if (res.data.code == '000000'){
							if (null != res.data.data.imageUrl){
								e.url = res.data.data.imageUrl;
							}
							e.name = res.data.data.name;
							e.phone = res.data.data.phone;
							e.current = res.data.data.gender;
							if (null != res.data.data.description){
								e.description = res.data.data.description;
							}
							e.day = res.data.data.day;
							e.score = res.data.data.score;
							e.money = res.data.data.money;
							e.sign = res.data.data.isSign;
						}
	                }
				});
			},
		},
		created() {
			this.getUserInfo(this.userInfo);
		}
	}
</script>

<style lang="scss">
	page{
		background-color:#f1f1f1;
		font-size: 30upx;
	}
	.header{
		background: #fff;
		height: 360upx;
		padding-bottom: 110upx;
		.bg{
			width: 100%;
			height:250upx;
			padding-top:100upx;
			background-color:#4191ea;
		}
	}
	.box{
		width: 650upx;
		height: 300upx;
		border-radius: 20upx;
		margin: 0 auto;
		margin-top: 50upx;
		background: #fff;
		box-shadow: 0 5upx 20upx 0upx rgba(0, 0, 150, .2); 
		.box-hd{
			display: flex;
			flex-wrap: wrap;
			flex-direction: row;
			justify-content: center;
			.avator{
				width: 160upx;
				height: 160upx;
				background: #fff;
				border: 5upx solid #fff;
				border-radius: 50%;
				margin-top: -80upx;
				overflow: hidden;
				img{
					width: 100%;
					height: 100%;
				}
			}
			.phone-number{
				width: 100%;
				text-align: right;
			}
		}
		.box-bd{
			display: flex;
			flex-wrap: nowrap;
			flex-direction: row;
			justify-content: center;
			.item{
				flex: 1 1 auto;
				display: flex;
				flex-wrap: wrap;
				flex-direction: row;
				justify-content: center;
				border-right: 1px solid #f1f1f1;
				margin: 15upx 0;
				&:last-child{
					border: none;
				}
				.icon{
					text-align: center;
					width: 100%;
					//width: 60upx;
					height: 60upx;
				}
				.text{
					width: 100%;
					text-align: center;
					margin-top: 10upx;
				}
			}
		}
	}
	.list-content{
		background: #fff;
		margin-top: 20upx;
	}
	.list{
		width:100%;
		border-bottom:15upx solid  #f1f1f1;
		background: #fff;
		&:last-child{
			border: none;
		}
		.li{
			width:92%;
			height:100upx;
			padding:0 4%;
			border-bottom:1px solid rgb(243,243,243);
			display:flex;
			align-items:center;
		&.noborder{
			border-bottom:0
			}
			.icon{
				flex-shrink:0;
				width:50upx;
				height:50upx;
				image{
					width:50upx;
					height:50upx;
				}
			}
			.text{
				padding-left:20upx;
				width:100%;
				color:#666;
			}
			.to{
				flex-shrink:0;
				width:40upx;
				height:40upx;
			}
		}
	}
	.btn-row{
		margin-top: 50upx;
		button{
			background-color: #FFFFFF;
			color: #007AFF;
		}
	}
</style>
