<template>
	<scroll-view scroll-x class="bg-green nav text-center">
		<view class="cu-item" :class="0==TabCur?'text-white cur':''" @tap="tabSelect" data-id="0">
			<text class="cuIcon-camerafill"></text> 数码
		</view>
		<view class="cu-item" :class="1==TabCur?'text-white cur':''" @tap="tabSelect" data-id="1">
			<text class="cuIcon-upstagefill"></text> 排行榜
		</view>
		<view class="cu-item" :class="2==TabCur?'text-white cur':''" @tap="tabSelect" data-id="2">
			<text class="cuIcon-clothesfill"></text> 皮肤
		</view>
	</scroll-view>
</template>

<script>
	
</script>

<style lang='scss'>	
	@import "../../colorui/main.css";
	@import "../../colorui/icon.css";
	
	
</style>
