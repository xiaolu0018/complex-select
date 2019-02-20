<template>
	<div class="complex-select" ref="complexSelect">
		<div v-popover:selectPopover class="simulate-select"  :class="{'is-open':isOpen}">
			<ul class="simulate-select-inner" :placeholder="placeholder">
				<li class="simulate-select-checked-item pull-left" v-for="item in selectStr">
					<span>{{item.desc}}</span>
					<i class='el-icon-circle-close' @click.stop="delSelect(item.type)"></i>
				</li>
			</ul>
			<span class="simulate-select-suf">
					<i class="el-icon-arrow-down"></i>
				</span>
		</div>
		<el-popover ref="selectPopover" popper-class="simulate-select-popover" placement="bottom-start" trigger="click" @show="showSelect" :append-to-body="false" @hide="hideSelect">
			<div class="select-box">
				<div class="primary-menu select-menu-list">
					<ul>
						<li :class="{'is-active':(item.isAct!=='none')}" @click="selectMenuHandle(item,ind)" v-for="(item,ind) in selectMenuList" :key="item.type">
							<span class="menu-content">{{item.desc}}</span>
							<i :class="item.isAct | formatClass"></i>
						</li>
					</ul>
				</div>
				<div class="sub-menu select-menu-list" v-show="subMenu.length">
					<ul>
						<li :class="{'is-active':(sub.isAct === 'all')}" @click="selectSubMenuHandle(sub,ind)" v-for="(sub,ind) in subMenu" :key="sub.type">
							<span class="menu-content">{{sub.desc}}</span>
							<i class="el-icon-check"></i>
						</li>
					</ul>
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
				isOpen: false,

				selectMenuList: [], //合成数据

				activeMenu: null, //当前激活一级菜单index
				subMenu: [], //二级菜单

				checkedArr: [], //选中menu
				selectStr: [], //展示选中字符串	
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
		computed: {

		},
		watch: {
			menuList: {
				handler: function(val) {
					this.$nextTick(() => {
						val.length && this.setList(val, this.selectType);
					});
				},
				　　　　deep: true
			},
			selectType: {
				handler: function(val) {
					this.$nextTick(() => {
						this.menuList && this.menuList.length && this.setList(this.menuList, val);
					});
				},
				　　　　deep: true
			},
		},
		filters: {
			formatClass(allCheck) {
				return(allCheck === 'all') ? 'el-icon-check' : 'el-icon-minus';
			},
		},
		methods: {
			setList(menuList, selectType) {
				this.selectStr = [];
				this.checkedArr = [];
				this.selectMenuList = menuList.map(item => {

					let itemCache = Object.assign({}, item);

					let isAll = false; //子菜单是否有选中
					if(itemCache.menu && itemCache.menu.length) {
						itemCache.menu.forEach(its => {
							let isAct = selectType.includes(its.type) ? 'all' : 'none';
							its.isAct = isAct;

							if(isAct === 'all') {
								this.selectStr.push(its);
								this.checkedArr.push(its.type);
								isAll = true;
							}
						});
						if(isAll) {
							isAll = itemCache.menu.every(ites => ites.isAct === 'all') ? 'all' : 'some';
						} else {
							isAll = 'none';
						}
					} else {
						isAll = selectType.find(ind => ind == itemCache.type) ? 'all' : 'none';
					}

					itemCache.isAct = isAll;
					if(itemCache.isAct !== 'none') {
						this.selectStr.push({
							type: item.type,
							desc: item.desc
						});
						this.checkedArr.push(item.type);
					}
					return itemCache;
				});
				this.$nextTick(()=>{
					this.$refs.selectPopover.$refs.popper.style.top = this.$refs.selectPopover.$refs.reference.offsetHeight + 'px';
				});
			},
			showSelect() {
				this.isOpen = true;
				this.subMenu = [];
			},
			hideSelect() {
				this.isOpen = false;
				this.subMenu = [];
			},
			selectMenuHandle(item, index) {
				let cache = this.checkedArr.concat();

				let isAct = item.isAct === 'none';

				let isMenu = item.menu && item.menu.length;

				if(isAct) {
					cache = cache.concat(item.type);
					isMenu && (cache = cache.concat(item.menu.map(its => its.type)));
				} else {
					cache = [...new Set(cache)];
					cache.splice(cache.findIndex(sit => sit == item.type), 1);

					isMenu && item.menu.forEach(its => {
						cache.splice(cache.findIndex(sit => sit == its.type), 1)
					});
				};
				this.$emit('update:selectType', [...new Set(cache)]);
				if(isMenu) {
					this.subMenu = item.menu;
				} else {
					this.subMenu = [];
				}

			},
			selectSubMenuHandle(item, index) {
				let cache = this.checkedArr.concat();
				let isAct = item.isAct === 'none';
				if(isAct) {
					cache = cache.concat(item.type);
				} else {
					cache = [...new Set(cache)];
					cache.splice(cache.findIndex(sit => sit == item.type), 1);
				}
				cache = [...new Set(cache)];

				this.$emit('update:selectType', cache);
			},
			delSelect(type) {
				let cache = this.checkedArr.concat();
				cache.splice(cache.findIndex(sit => sit == type), 1);
				this.$emit('update:selectType', cache);
			}
		}
	}
</script>
<style>
	.simulate-select-popover {
		padding: 5px 0;
	}
</style>
<style lang="scss" scoped>
	.complex-select {
		display: inline-block;
		font-size: 12px;
		width: 190px;
		position:relative;
		top:6px;
	}
	
	.simulate-select {
		line-height: 20px;
		min-height: 30px;
		height: auto;
		box-sizing: border-box;
		border: 1px solid #dcdfe6;
		border-radius: 4px;
		padding: 4px 15px;
		padding-right: 20px;
		width: 100%;
		display: inline-block;
		min-width: 180px;
		overflow: hidden;
		position: relative;
		.simulate-select-inner {
			overflow: hidden;
			width: auto;
			&:empty:before {
				content: attr(placeholder);
				color: #cfcfcf;
			}
			&:focus:before {
				content: none;
			}
			.simulate-select-checked-item {
				display: block;
				box-sizing: border-box;
				border-color: transparent;
				margin: 2px 2px;
				background-color: #f0f2f5;
				height: 20px;
				line-height: 20px;
				width: auto;
				border-radius: 4px;
				overflow: hidden;
				padding: 0 5px;
				color: #909399;
				span {
					vertical-align: mmiddle;
				}
				i {
					cursor: pointer;
					font-size: 13px;
					vertical-align: mmiddle;
					color: #909399;
					&:hover {
						color: #66686b;
					}
				}
			}
		}
		.simulate-select-suf {
			position: absolute;
			right: 5px;
			top: 50%;
			font-size: 14px;
			transform: translateY(-50%) rotateZ(0deg);
			transition: transform .3s;
		}
		&.is-open .simulate-select-suf {
			transform: translateY(-50%) rotateZ(180deg);
		}
	}
	
	.select-box {
		background: #fff;
		height: 257px;
		padding: 0 3px;
		overflow: hidden;
		display: flex;
		flex-direction: row;
		.select-menu-list {
			height: 100%;
			width: 190px;
			overflow-y: auto;
			@include scrollbar();
			&::-webkit-scrollbar-track {
				background: transparent;
				-webkit-box-shadow: none;
			}
			li {
				font-size: 12px;
				padding: 0 20px;
				position: relative;
				color: #606266;
				height: 34px;
				line-height: 34px;
				box-sizing: border-box;
				cursor: pointer;
				display: flex;
				overflow: hidden;
				flex-direction: row;
				flex-wrap: nowrap;
				justify-content: space-between;
				align-items: center;
				.menu-content {
					@include oneLineEsp(calc(100% - 15px));
				}
				i.el-icon-check,
				i.el-icon-minus {
					display: none;
					color: #409EFF;
				}
				&:hover {
					background: #f5f7fa;
				}
				&.is-active {
					.menu-content {
						color: #409EFF;
					}
					i.el-icon-check,
					i.el-icon-minus {
						display: inline-block;
					}
				}
			}
		}
	}
</style>