<template>
	<div class="complex-select" ref="complexSelect">
		<el-tooltip :disabled="!selectStrTips" popper-class="complex-select-tooltip" :width="300" placement="left" effect="light">
			<div class="complex-select-tooltip_content" v-html="selectStrTips" slot="content"></div>
			<div ref="complexSelectTrigger" v-popover:selectPopover class="simulate-select">
				<div class="simulate-select-inner">
					<ul class="simulate-select-inner-content" :placeholder="placeholder">
						<li class="simulate-select-checked-item" v-for="item in selectStr">
							<span>{{item.desc}}</span>
						</li>
					</ul>
				</div>
				<span @click.stop="cleanSelect" class="simulate-select-suf">
					<i class="el-icon-close"></i>
				</span>
			</div>
		</el-tooltip>
		<el-popover v-model="selectPopoverVisible" ref="selectPopover" popper-class="simulate-select-popover" placement="bottom-start" trigger="click" @show="showSelect" @hide="hideSelect">
			<div class="select-box">
				<div class="primary-menu select-menu-list">
					<ul>
						<el-popover  trigger="hover" :enterable='true' :append-to-body="false" ref="subSelectPopover" v-for="(item,ind) in selectMenuList" :key="item.type" popoer-class="simulate-select-sub-popover" placement="right">
							<li slot="reference" class="pull-left" :class="{'is-active':item.isAct}">
								<el-checkbox :indeterminate="item.isAll" v-model="item.isAct" @change="selectMenuHandle(item,ind)">{{item.desc}}</el-checkbox>
							</li>
							<div class="simulate-select-subMenu-content" v-cloak>
								<div class="simulate-select-subMenu-content-head">
									<el-checkbox :indeterminate="item.isAll" v-model="item.isAct" @change="selectMenuHandle(item,ind)">{{item.desc}} <span class="text-primary" v-show="item.menu&&item.menu.length">(全选)</span></el-checkbox>
								</div>
								<div class="check-group-list">
									<el-checkbox size="mini" @change="changeSubSelect(sub,ind)" style="margin-left: 0;" v-model="sub.isAct" v-for="(sub,inds) in item.menu" :key="sub.type">
										{{sub.desc}}
									</el-checkbox>
								</div>
							</div>
						</el-popover>
					</ul>
				</div>
				<div class="btn-row">
					<el-button plain size="mini" @click="submitSelect">确定</el-button>
					<el-button plain size="mini" @click="closePopover">取消</el-button>
				</div>
			</div>
		</el-popover>
	</div>
</template>

<script>
	export default {
		name: 'complexSelect',
		data() {
			return {
				selectMenuList: [], //合成数据
				selectStr: [],
				selectPopoverVisible:false,
			}
		},
		props: {
			menuList: {
				type: Array,
				default: function() {
					return []
				}
			},
			selectType: {
				type: Array,
				default: function() {
					return []
				}
			},
			placeholder: {
				type: String,
				default: '请选择类型'
			}
		},
		computed:{
			selectStrTips(){			
				return this.selectStr && this.selectStr.length ? this.selectStr.map(item => item.desc).join('&emsp;') : '';
			}
		},
		filters: {
			formatClass(allCheck) {
				return(allCheck === 'all') ? 'el-icon-check' : 'el-icon-minus';
			},
		},
		watch:{
			selectType(val){
				val && this.setList(this.menuList,val);
			}
		},
		methods: {
			setList(menuList, selectType) {
				if(!(menuList && menuList.length)) {
					return this.selectMenuList = [];
				}
				this.selectMenuList = menuList.map(item => {
					let itemCache = JSON.parse(JSON.stringify(item));
					let isAll = false;
					let isAct = true;
					if(itemCache.menu && itemCache.menu.length) {
						let checkCount = 0;
						itemCache.menu.forEach(its => {
							its.isAct = selectType.includes(its.type);
							if(its.isAct) {
								checkCount++;
							} else {
								isAct = false;
							}
						});
						isAll = checkCount!==0 && checkCount !== itemCache.menu.length;
					} else {
						isAct = selectType.includes(itemCache.type);
						isAll = false;
						itemCache.menu = [];
					}
					itemCache.isAct = isAct;
					itemCache.isAll = isAll;
					return itemCache;
				});
				this.setSelect();
			},
			setSelect() {
				let val = JSON.parse(JSON.stringify(this.selectMenuList));
				if(val) {
					let str = [];
					val.forEach(item => {
						let isMenu = item.menu && item.menu.length;
						if(item.isAct) {
							if(isMenu) {
								item.menu.forEach(its => {
									str.push(its);
								})
							} else {
								str.push(item);
							}
						} else {
							if(isMenu) {
								item.menu.forEach(its => {
									its.isAct && str.push(its);
								});
							}
						}
					});
					this.selectStr = str;
				}else{
					this.selectStr = [];
				}
			},
			showSelect() {
				this.setList(this.menuList, this.selectType);
				
			},
			hideSelect() {

			},
			selectMenuHandle(item, ind) {
				let cache = this.selectMenuList;
				let isAct = item.isAct;
				let isMenu = item.menu && item.menu.length;
				cache[ind].isAll = false;
				if(isMenu) {
					if(isAct) {
						cache[ind].menu.forEach(its => {
							its.isAct = true;
						});
					} else {
						cache[ind].menu.forEach(its => {
							its.isAct = false;
						});
					}
					this.selectMenuList = cache;
				}
				this.setSelect();
				this.$emit('update:selectType', (this.selectStr && this.selectStr.length) ? this.selectStr.map(item => item.type) : []);				
				
			},
			changeSubSelect(sub, ind) {
				let isAct = this.selectMenuList[ind].menu.every(item => item.isAct);
				this.selectMenuList[ind].isAll = isAct ? false : (this.selectMenuList[ind].menu.find(item => item.isAct) ? true : false);
				this.selectMenuList[ind].isAct = isAct;
				this.setSelect();
				this.$emit('update:selectType', (this.selectStr && this.selectStr.length) ? this.selectStr.map(item => item.type) : []);								
			},
			cleanSelect() {
				this.$emit('update:selectType', []);
			},
			submitSelect() {
				this.$emit('update:selectType', (this.selectStr && this.selectStr.length) ? this.selectStr.map(item => item.type) : []);
				this.closePopover();
			},
			closePopover(){
				this.selectPopoverVisible = false;
			}
		}
	}
</script>
<style>
	.complex-select-tooltip {
		max-width: 300px;
	}
	
	.complex-select-tooltip .complex-select-tooltip_content {
		line-height: 1.8;
	}
	
	.simulate-select-popover {
		padding: 5px 0;
	}
</style>
<style lang="scss" scoped>
	.complex-select {
		display: inline-block;
		font-size: 12px;
		width: 190px;
		position: relative;
		top: 6px;
		i.el-icon-close {
			cursor: pointer;
			font-size: 12px;
			border-radius: 50%;
			overflow: hidden;
			background: #c0c4cc;
			color: #fff;
			&:hover {
				background: #909399;
			}
		}
	}
	
	.simulate-select {
		line-height: 20px;
		height: 30px;
		box-sizing: border-box;
		border: 1px solid #dcdfe6;
		border-radius: 4px;
		padding: 4px 5px;
		padding-right: 20px;
		width: 100%;
		display: inline-block;
		min-width: 180px;
		overflow: hidden;
		position: relative;
		.simulate-select-inner {
			overflow: hidden;
			width: 100%;
			height: 100%;
			.simulate-select-inner-content {
				height: 100%;
				width: auto;
				white-space: nowrap;
				overflow: hidden;
				overflow-x: auto;
				&::-webkit-scrollbar {
					display: none;
				}
				&:empty:before {
					content: attr(placeholder);
					color: #cfcfcf;
				}
				&:focus:before {
					content: none;
				}
			}
			.simulate-select-checked-item {
				display: inline-block;
				box-sizing: border-box;
				border-color: transparent;
				margin: 0 2px;
				background-color: #f0f2f5;
				height: 20px;
				line-height: 20px;
				width: auto;
				border-radius: 4px;
				overflow: hidden;
				padding: 0 5px;
				color: #909399;
				i {
					vertical-align: -2px;
				}
			}
		}
		.simulate-select-suf {
			position: absolute;
			right: 5px;
			top: 50%;
			font-size: 14px;
			transform: translateY(-50%);
			i {
				vertical-align: -2px;
			}
		}
	}
	
	.select-box {
		background: #fff;
		height: auto;
		padding: 0 3px;
		overflow: hidden;
		.select-menu-list {
			height: 100%;
			width: 390px;
			padding-bottom: 5px;
			overflow-y: auto;
			@include scrollbar();
			&::-webkit-scrollbar-track {
				background: transparent;
				-webkit-box-shadow: none;
			}
			li {
				width: 33.333%;
				font-size: 12px;
				padding: 0;
				padding-left: 20px;
				position: relative;
				color: #606266;
				height: 34px;
				line-height: 34px;
				box-sizing: border-box;
				cursor: pointer;
				overflow: hidden;
				/deep/ .el-checkbox__label {
					@include oneLineEsp(84px);
					vertical-align: -6px;
				}
			}
		}
		.btn-row {
			text-align: center;
			padding: 8px 15px;
			border-top: 1px solid #fff;
		}
	}
	
	.simulate-select-subMenu-content {
		max-width: 170px;
		overflow: hidden;
		line-height: 28px;
		.simulate-select-subMenu-content-head {}
		.el-checkbox {
			display: block;
			/deep/ .el-checkbox__label {
				@include oneLineEsp(140px);
				vertical-align: -5px;
			}
		}
	}
</style>
