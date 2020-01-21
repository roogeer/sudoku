<template>
	<div id="app">
		<!--<img alt="Vue logo" src="./assets/logo.png" />-->
		<HelloWorld msg="Sudoku v0.6" />
		<div class="hello">
			<h3>#{{ sudokuid }}</h3>
		</div>
		<div class="game reflect">
			<div class="sudoku">
				<template v-for="(area_row, index_area_row) in 3">
					<template v-for="(area_col, index_area_col) in 3">
						<div class="sudoku_area" :key="(index_area_row - 1) * 3 + (index_area_col -1 )">
							<template v-for="row in 3">
								<template v-for="col in 3">
									<cell @pressEnter="handle_pressEnter" @lostblur="handle_lostblur" @selectx="handle_selectx" :initcellnumber="sudoku_data[(area_row - 1) * 27 + (area_col - 1) * 3 + (row -1) * 9 + (col - 1)]" :key="(area_row -1 ) * 27 + (area_col -1 ) * 3 + (row - 1) * 9 + (col - 1)" :id="(area_row - 1) * 27 + (area_col - 1) * 3 + (row -1) * 9 + col" />
								</template>
							</template>
						</div>
					</template>
				</template>
			</div>
		</div>
		<div style="height: 120px;"></div>
		<div class="footer">
			<div style="width: 25%" />
			<div style="width: 50%" class="footer content">
				<div class="newgame">
					<button @click="gameStart()" v-if="!userDefineMode" class="button black red">新的游戏</button>
					<button @click="gameReStart()" v-if="!userDefineMode" class="button black green">重新开始</button>
					<button @click="userDefine()" v-if="!userDefineMode" class="button black blue">自定义游戏</button>
					<button @click="userDefineComplete()" v-else class="button black blue">自定义完成</button>
				</div>
				<div class="cheat">
					<template v-if="!userDefineMode">
						<template v-if="cheat">
							<button @click="Cheat()" class="button black tags">开始作弊</button>
						</template>
						<template v-else>
							<button @click="LockOnlyOne()" class="button black tags">锁定唯一值</button>
							<button @click="DuplicateRemoval()" class="button black tags">多值去重</button>
							<button @click="FilterOnlyOne()" class="button black tags">筛选唯一值</button>
						</template>
					</template>
				</div>
			</div>
			<div style="width: 25%" />
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
				userDefineMode: false,	//当前工作模式
				cheat: true,			//显示作弊按钮
				sudokuid: '有点难',		//题号
				//sudoku_array: [],		//系统给出的初始数据
				
				sudoku_array: [
					0, 0, 8, 0, 7, 0, 4, 0, 0,
					4, 2, 0, 0, 0, 0, 0, 9, 5,
					0, 0, 0, 3, 0, 4, 0, 0, 0,
					0, 7, 0, 0, 1, 0, 0, 8, 0,
					0, 8, 0, 9, 4, 7, 0, 1, 0,
					0, 5, 0, 0, 6, 0, 0, 7, 0,
					0, 0, 0, 6, 0, 1, 0, 0, 0,
					7, 3, 0, 0, 0, 0, 0, 2, 9,
					0, 0, 5, 0, 3, 0, 8, 4, 0
				],
				
				sudoku_data: [],
				
				//9个行对象
				sudoku_rows: [],

				//9个列对象
				sudoku_cols: [],

				//9个区域对象
				sudoku_areas: [],
				
				//当前被选中的数值
				numberSelected: 0,
				
				//已被选中的cells
				cellSelected: null,
				
				//将被选中的所有cells
				cellsSelected: [],
				
				//当前被辐射到的cells
				cellsIrradiated: []
			};
		},
		
		methods: {
			//处理用户数据锁定事件
			handle_pressEnter(){
				this.LockOnlyOne();
				// console.log('handle_pressEnter lockedCell', lockedCell);
				// console.log('handle_pressEnter sudou_data ', this.sudoku_data[lockedCell.sn]);
				// this.sudoku_data[lockedCell.sn] = lockedCell;
			},
			
			//处理焦点丢失事件
			handle_lostblur(){
				//清理高亮cell
				for(let i = 0; i < this.cellsSelected.length; i++){
					this.cellsSelected[i].selected = false;
				}
				this.cellsSelected.splice(0);
				
				//清理被辐射的cell
				for(let i=0; i<this.cellsIrradiated.length; i++){
					this.cellsIrradiated[i].irradiated = false;
				}
				this.cellsIrradiated.splice(0);
			},
			//处理cell组件中传回的数据
			handle_selectx(selected_cell){
				this.cellSelected = selected_cell;
				this.numberSelected = selected_cell.innerdata[0];
				//console.log(this.numberSelected);
				
				//如果cell中的数值唯一，且与选中的数值相等，添加到cellsSelected数组中
				for(let i = 0; i < this.cellsSelected.length; i++){
					this.cellsSelected[i].selected = false;
				}
				
				this.cellsSelected.splice(0);
				
				for(let index = 0; index < 81; index++){
					if(this.sudoku_data[index].system || this.sudoku_data[index].userlocked ){
						if(this.numberSelected!==undefined && this.sudoku_data[index].innerdata[0]===this.numberSelected){
							//console.log(this.sudoku_data[index].innerdata[0], this.numberSelected);
							this.sudoku_data[index].selected = true;
							this.cellsSelected.push(this.sudoku_data[index]);
						}
					}
				}
				//console.log(this.cellsSelected);
				this.processCellsIrradiated();
			},
			
			//处理被辐射到的单元格
			processCellsIrradiated(){
				for(let index=0; index<9; index++){
					this.cellSelected.belongarea.cells[index].irradiated = true;
					this.cellsIrradiated.push(this.cellSelected.belongarea.cells[index]);
				}
				
				for(let index=0; index<this.cellsSelected.length; index++){
					for(let i=0; i<9; i++){
						//处理被辐射的区域
						this.cellsSelected[index].belongarea.cells[i].irradiated = true;
						this.cellsIrradiated.push(this.cellsSelected[index].belongarea.cells[i]);
						
						//处理被辐射的行
						this.cellsSelected[index].belongrow.cells[i].irradiated = true;
						this.cellsIrradiated.push(this.cellsSelected[index].belongrow.cells[i]);
						
						//处理被辐射的列
						this.cellsSelected[index].belongcol.cells[i].irradiated = true;
						this.cellsIrradiated.push(this.cellsSelected[index].belongcol.cells[i]);
					}
				}

			},
			
			userDefine(){
				this.sudokuid = '自定义';
				this.sudoku_array = [
					0, 0, 0, 0, 0, 0, 0, 0, 0,
					0, 0, 0, 0, 0, 0, 0, 0, 0,
					0, 0, 0, 0, 0, 0, 0, 0, 0,
					0, 0, 0, 0, 0, 0, 0, 0, 0,
					0, 0, 0, 0, 0, 0, 0, 0, 0,
					0, 0, 0, 0, 0, 0, 0, 0, 0,
					0, 0, 0, 0, 0, 0, 0, 0, 0,
					0, 0, 0, 0, 0, 0, 0, 0, 0,
					0, 0, 0, 0, 0, 0, 0, 0, 0
				];
				this.initSudokuData();
				this.cheat = true;
				this.userDefineMode = true;
			},
			userDefineComplete(){
				this.cheat = true;
				this.userDefineMode = false;
				for(let index=0; index<81; index++){
					//console.log(index, this.sudoku_data[index].innerdata);
					if(this.sudoku_data[index].innerdata.length!==0){
						//console.log(index, this.sudoku_data[index].innerdata[0])
						let _tmp = this.sudoku_data[index].innerdata[0];
						this.sudoku_data[index].innerdata.splice(0);
						this.sudoku_data[index].innerdata.push(_tmp);
						this.sudoku_data[index].system = true;
						this.sudoku_data[index].userlocked = false;
						this.sudoku_data[index].belongrow = {}; //单元格所属的行
						this.sudoku_data[index].belongcol = {}; //单元格所属的列
						this.sudoku_data[index].belongarea = {}; //单元格所属的区域
						this.sudoku_data[index].selected = false;//单元格是否被选中
						
						this.sudoku_array[index] = _tmp;
					}
					else{
						this.sudoku_data[index].innerdata.splice(0);
						this.sudoku_data[index].system = false;
						this.sudoku_data[index].userlocked = true;
						this.sudoku_data[index].belongrow = {}; //单元格所属的行
						this.sudoku_data[index].belongcol = {}; //单元格所属的列
						this.sudoku_data[index].belongarea = {}; //单元格所属的区域
						this.sudoku_data[index].selected = false;//单元格是否被选中
						
						this.sudoku_array[index] = 0;
					}
				}
				
				this.initSudokuData();
				this.initSudokuRows();
				this.initSudokuCols();
				this.initSudokuAreas();

				//将用户自定义的数据，存入浏览器，F5刷新时可取回
				sessionStorage.setItem('userdefineid', JSON.stringify('自定义'));
				sessionStorage.setItem('userdefine', JSON.stringify(this.sudoku_array));
			},
			gameStart(){
				this.$axios.get('http://192.168.108.79:8983/cgi-bin/getdata.py')
				.then(response => {
					this.sudokuid = response.data.sudokuid;
					this.sudoku_array.splice(0);
					this.sudoku_array.push(...response.data.sudokudata);
					this.cheat = true;
					
					this.initSudokuData();
					
					this.initSudokuRows();
					this.initSudokuCols();
					this.initSudokuAreas();
					
					//将游戏数据，存入浏览器缓存，F5刷新时可取回
					sessionStorage.setItem('userdefineid', JSON.stringify(this.sudokuid));
					sessionStorage.setItem('userdefine', JSON.stringify(this.sudoku_array));
				})
			},
			
			filterOnlyOne_rows(){
				// 处理 行
				for(let i=0; i<81; i++){
					let _tempcell = this.sudoku_data[i];		//取出待处理单元格
					if(!_tempcell.system && _tempcell.innerdata.length>1){
						// 如果是用户单元格，且有多个可选值
						// 把该行上其他单元格的可选数值取出来
						let _tempRows = _tempcell.belongrow;	//取所属的 行 对象
						let _tempOtherCellsArray = [];			//非待处理单元格的所有可选数值集合
						for(let cell of _tempRows.cells){
							if(cell !== _tempcell){
								//如果不是待处理单元格本身，记录单元格的可选数值
								_tempOtherCellsArray.push(...cell.innerdata);
							}
						}
						//清除重复添加的数值
						_tempOtherCellsArray = Array.from(new Set(_tempOtherCellsArray));
						
						// 判断 x 是否在其他单元格的可选数值中
						for(let x of _tempcell.innerdata){
							if(_tempOtherCellsArray.indexOf(x) === -1){
								//console.log('行中其他单元格的可选值', _tempOtherCellsArray, '单元格', i+1, '的:',x, "是唯一值");
								// 处理单元格
								this.sudoku_data[i].innerdata.splice(0);
								this.sudoku_data[i].innerdata.push(x);
							}
						}
					}
				}
			},
			
			filterOnlyOne_cols(){
				// 处理 列
				for(let i=0; i<81; i++){
					let _tempcell = this.sudoku_data[i];		//取出待处理单元格
					if(!_tempcell.system && _tempcell.innerdata.length>1){
						// 如果是用户单元格，且有多个可选值
						// 把该行上其他单元格的可选数值取出来
						let _tempRows = _tempcell.belongcol;	//取所属的 列 对象
						let _tempOtherCellsArray = [];			//非待处理单元格的所有可选数值集合
						for(let cell of _tempRows.cells){
							if(cell !== _tempcell){
								//如果不是待处理单元格本身，记录单元格的可选数值
								_tempOtherCellsArray.push(...cell.innerdata);
							}
						}
						//清除重复添加的数值
						_tempOtherCellsArray = Array.from(new Set(_tempOtherCellsArray));
						
						// 判断 x 是否在其他单元格的可选数值中
						for(let x of _tempcell.innerdata){
							if(_tempOtherCellsArray.indexOf(x) === -1){
								//console.log('列中其他单元格的可选值', _tempOtherCellsArray, '单元格', i+1, '的:',x, "是唯一值");
								// 处理单元格
								this.sudoku_data[i].innerdata.splice(0);
								this.sudoku_data[i].innerdata.push(x);
							}	
						}
					}
				}
			},
			
			filterOnlyOne_areas(){
				// 处理 区域
				for(let i=0; i<81; i++){
					let _tempcell = this.sudoku_data[i];		//取出待处理单元格
					if(!_tempcell.system && _tempcell.innerdata.length>1){
						// 如果是用户单元格，且有多个可选值
						// 把该行上其他单元格的可选数值取出来
						let _tempRows = _tempcell.belongarea;	//取所属的 区域 对象
						let _tempOtherCellsArray = [];			//非待处理单元格的所有可选数值集合
						for(let cell of _tempRows.cells){
							if(cell !== _tempcell){
								//如果不是待处理单元格本身，记录单元格的可选数值
								_tempOtherCellsArray.push(...cell.innerdata);
							}
						}
						//清除重复添加的数值
						_tempOtherCellsArray = Array.from(new Set(_tempOtherCellsArray));
						
						// 判断 x 是否在其他单元格的可选数值中
						for(let x of _tempcell.innerdata){
							if(_tempOtherCellsArray.indexOf(x) === -1){
								//console.log('区域中其他单元格的可选值', _tempOtherCellsArray, '单元格', i+1, '的:',x, "是唯一值");
								// 处理单元格
								this.sudoku_data[i].innerdata.splice(0);
								this.sudoku_data[i].innerdata.push(x);
							}
						}
					}
				}
			},
			
			FilterOnlyOne(){
				// 如果某个单元格的可选值在行、列或是区域中是唯一的，则表示该单元格就是该值
				this.filterOnlyOne_rows();
				this.filterOnlyOne_cols();
				this.filterOnlyOne_areas();
			},
			
			///初始化sudoku_data成员
			initSudokuData(){
				let _temp = [];
				for (let i = 0; i < this.sudoku_array.length; i++) {
					if (0 === this.sudoku_array[i]) {
						_temp.push({
							sn: i,				//单元格序号
							innerdata: [],		//填入的可选数值
							system: false,		//系统给定了提示数值
							userlocked: false,	//用户填入唯一数值，并确定
							isEmpty: true,		//没有填入任何数值
							belongrow: {},		//单元格所属的行
							belongcol: {},		//单元格所属的列
							belongarea: {},		//单元格所属的区域
							selected: false,	//单元格是否被选中
							irradiated: false,	//单元格是否被辐射
						});
					} else {
						_temp.push({
							sn: i,				//单元格序号
							innerdata: [this.sudoku_array[i]],
							system: true,		//系统给定了提示数值
							userlocked: false,	//用户填入唯一数值，并确定
							isEmpty: false,		//没有填入任何数值
							belongrow: {},		//单元格所属的行
							belongcol: {},		//单元格所属的列
							belongarea: {},		//单元格所属的区域
							selected: false,	//单元格是否被选中
							irradiated: false,	//单元格是否被辐射
						})
					}
				}
				this.sudoku_data = _temp;
				//return _temp;
			},
			
			//初始化9个行对象
			initSudokuRows() {
				//先清空数组
				this.sudoku_rows.splice(0);
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
			},

			//初始化9个列对象
			initSudokuCols() {
				//先清空数组
				this.sudoku_cols.splice(0);
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
			},

			//初始化9个区域对象
			initSudokuAreas() {
				//先清空数组
				this.sudoku_areas.splice(0);
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
			},

			//开始游戏
			Cheat() {
				//进入作弊模式后，关闭作弊按钮，开启另外2个按钮
				this.cheat = false;
				
				//初始化9个行对象
				this.initSudokuRows();
				
				//初始化9个列对象
				this.initSudokuCols();
				
				//初始化9个区域对象
				this.initSudokuAreas();

				//对每个单元格的行，列，区域进行查找已锁定的数据
				for (let i = 0; i < 81; i++) {
					if (!this.sudoku_data[i].system && this.sudoku_data[i].innerdata.length!==1) {
						//对用户单元格进行处理
						let _set_row = new Set(this.sudoku_data[i].belongrow.locked);
						let _set_col = new Set(this.sudoku_data[i].belongcol.locked);
						let _set_area = new Set(this.sudoku_data[i].belongarea.locked);
						let _set_locked = new Set([..._set_row, ..._set_col, ..._set_area]);
						let _set_unLocked = new Set([1, 2, 3, 4, 5, 6, 7, 8, 9].filter(x => !_set_locked.has(x)));
						if(_set_unLocked.size === 1){
							this.sudoku_data[i].userlocked = false;
							this.sudoku_data[i].isEmpty = false;
							this.sudoku_data[i].innerdata = Array.from(_set_unLocked);
						}
						else {
							this.sudoku_data[i].userlocked = false;
							this.sudoku_data[i].isEmpty = false;
							this.sudoku_data[i].innerdata = Array.from(_set_unLocked);
						}
					}
				}
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
			},
			
			gameReStart(){
				if(sessionStorage.getItem('userdefine')!==null){
					//console.log('有用户自定义的盘局');
					this.sudokuid = JSON.parse(sessionStorage.getItem('userdefineid'));
					this.sudoku_array = JSON.parse(sessionStorage.getItem('userdefine'));
					//console.log(this.sudoku_array);
				}
				this.initSudokuData();
				this.initSudokuRows();
				this.initSudokuCols();
				this.initSudokuAreas();
				
				this.cheat = true;
			}
		},

		created: function() {
			this.gameReStart();
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
		grid-gap: 18px 18px
	}

	.sudoku_area {
		display: grid;
		grid-template-columns: repeat(3, 60px);
		grid-template-rows: repeat(3, 60px);
		grid-gap: 3px 3px;
	}

	.footer {
		display: flex;
		flex-wrap: wrap;
		align-content: space-around;
		justify-content: center;
		align-items: center;
		width: 100%;
		height: 80px;
	}
	
	.footer.content{
		opacity: 0;
		transition:opacity 2s;
		-moz-transition:opacity 2s;		/* Firefox 4 */
		-webkit-transition:opacity 2s;	/* Safari and Chrome */
		-o-transition:opacity 2s;		/* Opera */		
	}
	
	.footer.content:hover{
		opacity: 1;
	}
	
	.footer .newgame {
		width: 100%;
	}
	
	.footer .cheat {
		width: 100%;
	}
	
	.reflect{
        width: 100%;
        -webkit-box-reflect: below 20px -webkit-linear-gradient(transparent,transparent 90%,rgba(255, 255, 255, .3));
	}
	
    .button{
        width: 120px;
        line-height: 28px;
        text-align: center;
        font-weight: bold;
        color: #aaaaaa;
        text-shadow:1px 1px 1px #333;
        border-radius: 5px;
        margin:0 10px 20px 10px;
        position: relative;
        overflow: hidden;
    }
    .button.black{
        border:1px solid #333;
        box-shadow: 0 1px 2px #8b8b8b inset,0 -1px 0 #3d3d3d inset,0 -2px 3px #8b8b8b inset;
        background: -webkit-linear-gradient(top,#656565,#4c4c4c);
        background: -moz-linear-gradient(top,#656565,#4a4a4a);
        background: linear-gradient(top,#656565,#4a4a4a);
    }
    .black.tags:after{
        background: #333;
        border:2px solid #777;
    }
    .tags:after{
        font-weight: normal;
        position: absolute;
        display: inline-block;
        content: "sudoku";
        top:-3px;
        right: -33px;
        color: #fff;
        text-shadow:none;
        width: 85px;
        height:25px;
        line-height: 28px;
        -webkit-transform:rotate(45deg) scale(.7,.7);
        -moz-transform:rotate(45deg) scale(.7,.7);
        transform:rotate(45deg) scale(.7,.7);
    }
	
    .black.red:after{
        background: #b11a1a;
        border:2px solid #777;
    }
    .red:after{
        font-weight: normal;
        position: absolute;
        display: inline-block;
        content: "sudoku";
        top:-3px;
        right: -33px;
        color: #fff;
        text-shadow:none;
        width: 85px;
        height:25px;
        line-height: 28px;
        -webkit-transform:rotate(45deg) scale(.7,.7);
        -moz-transform:rotate(45deg) scale(.7,.7);
        transform:rotate(45deg) scale(.7,.7);
    }
	
    .black.blue:after{
        background: #1a1ab1;
        border:2px solid #777;
    }
    .blue:after{
        font-weight: normal;
        position: absolute;
        display: inline-block;
        content: "sudoku";
        top:-3px;
        right: -33px;
        color: #fff;
        text-shadow:none;
        width: 85px;
        height:25px;
        line-height: 28px;
        -webkit-transform:rotate(45deg) scale(.7,.7);
        -moz-transform:rotate(45deg) scale(.7,.7);
        transform:rotate(45deg) scale(.7,.7);
    }
	
    .black.green:after{
        background: #1ab11a;
        border:2px solid #777;
    }
    .green:after{
        font-weight: normal;
        position: absolute;
        display: inline-block;
        content: "sudoku";
        top:-3px;
        right: -33px;
        color: #fff;
        text-shadow:none;
        width: 85px;
        height:25px;
        line-height: 28px;
        -webkit-transform:rotate(45deg) scale(.7,.7);
        -moz-transform:rotate(45deg) scale(.7,.7);
        transform:rotate(45deg) scale(.7,.7);
    }
</style>