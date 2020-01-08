<template>
	<!--是系统提供的数据-->
	<div v-if="this.cellnumber.system" class="container01"  tabindex="0" @keyup="cellkeyup($event)" @focus="cellclick(cellnumber)" @blur="cellblur(cellnumber)">
		<div class="area0" :class="{'selected':cellnumber.selected}">
			<span>{{this.cellnumber.innerdata[0]}}</span>
		</div>
	</div>
	<!--用户数据，且锁定了唯一值-->
	<div v-else-if="this.cellnumber.userlocked" class="container01"  tabindex="0" @keyup="cellkeyup($event)" @focus="cellclick(cellnumber)" @blur="cellblur(cellnumber)">
		<div class="areaUserlocked" :class="{'selected':cellnumber.selected}">
			<span>{{this.cellnumber.innerdata[0]}}</span>
		</div>
	</div>
	<!--用户数据，没有锁定唯一值，数据按九宫格展示-->
	<div v-else class="container" tabindex="0" @keyup="cellkeyup($event)">
		<div class="area1">
			<span>{{(this.cellnumber.innerdata.indexOf(1)!==-1)?1:''}}</span>
		</div>
		<div class="area2">
			<span>{{(this.cellnumber.innerdata.indexOf(2)!==-1)?2:''}}</span>
		</div>
		<div class="area3">
			<span>{{(this.cellnumber.innerdata.indexOf(3)!==-1)?3:''}}</span>
		</div>

		<div class="area4">
			<span>{{(this.cellnumber.innerdata.indexOf(4)!==-1)?4:''}}</span>
		</div>
		<div class="area5">
			<span>{{(this.cellnumber.innerdata.indexOf(5)!==-1)?5:''}}</span>
		</div>
		<div class="area6">
			<span>{{(this.cellnumber.innerdata.indexOf(6)!==-1)?6:''}}</span>
		</div>

		<div class="area7">
			<span>{{(this.cellnumber.innerdata.indexOf(7)!==-1)?7:''}}</span>
		</div>
		<div class="area8">
			<span>{{(this.cellnumber.innerdata.indexOf(8)!==-1)?8:''}}</span>
		</div>
		<div class="area9">
			<span>{{(this.cellnumber.innerdata.indexOf(9)!==-1)?9:''}}</span>
		</div>
	</div>
</template>

<script>
	export default {
		name: 'cell',
		props: {
			initcellnumber: {
				type: Object,
				default: function() {
					return {};
				},
			},
		},
		methods: {
			cellblur(blur_cell) {
				console.log('blur',blur_cell);
				blur_cell.selected = false;
			},
			cellclick(selected_cell) {
				console.log(selected_cell);
				selected_cell.selected = true;
			},
			cellkeyup(event) {
				console.log(event);
				switch (event.keyCode) {
					case 13:	// 按下回车键，确定要填写的数据
						{
							if (!this.cellnumber.system && this.cellnumber.innerdata.length === 1) {
								//如果是用户数据部分，且只有唯一数据，表示用户准备锁定该单元格
								this.cellnumber.userlocked = true;
							}
							break;
						}
					case 32:	// 按下了空格键， 清除所填写的数据
						{
							if(!this.cellnumber.system){
								this.cellnumber.innerdata = [];
								this.cellnumber.userlocked = true;
							}
							break;
						}
					case 37:	// 左方向键
						{
							let cell_id = parseInt(event.target.id);
							cell_id = cell_id - 1;
							if(cell_id % 9 === 0){
								// 在第1列的单元格按下了左方向锓，跳到第9列
								cell_id = cell_id + 9;
							}
							document.getElementById(cell_id).focus();
							break;
						}
					case 38:	// 上方向键
						{
							let cell_id = parseInt(event.target.id);
							cell_id = cell_id - 9;
							if(cell_id <= 0){
								// 在第1行的单元各按下了上方向键，跳到第9行
								cell_id = cell_id + 81;
							}
							document.getElementById(cell_id).focus();
							break;
						}
					case 39:	// 右方向键
						{
							let cell_id = parseInt(event.target.id);
							cell_id = cell_id + 1;
							if(cell_id % 9 === 1){
								// 在第9列的单元格按下了左方向锓，跳到第1列
								cell_id = cell_id - 9;
							}
							document.getElementById(cell_id).focus();
							break;
						}
					case 40:	//  下方向键
						{
							let cell_id = parseInt(event.target.id);
							cell_id = cell_id + 9;
							if(cell_id >= 82){
								// 在第9行的单元各按下了上方向键，跳到第1行
								cell_id = cell_id - 81;
							}
							document.getElementById(cell_id).focus();
							break;
						}
					case 49:
					case 50:
					case 51:
					case 52:
					case 53:
					case 54:
					case 55:
					case 56:
					case 57:
					case 97:
					case 98:
					case 99:
					case 100:
					case 101:
					case 102:
					case 103:
					case 104:
					case 105:
						{
							if (!this.cellnumber.system) {
								let x = parseInt(event.key);
								if (this.cellnumber.innerdata.indexOf(x) === -1) {
									//添加数值
									this.cellnumber.innerdata.push(x);
									this.cellnumber.userlocked = false;
								} else {
									//删除数值
									this.cellnumber.innerdata.splice(this.cellnumber.innerdata.indexOf(x), 1);
									if(this.cellnumber.innerdata.length===0){
										this.cellnumber.userlocked = true;
									}
								}
							}
							break;
						}
				}
			}
		},
		computed: {
			// 如果单元格中的数据有2个及以上，返回true
			// ismulti:function(){
			// 	return (this.cellnumber.innerdata.length>1)?true:false;
			// },
		},
		data: function() {
			return {
				cellnumber: this.initcellnumber
			}
		},
		watch:{
			initcellnumber:{
				handler(newData){
					this.cellnumber = newData;
				},
				//deep: true,
				//immediate: true
			}
		}
	}
</script>

<style scoped>
	.focused {
		border: #FF0000 solid 1px
	}

	.focusedarea1 {
		border-style: solid;
		border-width: 1px;
		border-top-color: red;
		border-right-color: transparent;
		border-bottom-color: transparent;
		border-left-color: red;
	}

	.focusedarea2 {
		border-style: solid;
		border-width: 1px;
		border-top-color: red;
		border-right-color: transparent;
		border-bottom-color: transparent;
		border-left-color: transparent;
	}

	.focusedarea3 {
		border-style: solid;
		border-width: 1px;
		border-top-color: red;
		border-right-color: red;
		border-bottom-color: transparent;
		border-left-color: transparent;
	}

	.focusedarea4 {
		border-style: solid;
		border-width: 1px;
		border-top-color: transparent;
		border-right-color: transparent;
		border-bottom-color: transparent;
		border-left-color: red;
	}

	.focusedarea6 {
		border-style: solid;
		border-width: 1px;
		border-top-color: transparent;
		border-right-color: red;
		border-bottom-color: transparent;
		border-left-color: transparent;
	}

	.focusedarea7 {
		border-style: solid;
		border-width: 1px;
		border-top-color: transparent;
		border-right-color: transparent;
		border-bottom-color: red;
		border-left-color: red;
	}

	.focusedarea8 {
		border-style: solid;
		border-width: 1px;
		border-top-color: transparent;
		border-right-color: transparent;
		border-bottom-color: red;
		border-left-color: transparent;
	}

	.focusedarea9 {
		border-style: solid;
		border-width: 1px;
		border-top-color: transparent;
		border-right-color: red;
		border-bottom-color: red;
		border-left-color: transparent;
	}

	.container01 {
		display: grid;
		grid-template-columns: repeat(3, 20px);
		grid-template-rows: repeat(3, 20px);
		grid-gap: 1px 1px;
	}
	
	.container01:focus{
		/* border:green solid thin; */
		outline:#ff0000 solid 3px;
	}

	.area0 {
		text-align: center;
		background-color: gray;
		font-size: 48px;
		line-height: 1.25;
		padding: 0;
		width: 60px;
		height: 60px;
	}

	.areaUserlocked {
		text-align: center;
		background-color: darkseagreen;
		font-size: 48px;
		line-height: 1.25;
		padding: 0;
		width: 60px;
		height: 60px;
	}

	.container {
		display: grid;
		grid-template-columns: repeat(3, 20px);
		grid-template-rows: repeat(3, 20px);
	}
	
	.container:focus{
		/* border:green solid thin; */
		outline:#ff0000 solid 3px;
	}

	.area1 {
		text-align: center;
		background-color: darkgray;
		font-size: 18px;
		line-height: 1.15;
		width: 20px;
		height: 20px;
	}

	.area2 {
		text-align: center;
		background-color: wheat;
		font-size: 18px;
		line-height: 1.15;
		width: 20px;
		height: 20px;
	}

	.area3 {
		text-align: center;
		background-color: darkgray;
		font-size: 18px;
		line-height: 1.15;
		width: 20px;
		height: 20px;
	}

	.area4 {
		text-align: center;
		background-color: wheat;
		font-size: 18px;
		line-height: 1.15;
		width: 20px;
		height: 20px;
	}

	.area5 {
		text-align: center;
		background-color: darkgray;
		font-size: 18px;
		line-height: 1.15;
		width: 20px;
		height: 20px;
	}

	.area6 {
		text-align: center;
		background-color: wheat;
		font-size: 18px;
		line-height: 1.15;
		width: 20px;
		height: 20px;
	}

	.area7 {
		text-align: center;
		background-color: darkgray;
		font-size: 18px;
		line-height: 1.15;
		width: 20px;
		height: 20px;
	}

	.area8 {
		text-align: center;
		background-color: wheat;
		font-size: 18px;
		line-height: 1.15;
		width: 20px;
		height: 20px;
	}

	.area9 {
		text-align: center;
		background-color: darkgray;
		font-size: 18px;
		line-height: 1.15;
		width: 20px;
		height: 20px;
	}
	
	.selected {
		background-color: yellow;
	}
</style>
