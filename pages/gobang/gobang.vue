<template>
	<view class="gobang-content">
		<view>
			<canvas canvas-id="gobang" id="gobang" width="450px" height="450px" @error="canvasIdErrorCallback"></canvas>
		</view>

		<view class="handle">
			<button class="handle-btn" @click="restart">重新开始</button>
			<button class="handle-btn" @click="goback">悔棋</button>
			<button class="handle-btn" @click="goreturn">撤销</button>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				
			}
		},
		methods: {
			canvasIdErrorCallback (e) {
        console.error(e.detail.errMsg)
			},
			// 重新开始
			restart () {
        console.log('重新开始')
			},
			// 悔棋
			goback () {
        console.log('悔棋')
      },
			goreturn () {
        console.log('撤销')
      }
		}
	}
</script>

<style>
.gobang-content .handle {
	width: 100%;
	padding: 10px 20px;
}
.gobang-content .handle .handle-btn {
	float: left;
	padding: 0 20px;
	margin: 12px 2%;
}
</style>
