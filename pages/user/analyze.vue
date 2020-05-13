<template>
	<view class="list-content">
		<view class="cu-bar bg-white margin-top solid-bottom">
			<view class="action" style="width: 100%;">
				<image src="../../static/cate/fanhui.png" style="width: 50upx; height: 40upx;" @click="fanhui"></image>
				<text style="margin-left: 33%; width: 80%; font-size: large;">收支分析</text> 
			</view>
		</view>
		
		<view class="list">
			<view class="li" >
				<view class="text">本月收入</view>
				<view class="right-text">
					{{this.income}}
				</view>
			</view>
			<view class="li" >
				<view class="text">本月支出</view>
				<view class="right-text">
					{{this.spending}}
				</view>
			</view>
			<view class="li" >
				<view class="text">本月结余</view>
				<view class="right-text">
					{{this.surplus}}
				</view>
			</view>
			<view class="li" >
				<view class="text">平均支出</view>
				<view class="right-text">
					{{this.spendingAvg}}&nbsp;/&nbsp;天
				</view>
			</view>		
		</view>
		
		<view style="margin: 50upx;">收支折线图</view>
		<view class="qiun-charts" >
			<!--#ifdef MP-ALIPAY -->
			<canvas canvas-id="canvasLineA" id="canvasLineA" class="charts" :width="cWidth*pixelRatio" :height="cHeight*pixelRatio" :style="{'width':cWidth+'px','height':cHeight+'px'}" @touchstart="touchLineA" @touchmove="moveLineA" @touchend="touchEndLineA"></canvas>
			<!--#endif-->
			<!--#ifndef MP-ALIPAY -->
			<canvas canvas-id="canvasLineA" id="canvasLineA" class="charts" @touchstart="touchLineA" @touchmove="moveLineA" @touchend="touchEndLineA"></canvas>
			<!--#endif-->
		</view>
			
		<view style="margin: 50upx;">本月支出排行</view>
		<view>
			<view v-for="(item,index) in rank" :index="index" :key="index">
				<view class="li " >
					<view class="text">
						<image class="image" :src="item.imageUrl"></image>&nbsp;{{item.name}}&nbsp;{{item.proportion}}%
					</view>
					<view class="right-text">
						{{item.spending}}
					</view>
				</view>	
			</view>
		</view>
		
	</view>
</template>

<script>
	const app = getApp()
	import uCharts from '@/components/u-charts/u-charts.js';
	import  { isJSON } from '@/util/checker.js';
	var _self;
	var canvaLineA=null;
	var lastMoveTime=null;//最后执行移动的时间戳
	export default {
		data() {
			return {
				income: 0,
				spending: 0,
				surplus: 0,
				spendingAvg: 0,
				rank: [],
				
				cWidth:'',
				cHeight:'',
				pixelRatio:1,
				textarea:'',
				Interactive:'',//交互显示的数据
			}
		},
		onLoad() {
			_self = this;
			//#ifdef MP-ALIPAY
			uni.getSystemInfo({
				success: function (res) {
					if(res.pixelRatio>1){
						//正常这里给2就行，如果pixelRatio=3性能会降低一点
						//_self.pixelRatio =res.pixelRatio;
						_self.pixelRatio =2;
					}
				}
			});
			//#endif
			this.cWidth=uni.upx2px(750);
			this.cHeight=uni.upx2px(500);
			this.getServerData();
		},
		methods:{
			fanhui(){
				uni.navigateBack({
					delta: 1
				});
			},
			getAnalyze(){
				uni.request({
					method: "GET",
					url: app.globalData.mainUrl + 'user/account/analyze',
					header:{
						'Authorization' : uni.getStorageSync('token')
					},
					success: res =>{
						console.log(res);
						if (res.data.code == '000000'){
							this.income = res.data.data.income;
							this.spending = res.data.data.spending;
							this.surplus = res.data.data.surplus;
							this.spendingAvg = res.data.data.spendingAvg;
							this.rank = res.data.data.rank;
						}
					}
				});
			},
			getServerData(){
				// let LineA={
				//   "categories": ["5.1", "5.2", "5.3", "5.4", "5.5", "5.6"],
				//   "series": [{
				// 	"name": "收入",
				// 	"data": [101.88, 8, 25, 37, 4, 20]
				//   }, {
				// 	"name": "支出",
				// 	"data": [70, 40, 65, 100, 44, 68]
				//   }, {
				// 	"name": "结余",
				// 	"data": [-100, 80, 95, 150, 112, 132]
				//   }]
				// };
				
				// //第二根线为虚线的设置
				// LineA.series[1].lineType='dash';
				// LineA.series[1].dashLength=10;
				// _self.textarea = JSON.stringify(LineA);
				// _self.showLineA("canvasLineA",LineA);
				uni.request({
					method: "GET",
					url: app.globalData.mainUrl + 'user/account/getLineData',
					header:{
						'Authorization' : uni.getStorageSync('token')
					},
					success: function(res) {
						console.log(res.data.data)
						let LineA={categories:[],series:[]};
						//这里我后台返回的是数组，所以用等于，如果您后台返回的是单条数据，需要push进去
						LineA.categories=res.data.data.categories;
						LineA.series=res.data.data.series;
						
						//第二根线为虚线的设置
						LineA.series[1].lineType='dash';
						LineA.series[1].dashLength=10;
						_self.textarea = JSON.stringify(res.data.data.LineA);
						_self.showLineA("canvasLineA",LineA);
					},
					fail: () => {
						_self.tips="网络错误，小程序端请检查合法域名";
					},
				});
			},
			showLineA(canvasId,chartData){
				canvaLineA=new uCharts({
					$this:_self,
					canvasId: canvasId,
					type: 'line',
					colors:['#facc14', '#f04864', '#8543e0', '#90ed7d'],
					fontSize:11,
					padding:[15,15,0,15],
					legend:{
						show:true,
						padding:5,
						lineHeight:11,
						margin:0,
					},
					dataLabel:false,
					dataPointShape:true,
					background:'#FFFFFF',
					pixelRatio:_self.pixelRatio,
					categories: chartData.categories,
					series: chartData.series,
					animation: true,
					xAxis: {
						type:'grid',
						gridColor:'#CCCCCC',
						gridType:'dash',
						dashLength:8,
						//disableGrid:true
					},
					yAxis: {
						gridType:'dash',
						gridColor:'#CCCCCC',
						dashLength:8,
					},
					width: _self.cWidth*_self.pixelRatio,
					height: _self.cHeight*_self.pixelRatio,
					extra: {
						line:{
							type: 'straight'
						}
					}
				});
				
			},
			touchLineA(e) {
				lastMoveTime=Date.now();
			},
			moveLineA(e){
				let currMoveTime = Date.now();
				let duration = currMoveTime - lastMoveTime;
				if (duration < Math.floor(1000 / 60)) return;//每秒60帧
				lastMoveTime = currMoveTime;
				
				let currIndex=canvaLineA.getCurrentDataIndex(e);
				if(currIndex>-1&&currIndex<canvaLineA.opts.categories.length){
					let riqi=canvaLineA.opts.categories[currIndex];
					let leibie=canvaLineA.opts.series[0].name;
					let shuju=canvaLineA.opts.series[0].data[currIndex];
					this.Interactive=leibie+':'+riqi+'-'+shuju+'元';
				}
				canvaLineA.showToolTip(e, {
					format: function (item, category) {
						return category + ' ' + item.name + ':' + item.data 
					}
				});
			},
			touchEndLineA(e){
				let currIndex=canvaLineA.getCurrentDataIndex(e);
				if(currIndex>-1&&currIndex<canvaLineA.opts.categories.length){
					let riqi=canvaLineA.opts.categories[currIndex];
					let leibie=canvaLineA.opts.series[0].name;
					let shuju=canvaLineA.opts.series[0].data[currIndex];
					this.Interactive=leibie+':'+riqi+'-'+shuju+'元';
				}
				canvaLineA.touchLegend(e);
				canvaLineA.showToolTip(e, {
					format: function (item, category) {
						return category + ' ' + item.name + ':' + item.data 
					}
				});
			},
			changeData(){
				if(isJSON(_self.textarea)){
					let newdata=JSON.parse(_self.textarea);
					canvaLineA.updateData({
						series: newdata.series,
						categories: newdata.categories
					});
				}else{
					uni.showToast({
						title:'数据格式错误',
						image:'../../../static/images/alert-warning.png'
					})
				}
			}
		},
		created() {
			this.getAnalyze();
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
		width:50%;
		color:#666;
		font-size: medium;
	}
	.right-text{
		width: 50%; 
		padding-right: 20upx; 
		text-align: right;
		font-size: medium;
		color:#666;
	}
	.image{
		width: 80upx;
		height: 80upx;
	}
	
	/*样式的width和height一定要与定义的cWidth和cHeight相对应*/
	.qiun-charts {
		width: 750upx;
		height: 500upx;
		background-color: #FFFFFF;
	}
	
	.charts {
		width: 750upx;
		height: 500upx;
		background-color: #FFFFFF;
	}
</style>
