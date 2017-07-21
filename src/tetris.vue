<style scoped>
#wrap {
	width: 100%;
	text-align: center;
}
.wrap-ul {
	display: inline-block;
}
ul {
	font-size: 0;
	padding-left: 0;
}
li {
	list-style: none;
}
.class-box {
	display: inline-block;
	width: 28px;
	height: 28px;
	line-height: 28px;
	margin: 1px;
	text-align: center;
	font-size: 12px;
}
.color-normal {
	background: #eee;
}
.color-blank {
	background: #000;
}
</style>
<template>
	<div id="wrap">
		<ul class="wrap-ul">
			<li v-for="rows in tetrisArray">
				<ul>
					<li v-for="col in rows" class="class-box color-normal" :class="[col.active || col.static ? colorClass : '']">
					</li>
				</ul>
			</li>
		</ul>
	</div>
</template>
<script>
export default {
	data: function () {
		return {
			shapesArray: [
				[
					[0,0,0,0],
					[1,1,1,1],
					[0,0,0,0],
					[0,0,0,0]
				],
				[
					[0,1,0,0],
					[0,1,0,0],
					[0,1,1,0],
					[0,0,0,0]
				],
				[
					[0,0,1,0],
					[0,0,1,0],
					[0,1,1,0],
					[0,0,0,0]
				],
				[
					[0,0,0,0],
					[1,1,0,0],
					[0,1,1,0],
					[0,0,0,0]
				],
				[
					[0,0,0,0],
					[0,0,1,1],
					[0,1,1,0],
					[0,0,0,0]
				],
				[
					[0,0,0,0],
					[0,1,1,0],
					[0,1,1,0],
					[0,0,0,0]
				],
				[
					[0,0,1,0],
					[0,1,1,0],
					[0,0,1,0],
					[0,0,0,0]
				]
			],
			shapeArray: [],
			px: 0,
			py: 0,
			tetrisArray: [],
			colorClass: 'color-blank',
			colcount: 10,
			rowcount: 18,
			timeId1: '',
			timeId2: '',
			speed: 600
		}
	},
	mounted: function () {
		this.init();
	},
	methods: {
		// 随机生成一个俄罗斯方块数组
		randomShape: function () {
			this.shapeArray = this.shapesArray[parseInt(Math.random() * 7)];
		},
		// 初始化
		create: function () {
			if (!this.tetrisArray.length) {
				for (var i = 0; i < this.rowcount; i++) {
					var tempArray = [];
					for (var j = 0; j < this.colcount; j++) {
						tempArray.push({active: 0,static: 0});
					}
					this.tetrisArray.push(tempArray);
				}
			}
		},
		// 显示俄罗斯方块
		show: function () {
			for (var i = 0; i < this.tetrisArray.length; i ++) {
				for (var j = 0; j < this.tetrisArray[i].length; j++) {
					if (this.tetrisArray[i][j].active) {
						this.tetrisArray[i][j].active = 0;
					}
				}
			}
			for (var i = 0; i < this.shapeArray.length; i ++) {
				for (var j = 0; j < this.shapeArray[i].length; j++) {
					if (this.shapeArray[i][j]) {
						this.tetrisArray[this.py + i][this.px + j].active = 1;
					}
				}
			}
		},
		// 检测是否碰壁
		check: function (x, y, _shapeArray) {
			for (var i = 0; i < _shapeArray.length; i ++) {
				for (var j = 0; j < _shapeArray[i].length; j++) {
					if (_shapeArray[i][j]) {
						if (y + i > this.rowcount - 1 || x + j > this.colcount - 1 || x + j < 0 || this.tetrisArray[y + i][x + j].static) return false;
					}
				}
			}
			return true;
		},
		//  删除某行
		delete: function (line) {
			for ( var j = 0; j < this.tetrisArray[line].length; j++) {
				this.tetrisArray[line][j].static = 0;
			}
			for (var i = line; i > 0; i --) {
				for (var j = 0; j < this.tetrisArray[i].length; j++) {
					this.tetrisArray[i][j].static = this.tetrisArray[i - 1][j].static;
				}
			}
		},
		//  检测是否满行
		preDelete: function () {
			for (var i = 0; i < this.tetrisArray.length; i ++) {
				var count = 0;
				for (var j = 0; j < this.tetrisArray[i].length; j++) {
					if (!this.tetrisArray[i][j].static) {
						break;
					}
					count++;
				}
				if (count == this.tetrisArray[i].length) {
					this.delete(i);
				}
			}
		},
		// 固定方块
		fix: function () {
			for (var i = 0; i < this.tetrisArray.length; i ++) {
				for (var j = 0; j < this.tetrisArray[i].length; j++) {
					if (this.tetrisArray[i][j].active) {
						this.tetrisArray[i][j].active = 0;
						this.tetrisArray[i][j].static = 1;
					}
				}
			}
			this.preDelete();
		},
		// 移动方块
		move: function (a, b) {
			var _px = this.px + a;
			var _py = this.py + b;
			if (this.check(_px, _py, this.shapeArray)) {
				this.px = _px;
				this.py = _py;
				this.show();
			} else {
				if (b == 0) return;
				this.fix();
				this.px = 0;
				this.py = 0;
				this.randomShape();
				this.show();
				clearInterval(this.timeId2);
			}
		},
		// 旋转方块
		rotate: function () {
			var newShapeArray = [];
			for (var i = 0; i < this.shapeArray[0].length; i++) {
				var tempArray = [];
				for (var j = 0; j < this.shapeArray.length; j++) {
					tempArray.push(0);
				}
				newShapeArray.push(tempArray);
			}
			for (var i = 0; i < this.shapeArray.length; i++) {
				for (var j = 0; j < this.shapeArray[i].length; j++) {
					newShapeArray[this.shapeArray.length-1-j][i] = this.shapeArray[i][j];
				}
			}
			if (this.check(this.px, this.py, newShapeArray)) {
				this.shapeArray = newShapeArray;
				this.show();
			}
		},
		// 快降
		quickDown: function () {
			var that = this;
			this.timeId2 = setInterval(function () {
				that.move(0, 1);
			}, 0);
		},
		// 绑定事件
		bindEvents: function () {
			var that = this;
			document.onkeydown = function (e) {
				var e = e || window.event,
					code = e.code;
				switch (code) {
					case 'Space': that.quickDown(); break;
					case 'ArrowLeft': that.move(-1, 0); break;
					case 'ArrowUp': that.rotate(); break;
					case 'ArrowDown': that.move(0, 1); break;
					case 'ArrowRight': that.move(1, 0); break;
				}
			}
		},
		// 初始化
		init: function () {
			var that = this;
			this.randomShape();
			this.create();
			this.show();
			this.bindEvents();
			this.timeId1 = setInterval(function () {
				that.move(0, 1);
			}, this.speed);
		}
	}
}
</script>