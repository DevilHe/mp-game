<template>
	<view class="content">
		<view class="text-area">
			<text class="title">{{title}}</text>
		</view>

		<view style="width: 100%;padding:10px 20px;">
			<button style="float: left;width: 40%;margin: 12px 5%;" @click="btnGobang">五子棋</button>
			<button style="float: left;width: 40%;margin: 12px 5%;" @click="btnSudoku">数独</button>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: '请选择'
			}
		},
		onLoad() {

		},
		methods: {
			// 五子棋
			btnGobang() {
				uni.navigateTo({
					url: '/pages/gobang/gobang'
				})
			},
			// 数独
			btnSudoku() {
				// uni.navigateTo({
				// 	url: '/pages//'
				// })
			}
		}
	}
</script>

<style>
.content {
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
}
.text-area {
	display: flex;
	justify-content: center;
	margin: 12px auto;
}
.title {
	font-size: 36rpx;
	color: #8f8f94;
}
	

	
</style>
