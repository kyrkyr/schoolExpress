<template>
	<view class="list-content">
		<view class="cu-bar bg-white margin-top solid-bottom">
			<view class="action" style="width: 100%;">
				<image src="../../static/cate/fanhui.png" style="width: 50upx; height: 40upx;" @click="fanhui"></image>
				<text style="margin-left: 33%; width: 50%; font-size: large;">我的信息</text> 
				<text class="text-blue" style="text-align: right; width: 30%; font-size: large;" @click="edit">编辑</text>
			</view>
		</view>
		
		<view class="list">
			<view class="li" style="margin-top: 30upx;">
				<view class="text">头像</view>
				<!--<view class="right-text">
					<image style="width: 120upx;height: 120upx;" src="../../static/face.jpg"></image>
				</view>-->
				<view v-if="isEdit == false" class="right-text">
					<image style="width: 120upx;height: 120upx;" :src="userInfo.url" mode="" v-bind:id="userInfo.url"></image>
				</view>
				<view v-if="isEdit" class="right-text" @click="upload">
					<image style="width: 120upx;height: 120upx;" :src="userInfo.url" mode="" v-bind:id="userInfo.url"></image>
				</view>
			</view>
			<view class="li" >
				<view class="text">姓名</view>
				<view v-if="isEdit == false" class="right-text">
					{{userInfo.name}}
				</view>
				<view v-if="isEdit" class="right-text">
					<input type="text" v-model="userInfo.name" style="color: #000;" />
				</view>
			</view>
			<view class="li" >
				<view class="text">手机号</view>
				<view class="right-text">
					{{userInfo.phone}}
				</view>
			</view>
			<view class="li" >
				<view class="text">性别</view>
				<view v-if="isEdit == false" class="right-text">
					{{genderList[this.userInfo.current]}}
				</view>
				<view v-if="isEdit" class="right-text">
					<radio-group @change="radioChange">
						<label class="radio" v-for="(item, index) in genderList" :key="item">
							<radio :value="item" :checked="index === userInfo.current" />{{item}}
						</label>
					</radio-group>
				</view>
			</view>
			<view class="li" >
				<view class="text">描述</view>
				<view v-if="isEdit == false" class="right-text">
					{{userInfo.description}}
				</view>
				<view v-if="isEdit" class="right-text">
					<input type="text" v-model="userInfo.description" style="color: #000;" />
				</view>
			</view>
			<view class="li" >
				<view class="text">记账天数</view>
				<view class="right-text">
					{{userInfo.day}}
				</view>
			</view>
			<view class="li" >
				<view class="text">积分</view>
				<view class="right-text">
					{{userInfo.score}}
				</view>
			</view>
			<view class="li" >
				<view class="text">虚拟币</view>
				<view class="right-text">
					{{userInfo.money}}
				</view>
			</view>
		</view>
		
		<view v-if="isEdit">
			<button type="default" @tap="save" style="color: #007AFF;">保存</button>
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
					current: 2
				},
				isEdit: false,
				genderList: ['男','女','未知'],
				iconcheck:0, //头像是否改变
				//image:'_doc/uniapp_temp_1584866317665/compressed/1584866322712.jpg', //默认头像
			}
		},
		methods:{
			upload(){
				let _self = this;
				uni.chooseImage({
					count: 1,
					sizeType: ['compressed'], //可以指定是原图还是压缩图，默认二者都有
					sourceType: ['album'], //从相册选择
					success: function (res) {
						const tempFilePaths = res.tempFilePaths;
						_self.userInfo.url = tempFilePaths[0];
						console.log(_self.userInfo.url);
						console.log("tempFilePaths[0]",tempFilePaths[0])  //能够打印出选中的图片
						_self.iconcheck = 1;//点击后图片更改状态由0变成1
						// uni.previewImage({
						// 	urls: tempFilePaths
						// });
						// uni.uploadFile({
						// 	url: app.globalData.mainUrl + 'user/uploadImg', //仅为示例，非真实的接口地址
						// 	filePath: tempFilePaths[0],
						// 	name: 'file',
						// 	header:{"Content-Type": "multipart/form-data","Authorization":"eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOjgsImV4cCI6MTU4NjkzNzQ3NH0.yvDCsLI1k-RWxYcbrF_raLwiOTy3oWzQRGPZFptfRZOAp-Eg2RzUpxg_QRG0TP3qF9YBHS50ktq708N9KXu4NQ"},
						// 	success: (uploadFileRes) => {
						// 		console.log(uploadFileRes.data);
						// 		imageUrl = uploadFileRes.data.data;
						// 	},
						// 	complete() {
						// 		console.log("上传");
						// 	}
						// });
					}
				});
			},
			fanhui(){
				uni.navigateBack({
					delta: 1
				});
			},
			edit(){
				this.isEdit = true;
			},
			save(){
				if (this.iconcheck == 1){
					uni.uploadFile({
						url: app.globalData.mainUrl + 'user/uploadImg', //仅为示例，非真实的接口地址
						filePath: this.userInfo.url,
						name: 'file',
						header:{"Content-Type": "multipart/form-data","Authorization": uni.getStorageSync('token')},
						success: (uploadFileRes) => {
							console.log(uploadFileRes.data);
							let a = JSON.parse(uploadFileRes.data);
							uni.request({
								url: app.globalData.mainUrl + "user/edit",
				                method:'POST',
								header:{
									'Authorization' : uni.getStorageSync('token')
								},
								data:{
									imageUrl:a.data,
									name:this.userInfo.name,
									gender:this.userInfo.current,
									description:this.userInfo.description
								},
				                success: function (res) {
				                    console.log(res.data);
									if (res.data.code == '000000'){
										uni.showToast({
											title:'保存成功'
										});
									}
				                }
							});
						}
					});
				} else{
					uni.request({
						url: app.globalData.mainUrl + "user/edit",
		                method:'POST',
						header:{
							'Authorization' : uni.getStorageSync('token')
						},
						data:{
							name:this.userInfo.name,
							gender:this.userInfo.current,
							description:this.userInfo.description
						},
		                success: function (res) {
		                    console.log(res.data);
							if (res.data.code == '000000'){
								uni.showToast({
									title:'保存成功'
								});
							}
		                }
					});
				}
				this.iconcheck = 0;
				this.isEdit = false;
			},
			radioChange: function(evt) {
				for (let i = 0; i < this.genderList.length; i++) {
					if (this.genderList[i] === evt.target.value) {
						this.userInfo.current = i;
						break;
					}
				}
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
							
						}
	                }
				});
			}
		},
		created() {
			this.getUserInfo(this.userInfo);
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
