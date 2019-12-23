<template>
	<div id="app">
		<!--<img alt="Vue logo" src="./assets/logo.png" />-->
		<HelloWorld msg="Sudoku v0.1" />
		<div class="hello">
			<h3>#{{ sudokuid }}</h3>
		</div>
		<div class="game">
			<div class="sudoku">
				<template v-for="(area_row, index_area_row) in 3">
					<template v-for="(area_col, index_area_col) in 3">
						<div class="sudoku_area" :key="(index_area_row - 1) * 3 + (index_area_col -1 )">
							<template v-for="row in 3">
								<template v-for="col in 3">
									<cell :initcellnumber="sudoku_data[(area_row - 1) * 27 + (area_col - 1) * 3 + (row -1) * 9 + (col - 1)]" :key="(area_row -1 ) * 27 + (area_col -1 ) * 3 + (row - 1) * 9 + (col - 1)" :id="(area_row - 1) * 27 + (area_col - 1) * 3 + (row -1) * 9 + col" />
								</template>
							</template>
						</div>
					</template>
				</template>
			</div>
		</div>
		<div class="footer">
			<template v-if="cheat">
				<button @click="Cheat()">开始作弊</button>
			</template>
			<template v-else>
				<button @click="LockOnlyOne()">锁定唯一值</button>
				<button @click="DuplicateRemoval()">多值去重</button>
			</template>
			<template>
				<button @click="testAxios()">Axios功能测试</button>
			</template>
		</div>
	</div>
</template>

<script>
	import HelloWorld from "./components/HelloWorld.vue";
	import cell from "./components/cell.vue";

	export default {
		name: "app",
		data: function() {
			return {
				//初始值
				cheat: true,		//显示作弊按钮
				sudokuid: '',		//题号
				//sudoku_array: [],	//系统给出的初始数据
				
				sudoku_array: [
					0, 8, 7, 0, 0, 2, 0, 0, 5,
					0, 0, 3, 0, 0, 6, 0, 8, 0,
					1, 0, 4, 0, 7, 0, 0, 0, 9,
					0, 0, 2, 0, 0, 9, 0, 0, 3,
					0, 6, 0, 0, 0, 0, 0, 7, 0,
					5, 0, 0, 4, 0, 0, 8, 0, 0,
					9, 0, 0, 0, 5, 0, 6, 0, 2,
					0, 2, 0, 8, 0, 0, 4, 0, 0,
					7, 0, 0, 9, 0, 0, 5, 1, 0,
				],
				
				//成员为每个单元格对象
				//sudoku_data: [],

				//9个行对象
				sudoku_rows: [],

				//9个列对象
				sudoku_cols: [],

				//9个区域对象
				sudoku_areas: []
			};
		},
		
		computed:{
			sudoku_data:function(){
				console.log('调用initSudokuData方法');
				let _temp = [];
				//console.log(this.sudoku_array.length);
				for (let i = 0; i < this.sudoku_array.length; i++) {
					if (0 === this.sudoku_array[i]) {
						_temp.push({
							innerdata: [],
							system: false,
							userlocked: true,
							belongrow: {}, //单元格所属的行
							belongcol: {}, //单元格所属的列
							belongarea: {} //单元格所属的区域
						});
					} else {
						_temp.push({
							innerdata: [this.sudoku_array[i]],
							system: true,
							userlocked: false,
							belongrow: {}, //单元格所属的行
							belongcol: {}, //单元格所属的列
							belongarea: {} //单元格所属的区域
						})
					}
				}
				//console.log(_temp);
				return _temp;
			}
		},

		methods: {
			testAxios(){
				this.$axios.get('http://192.168.108.79:8983/cgi-bin/getdata.py')
				.then(response => {
					//console.log(response.request.response);
					//let res = response.request.response;
					console.log(response.data);
					this.sudokuid = response.data.sudokuid;
					this.sudoku_array.splice(0);
					this.sudoku_array.push(...response.data.sudokudata);
					console.log(this.sudoku_array);
					//根据初始值，生成81个单元格对象放入sudoku_data中
					//this.changeArray();
					//this.sudoku_array.splice(0);
					//this.sudoku_array.push(...this.sudoku_test);
					//console.log('2个数组的区别');
					//console.log(this.sudoku_test, this.sudoku_array);
					//console.log(this.sudoku_data);
				})
			},
			
			// initSudokuData() {
			// 	console.log('调用initSudokuData方法');
			// 	let _temp = [];
			// 	//console.log(this.sudoku_array.length);
			// 	for (let i = 0; i < this.sudoku_array.length; i++) {
			// 		if (0 === this.sudoku_array[i]) {
			// 			_temp.push({
			// 				innerdata: [],
			// 				system: false,
			// 				userlocked: true,
			// 				belongrow: {}, //单元格所属的行
			// 				belongcol: {}, //单元格所属的列
			// 				belongarea: {} //单元格所属的区域
			// 			});
			// 		} else {
			// 			_temp.push({
			// 				innerdata: [this.sudoku_array[i]],
			// 				system: true,
			// 				userlocked: false,
			// 				belongrow: {}, //单元格所属的行
			// 				belongcol: {}, //单元格所属的列
			// 				belongarea: {} //单元格所属的区域
			// 			})
			// 		}
			// 	}
			// 	//console.log(_temp);
			// 	this.sudoku_data = _temp;
			// },

			//初始化9个行对象
			initSudokuRows() {
				for (let row = 0; row < 9; row++) {
					let _tempRow = {
						locked: [], //该行已经被锁定的数据
						cells: [] //该行包含的单元格
					};
					for (let col = 0; col < 9; col++) {
						let _tempcell = this.sudoku_data[row * 9 + col]; //取待处理的单元格
						_tempRow.cells.push(_tempcell); //将单元格添加到【行】表中
						_tempcell.belongrow = _tempRow; //记录单元格的【行】归属信息
						_tempRow.locked.push(_tempcell.innerdata[0]); //将系统给定的单元格数值添到锁定数据中
					}
					//清理锁定数据数组中的undefined
					_tempRow.locked = _tempRow.locked.filter(Boolean);
					this.sudoku_rows.push(_tempRow);
				}
				//console.log(this.sudoku_rows);
				//设置已锁定数值
			},

			//初始化9个列对象
			initSudokuCols() {
				for (let col = 0; col < 9; col++) {
					let _tempCol = {
						locked: [], //该列已被锁定的数据
						cells: [] //该列包含的单元格
					};
					for (let row = 0; row < 9; row++) {
						let _tempcell = this.sudoku_data[row * 9 + col]; //取待处理的单元格
						_tempCol.cells.push(_tempcell); //将单元格添加到【列】表中
						_tempcell.belongcol = _tempCol; //记录单元格的【列】归属信息
						_tempCol.locked.push(_tempcell.innerdata[0]); //将系统给定的单元格数值添到锁定数据中
					}
					//清理锁定数据数组中的undefined
					_tempCol.locked = _tempCol.locked.filter(Boolean);
					this.sudoku_cols.push(_tempCol);
				}
				//console.log(this.sudoku_cols);
			},

			//初始化9个区域对象
			initSudokuAreas() {
				for (let area_row = 0; area_row < 3; area_row++)
					for (let area_col = 0; area_col < 3; area_col++) {
						let _tempArea = {
							locked: [], //该区域已被锁定的数据
							cells: [] //该区域包含的单元格
						};
						for (let row = 0; row < 3; row++)
							for (let col = 0; col < 3; col++) {
								let _tempcell = this.sudoku_data[area_row * 27 + area_col * 3 + row * 9 + col]; //取待处理的单元格
								_tempArea.cells.push(_tempcell); //将单元格添加到【区域】表中
								_tempcell.belongarea = _tempArea; //记录单元格的【区域】归属信息
								_tempArea.locked.push(_tempcell.innerdata[0]); //将系统给定的单元格数值添到锁定数据中
							}
						//清理锁定数据数组中的undefined
						_tempArea.locked = _tempArea.locked.filter(Boolean);
						this.sudoku_areas.push(_tempArea);
					}
				//console.log(this.sudoku_areas);
			},

			//开始游戏
			Cheat() {
				//进入作弊模式后，关闭作弊按钮，开始另外2个按钮
				this.cheat = false;
				
				//对第个单元格的行，列，区域进行查找已锁定的数据
				for (let i = 0; i < 81; i++) {
					// console.log('1号单元格已锁定的  行  值：',this.sudoku_data[i].belongrow.locked);
					// console.log('1号单元格已锁定的  列  值：',this.sudoku_data[i].belongcol.locked);
					// console.log('1号单元格已锁定的 区域 值：',this.sudoku_data[i].belongarea.locked);
					if (!this.sudoku_data[i].system) {
						//对用户单元格进行处理
						let _set_row = new Set(this.sudoku_data[i].belongrow.locked);
						let _set_col = new Set(this.sudoku_data[i].belongcol.locked);
						let _set_area = new Set(this.sudoku_data[i].belongarea.locked);
						let _set_locked = new Set([..._set_row, ..._set_col, ..._set_area]);
						let _set_unLocked = new Set([1, 2, 3, 4, 5, 6, 7, 8, 9].filter(x => !_set_locked.has(x)));
						//console.log(_set_locked, _set_unLocked);
						if(_set_unLocked.size === 1){
							this.sudoku_data[i].userlocked = false;
							this.sudoku_data[i].innerdata = Array.from(_set_unLocked);
						}
						else {
							this.sudoku_data[i].userlocked = false;
							this.sudoku_data[i].innerdata = Array.from(_set_unLocked);
						}
					}
				}
				// console.log('14号单元格添加锁定前 行:',this.sudoku_data[14].belongrow.locked);
				// console.log('14号单元格添加锁定前 列:',this.sudoku_data[14].belongcol.locked);
				// console.log('14号单元格添加锁定前区域:',this.sudoku_data[14].belongarea.locked);
			},
			
			LockOnlyOne(){
				//将只有一个值的单元格进行锁定
				for(let i = 0; i<81; i++){
					if(!this.sudoku_data[i].system && this.sudoku_data[i].innerdata.length===1){
						this.sudoku_data[i].userlocked = true;
						//修改单元所属行的已锁定单元格
						this.sudoku_data[i].belongrow.locked.push(this.sudoku_data[i].innerdata[0]);
						//清除重复添加的数值
						this.sudoku_data[i].belongrow.locked = Array.from(new Set(this.sudoku_data[i].belongrow.locked))
						
						//修改单元所属列的已锁定单元格
						this.sudoku_data[i].belongcol.locked.push(this.sudoku_data[i].innerdata[0]);
						//清除重复添加的数值
						this.sudoku_data[i].belongcol.locked = Array.from(new Set(this.sudoku_data[i].belongcol.locked))
						
						//修改单元所属区域的已锁定单元格
						this.sudoku_data[i].belongarea.locked.push(this.sudoku_data[i].innerdata[0]);
						//清除重复添加的数值
						this.sudoku_data[i].belongarea.locked = Array.from(new Set(this.sudoku_data[i].belongarea.locked))
					}
				}
				// console.log('14号单元格添加锁定后 行:',this.sudoku_data[14].belongrow.locked);
				// console.log('14号单元格添加锁定后 列:',this.sudoku_data[14].belongcol.locked);
				// console.log('14号单元格添加锁定后区域:',this.sudoku_data[14].belongarea.locked);
			},
			
			DuplicateRemoval(){
				//有多个值的单元格，删除已在行，列，区域中被锁定的数值
				for(let i = 0; i<81; i++){
					if (!this.sudoku_data[i].system && !this.sudoku_data[i].userlocked) {
						//对用户单元格进行处理，且该单元格未被锁定
						let _set_row = new Set(this.sudoku_data[i].belongrow.locked);
						let _set_col = new Set(this.sudoku_data[i].belongcol.locked);
						let _set_area = new Set(this.sudoku_data[i].belongarea.locked);
						let _set_locked = new Set([..._set_row, ..._set_col, ..._set_area]);
						let _set_unLocked = new Set([1, 2, 3, 4, 5, 6, 7, 8, 9].filter(x => !_set_locked.has(x)));
						//console.log(_set_locked, _set_unLocked);
						if(_set_unLocked.size === 1){
							this.sudoku_data[i].userlocked = false;
							this.sudoku_data[i].innerdata = Array.from(_set_unLocked);
						}
						else {
							this.sudoku_data[i].userlocked = false;
							this.sudoku_data[i].innerdata = Array.from(_set_unLocked);
						}
					}
				}
			}
		},

		created: function() {
			//根据初始值，生成81个单元格对象放入sudoku_data中
			//this.initSudokuData()
			//初始化9个行对象
			this.initSudokuRows();
			//初始化9个列对象
			this.initSudokuCols();
			//初始化9个区域对象
			this.initSudokuAreas();

		},
		components: {
			HelloWorld,
			cell
		}
	};
</script>

<style>
	#app {
		font-family: "Avenir", Helvetica, Arial, sans-serif;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
		text-align: center;
		color: #2c3e50;
		margin-top: 60px;
	}

	.game {
		display: flex;
		justify-content: center;
		align-items: center;
	}

	.sudoku {
		display: grid;
		grid-template-columns: repeat(3, 180px);
		grid-template-rows: repeat(3, 180px);
		grid-gap: 12px 12px
	}

	.sudoku_area {
		display: grid;
		grid-template-columns: repeat(3, 60px);
		grid-template-rows: repeat(3, 60px);
		grid-gap: 3px 3px;
	}

	.footer {
		display: flex;
		justify-content: center;
		align-items: center;
		width: 100%;
		height: 100px;
	}
</style>
