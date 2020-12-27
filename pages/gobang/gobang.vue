<template>
	<view class="gobang-content">
		<view>
			<h3 id="result-wrap">{{resultTxt}}</h3>
			<canvas canvas-id="gobang" id="gobang" class="gobang-canvas" @error="canvasIdErrorCallback" @click="heroOn"></canvas>
		</view>

		<view class="handle">
			<button class="handle-btn" @click="restart">重新开始</button>
			<button class="handle-btn" :disabled="backAble" @click="goback">悔棋</button>
			<button class="handle-btn" :disabled="returnAble" @click="goreturn">撤销</button>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			resultTxt: 'gobang',
			context: uni.createCanvasContext('gobang'),
			over: false, // 游戏结束
			hero: true, // 玩家
			_nowi: 0, // 玩家下起横坐标
			_nowj: 0, // 玩家下起纵坐标
			_compi: 0, // 计算机下起横坐标
			_compj: 0, // 计算机下起纵坐标
			_myWin: [], // 玩家赢的情况
			_compWin: [], // 计算机赢的情况
			backAble: true, // 悔棋按钮
			returnAble: true, // 撤销按钮
			chressBord: [],	// 棋盘
			myWin: [],	// 玩家赢法的统计数组
			computerWin: [],	// 计算机赢法的统计数组
			wins: [],	// 赢法数组
			count: 0,	// 赢法总数
			heroArr: [], // 黑子坐标集合
			compArr: [], // 白子坐标集合
		}
	},
	onReady: function (e) {
		this.drawChessBoard();
		this.winState();
	},
	methods: {
		canvasIdErrorCallback (e) {
			console.error(e.detail.errMsg);
		},
		//赢法的统计数组
		winState(){
			for(var i = 0; i < 15; i++){
				this.wins[i] = [];
				for(var j = 0; j < 15; j++){
					this.wins[i][j] = [];
				}
			}
			//横线赢法
			for(var i = 0; i < 15; i++){
				for(var j = 0; j < 11; j++){
					for(var k = 0; k < 5; k++){
						this.wins[i][j+k][this.count] = true;
					}
					this.count++;
				}
			}
			//竖线赢法
			for(var i = 0; i < 15; i++){
				for(var j = 0; j < 11; j++){
					for(var k = 0; k < 5; k++){
						this.wins[j+k][i][this.count] = true;
					}
					this.count++;
				}
			}
			//正斜线赢法
			for(var i = 0; i < 11; i++){
				for(var j = 0; j < 11; j++){
					for(var k = 0; k < 5; k++){
						this.wins[i+k][j+k][this.count] = true;
					}
					this.count++;
				}
			}
			//反斜线赢法
			for(var i = 0; i < 11; i++){ 
				for(var j = 14; j > 3; j--){
					for(var k = 0; k < 5; k++){
						this.wins[i+k][j-k][this.count] = true;
					}
					this.count++;
				}
			}
			this.myWin = new Array(this.count).fill(0)
			this._myWin = new Array(this.count).fill(0)
			this.computerWin = new Array(this.count).fill(0)
			this._compWin = new Array(this.count).fill(0)
			for(var i = 0; i < this.count; i++){
				this.myWin[i] = 0;
				this._myWin[i] = 0;
				this.computerWin[i] = 0;
				this._compWin[i] = 0;
			}
		},
		initChessBoard() {
			let context = this.context;
			context.strokeStyle = '#bfbfbf'; //边框颜色
			for(let i = 0; i < 15; i++){
				context.moveTo(15 + i * 25 , 15);
				context.lineTo(15 + i * 25 , 365);
				context.stroke();
				context.moveTo(15 , 15 + i * 25);
				context.lineTo(365 , 15 + i * 25);
				context.stroke();
			}
		},
		// 画棋盘
		drawChessBoard() {
			this.initChessBoard();
			this.context.draw();

			// 棋盘数组置为0的二维数组
			for(let i = 0; i < 15; i++){
				this.chressBord[i] = [];
				for(let j = 0; j < 15; j++){
					this.chressBord[i][j] = 0;
				}
			}
		},
		// 画棋子
		oneStep(i,j,hero) {
			var temp = {
				i: i,
				j: j
			}
			if(hero){
				this.heroArr.push(temp)
			}else{
				this.compArr.push(temp)
			}
			this.initChessBoard();
			
			for(let m=0;m<this.heroArr.length;m++){
				this.context.beginPath();
				this.context.arc(15 + this.heroArr[m].i * 25, -10 + this.heroArr[m].j * 25, 13, 0, 2 * Math.PI);
				this.context.fillStyle = '#0a0a0a'; //黑子
				this.context.closePath();
				this.context.fill();
			}
			for(let n=0;n<this.compArr.length;n++){
				this.context.beginPath();
				this.context.arc(15 + this.compArr[n].i * 25, -10 + this.compArr[n].j * 25, 13, 0, 2 * Math.PI);
				this.context.fillStyle = '#d1d1d1'; //白子
				this.context.closePath();
				this.context.fill();
			}
			// 重新渲染
			this.context.draw()
		},
		// 玩家下棋
		heroOn(e){
			if(this.over){
				return;
			}
			if(!this.hero){
				return;
			}
			
			var x = e.offsetX || e.target.x;
			var y = e.offsetY || e.target.y;
			var i = Math.floor(x / 25);
			var j = Math.floor(y / 25);
			this._nowi = i;
			this._nowj = j;
			
			if(this.chressBord[i][j] == 0){
				this.oneStep(i,j,'hero');
				this.chressBord[i][j] = 1; //我，已占位置        
										
				for(var k = 0; k < this.count; k++){ // 将可能赢的情况都加1
					if(this.wins[i][j][k]){
						this.myWin[k]++;
						this._compWin[k] = this.computerWin[k];
						this.computerWin[k] = 6;//这个位置对方不可能赢了
						if(this.myWin[k] == 5){
							this.resultTxt = '恭喜，你赢了！';
							this.over = true;
						}
					}
				}
				if(!this.over){
					this.hero = !this.hero;
					setTimeout(()=>{
						this.computerAI();
					}, 500);
				}
			}
		},
		// 计算机下棋
		computerAI(){
			var myScore = [];
			var computerScore = [];
			var max = 0;
			var u = 0, v = 0;
			for(var i = 0; i < 15; i++){
				myScore[i] = [];
				computerScore[i] = [];
				for(var j = 0; j < 15; j++){
					myScore[i][j] = 0;
					computerScore[i][j] = 0;
				}
			}
			
			for(var i = 0; i < 15; i++){
				for(var j = 0; j < 15; j++){
					if(this.chressBord[i][j] == 0){
						for(var k = 0; k < this.count; k++){
							if(this.wins[i][j][k]){
								if(this.myWin[k] == 1){
									myScore[i][j] += 200;
								}else if(this.myWin[k] == 2){
									myScore[i][j] += 400;
								}else if(this.myWin[k] == 3){
									myScore[i][j] += 2000;
								}else if(this.myWin[k] == 4){
									myScore[i][j] += 10000;
								}
								
								if(this.computerWin[k] == 1){
									computerScore[i][j] += 220;
								}else if(this.computerWin[k] == 2){
									computerScore[i][j] += 420;
								}else if(this.computerWin[k] == 3){
									computerScore[i][j] += 2100;
								}else if(this.computerWin[k] == 4){
									computerScore[i][j] += 20000;
								}                     
							}
						}
						if(myScore[i][j] > max){
							max  = myScore[i][j];
							u = i;
							v = j;
						}else if(myScore[i][j] == max){
							if(computerScore[i][j] > computerScore[u][v]){
								u = i;
								v = j;    
							}
						}
						
						if(computerScore[i][j] > max){
							max  = computerScore[i][j];
							u = i;
							v = j;
						}else if(computerScore[i][j] == max){
							if(myScore[i][j] > myScore[u][v]){
								u = i;
								v = j;    
							}
						}
					}
				}
			}
			this._compi = u;
			this._compj = v;
			
			this.oneStep(u,v,false);
			this.chressBord[u][v] = 2;  //计算机占据位置
			for(var k = 0; k < this.count; k++){
				if(this.wins[u][v][k]){
					this.computerWin[k]++;
					this._myWin[k] = this.myWin[k];
					this.myWin[k] = 6;//这个位置对方不可能赢了
					if(this.computerWin[k] == 5){
						this.resultTxt = 'o(╯□╰)o，计算机赢了，继续加油哦！';
						this.over = true;
					}
				}
			}
			if(!this.over){
				this.hero = !this.hero;
			}
			this.backAble = false;
			this.returnAble = true;
		},
		//销毁棋子
		minusStep() {
			this.initChessBoard();
			this.heroArr.pop();
			this.compArr.pop();
			for(let m=0;m<this.heroArr.length;m++){
				this.context.beginPath();
				this.context.arc(15 + this.heroArr[m].i * 25, -10 + this.heroArr[m].j * 25, 13, 0, 2 * Math.PI);
				this.context.fillStyle = '#0a0a0a'; //黑子
				this.context.closePath();
				this.context.fill();
			}
			for(let n=0;n<this.compArr.length;n++){
				this.context.beginPath();
				this.context.arc(15 + this.compArr[n].i * 25, -10 + this.compArr[n].j * 25, 13, 0, 2 * Math.PI);
				this.context.fillStyle = '#d1d1d1'; //白子
				this.context.closePath();
				this.context.fill();
			}
			// 重新渲染
			this.context.draw()
		},
		// 重新开始
		restart () {
			// 重新加载页面
			uni.navigateTo({
				url: '/pages/gobang/gobang'
			});
		},
		// 悔棋
		goback () {
			console.log('悔棋',this.backAble)
			if(this.backAble) { return;}
			this.over = false;
			this.hero = true;
			this.resultTxt = 'o(╯□╰)o，悔棋中';
			
			// 我，悔棋
			this.chressBord[this._nowi][this._nowj] = 0; //我，已占位置 还原
			console.log(this.myWin)
			for(var k = 0; k < this.count; k++){ // 将可能赢的情况都减1
				if(this.wins[this._nowi][this._nowj][k]){
					this.myWin[k]--;
					this.computerWin[k] = 0;//这个位置对方可能赢
				}
			}
			// 计算机相应的悔棋
			this.chressBord[this._compi][this._compj] = 0; //计算机，已占位置 还原
			this.minusStep(); //销毁棋子                                  
			for(var k = 0; k < this.count; k++){ // 将可能赢的情况都减1
					if(this.wins[this._compi][this._compj][k]){
							this.computerWin[k]--;
							this.myWin[k] = 0;//这个位置对方可能赢
					}
			}
			this.resultTxt = '--益智五子棋--';
			this.backAble = true;
			this.returnAble = false;
		},
		// 撤销
		goreturn () {
			console.log('撤销')
			if(this.returnAble) { return; }
			// 我，撤销悔棋
			this.chressBord[this._nowi][this._nowj] = 1; //我，已占位置 
			this.oneStep(this._nowi,this._nowj,'hero');                           
			for(var k = 0; k < this.count; k++){ 
				if(this.wins[this._nowi][this._nowj][k]){
					this.myWin[k]++;
					this._compWin[k] = this.computerWin[k];
					this.computerWin[k] = 6;//这个位置对方不可能赢
				}
				if(this.myWin[k] == 5){
					this.resultTxt = '恭喜，你赢了！';
					this.over = true;
				}
			}
			// 计算机撤销相应的悔棋
			this.chressBord[this._compi][this._compj] = 2; //计算机，已占位置   
			this.oneStep(this._compi,this._compj,false)                           
			for(var k = 0; k < this.count; k++){ // 将可能赢的情况都减1
				if(this.wins[this._compi][this._compj][k]){
					this.computerWin[k]++;
					this._myWin[k] = this.myWin[k];
					this.myWin[k] = 6;//这个位置对方不可能赢
				}
				if(this.computerWin[k] == 5){
					this.resultTxt = 'o(╯□╰)o，计算机赢了，继续加油哦！';
					this.over = true;
				}
			}
			this.backAble = false;
			this.returnAble = true;
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
.gobang-content .gobang-canvas {
	width: 400px;
	height: 400px;
}

canvas {
	display: block;
	margin: 0 auto;
	box-shadow: -2px -2px 2px #efefef, 5px 5px 5px #b9b9b9;
	cursor: pointer;
}
.btn-wrap { 
	display: flex; 
	flex-direction: row; 
	justify-content:center;
}
.btn-wrap div { 
	margin: 0 10px;
}
div>span {
	display: inline-block;
	padding: 10px 20px;
	color: #fff;
	background-color: #EE82EE;
	border-radius: 5px;
	cursor: pointer;
}
div.unable span { 
	background: #D6D6D4; 
	color: #adacaa;
}
#result-wrap {text-align: center;}
</style>
