<template>
	<div id="app">
		<!--<img alt="Vue logo" src="./assets/logo.png" />-->
		<HelloWorld msg="Sudoku v0.1" />
		<!-- <button @click="test">查看B1单元格数据</button> -->
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
	</div>
</template>

<script>
	import HelloWorld from "./components/HelloWorld.vue";
	import cell from "./components/cell.vue";

	export default {
		name: "app",
		data: function() {
			return {
				sudoku_array:[
					0,4,1,0,0,2,0,3,0,
					6,0,0,0,8,0,0,7,0,
					8,0,0,1,9,0,4,5,0,
					4,0,0,5,0,6,7,0,0,
					1,2,0,0,0,3,0,4,8,
					0,0,3,4,0,0,0,0,1,
					0,1,8,0,6,5,0,0,4,
					0,5,0,0,3,0,0,0,7,
					0,6,0,8,0,0,3,1,0
				],
				sudoku_data:[]
			};
		},

		methods: {
			initSudokuData(){
				let _temp = [];
				console.log(this.sudoku_array.length);
				for(let i=0; i<this.sudoku_array.length; i++){
					if(0===this.sudoku_array[i]){
						_temp.push({
							innerdata: [],
							system: false,
							userlocked: true,
						});
					}
					else{
						_temp.push({
							innerdata: [this.sudoku_array[i]],
							system: true,
							userlocked: false,
						})
					}
				}
				console.log(_temp);
				this.sudoku_data = _temp;
			}
		},
		created:function(){
			this.initSudokuData()
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
</style>
