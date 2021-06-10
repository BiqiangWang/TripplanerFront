<template>
	<view class="app">
		<!-- <mescroll-body
			ref="mescrollRef" 
			@init="mescrollInit" 
			:top="0" 
			@down="downCallback" 
			:up="upOption" 
			@up="loadList" 
		> -->
			<view class="infocontainer">
				<view class="kongbai"></view>
				<view class="infocontainer2">
					<view style="display: flex;">
						<view class="routename">{{ routename }}</view>
						<image style="width: 60rpx;height: 60rpx;margin-left: 20rpx;margin-top: 6rpx;" src="../../static/tplaner/editor.png"></image>
					</view>
					<view class="routedes">{{ routedes }}</view>
				</view>
			</view>
			<view class="detail-desc">
				<view class="d-header center">
					<text>景点列表</text>
				</view>
			</view>
			<view class="kongbai2"></view>
			<!-- 列表 -->
			<uni-swipe-action>
				<uni-swipe-action-item v-for="(item, index) in list" :key="item._id" :options="options" @click="remove($event,index)" >
					<view class="item row" style="margin-bottom: 20rpx;">
						<text class="mix-icon icon-xuanzhong" :class="{active: item.checked}" @click.stop.prevent="checkRow(item)"></text>
						<view class="image-wrapper lazyload lazypic" :class="{loaded: item.loaded}" @click="navTo('/pages/product/detail?id='+item.sid)">
							<image :src="item.picUrl" mode="aspectFill" lazy-load="true" @load="imageOnLoad(item)" ></image>
						</view>
						<view class="right column">
							<text class="title clamp">{{item.name}}</text>
						</view>
					</view>
				</uni-swipe-action-item>
			</uni-swipe-action>
			<!-- 底部栏 -->
			<view class="bot-fill-view"></view>
		<!-- </mescroll-body> -->
		
		<!-- <view v-if="loaded && (list.length > 0 || invalidList.length > 0)" class="bottom row"> -->
		<!-- <view class="bottom row">
			<text class="mix-icon icon-xuanzhong" :class="{active: allChecked}" @click="checkAll"></text>
			<text v-if="!allChecked" class="check-tip">全选</text>
			<view class="del-btn center" :class="{active: allChecked}" @click="showPopup('mixModal')">
				<text>清空</text>
			</view>
			<text class="price fill"></text>
			<view class="btn center" @click="createOrder">
				<text>新路线</text>
			</view>
		</view> -->
		<!-- 缺省 -->
		<!-- <mix-empty v-else-if="loaded" type="cart"></mix-empty> -->
		<!-- 加载 -->
		<!-- <mix-loading v-if="isLoading" :type="loaded ? 1 : 2" :mask="true"></mix-loading> -->
		<!-- 确认对话框 -->
		<!-- <mix-modal ref="mixModal" title="删除提示" text="真的要狠心清空吗" confirmText="清空" @onConfirm="clear"></mix-modal> -->
	</view>
</template>

<script>
	import tabbarMixin from '@/pages/tabbar/mixin/tabbar' 
	import MescrollMixin from "@/components/mescroll-uni/mescroll-mixins.js";
	export default {
		mixins: [tabbarMixin, MescrollMixin], 
		data() {
			return {
				routeid:"60c20b6b3793a165210af32c",
				routename:"路线名称",
				routedes:"路线描述",
				options: [{
					text: '删除',
					style: {
						backgroundColor: '#ff536f'
					}
				}],
				upOption:{
					auto: false, // 是否自动加载
					page: {
					 	num: 0, // 当前页码,默认0,回调之前会加1,即callback(page)会从1开始
					 	size: 15 // 每页数据的数量
					},
					empty: {
						onShow: false,
					},
					noMoreSize: 50,
				},
				list: [
					// {
					// 		image: 'http://img1.qunarzz.com/sight/p0/2005/39/3979f1867defec4ea3.water.jpg_280x200_e1b47993.jpg',
					// 		title: '路线1',
					// 		sales: '2307',
					// 		price: '40'
					// 	},
					// 	{
					// 		image: 'http://img1.qunarzz.com/tuan/team2/1507/2c/83e0e0e7ae082a.jpg_280x200_8c8e548a.jpg',
					// 		title: '路线2',
					// 		sales: '2307',
					// 		price: '69.9'
					// 	},
					// 	{
					// 		image: 'http://img1.qunarzz.com/sight/p64/201211/04/9173ff9f33e97f3193835fbb.jpg_280x200_9537b05a.jpg',
					// 		title: '路线3',
					// 		sales: '2307',
					// 		price: '47'
					// 	},
						],
				invalidList: [],
				totalPrice: 0,
			}
		},
		computed: {
			allChecked(){
				return this.list.length > 0 && this.list.every(item=> item.checked);
			},
			confirmNumber(){
				return this.list.filter(item=> item.checked).length;
			},
		},
		watch: {
			list(){
				this.calcTotalPrice();
			}
		},
		onLoad(options) {
			this.routeid=options.id
			this.getRouteInfo()
			uni.$on('refreshCart', ()=>{
				this.list = [];
				this.invalidList = [];
				this.mescroll.resetUpScroll(false)
			})
		},
		onShow() {
			this.mescroll && this.mescroll.resetUpScroll(false)
		},
		methods: {
			async loadList(type){
				const res = await this.$request('cart', 'get');
				const list = res.map(item=> {
					return {
						loaded: this.loaded,
						...item
					}
				})
				this.list = list.filter(item=> item.invalid === false);
				this.invalidList = list.filter(item=> item.invalid);
				
				this.mescroll.endSuccess(list.length); //结束加载状态
			},
			mescrollInit(mescroll){
				this.isLoading = true;
				this.mescroll = mescroll;
				this.mescroll.resetUpScroll(false)
			},
			//去结算 创建订单
			createOrder(){
				const ids = this.list.filter(item=> item.checked).map(item=> item._id);
				if(ids.length === 0){
					this.$util.msg('请选择结算商品');
					return;
				}
				this.navTo('/pages/order/createOrder?type=cart&ids='+ids);
			},
			//数量修改
			onNumberChange(e){
				this.$util.debounce(async ()=>{
					const res = await this.$request('cart', 'updateNumber', {
						id: this.list[e.index]._id,
						number: e.number
					})
					if(res.status == 1){
						this.list[e.index].number = e.number
						this.calcTotalPrice();
					}else{
						this.$util.msg('数量更新失败');
					}
				}, 500)
			},
			//单选
			checkRow(item){
				if(this.chenageChecked([item._id], !item.checked)){
					item.checked = !item.checked;
					this.calcTotalPrice();
				}
			},
			//全选
			checkAll(){
				const ids = this.list.map(item=> item._id);
				const allChecked = !this.allChecked;
				if(this.chenageChecked(ids, allChecked)){
					this.list.forEach(item=> {
						item.checked = allChecked;
					})
					this.calcTotalPrice();
				}
			},
			//修改选中状态
			async chenageChecked(ids, checked){
				this.$request('cart', 'updateCheck', {
					ids, 
					checked
				}, {showLoading: true}).then(res=>{
					if(res.status === 1){
						return true;
					}else{
						this.$util.msg('修改失败');
						return false;
					}
				})
			},
			//删除
			remove(e, index){
				this.$util.throttle(async ()=>{
					const res = await this.$request('cart', 'remove', {
						ids: [this.list[index]._id]
					}, {
						showLoading: true
					})
					this.$util.msg(res.msg);
					if(res.status === 1){
						this.list.splice(index, 1);
						this.$store.dispatch('getCartCount');//更新数量
					}
				})
			},
			//清空失效
			clearInvalid(){
				this.$util.throttle(async ()=>{
					const res = await this.$request('cart', 'remove', {
						ids: this.invalidList.map(item=> item._id)
					}, {
						showLoading: true
					})
					this.$util.msg(res.msg);
					if(res.status === 1){
						this.invalidList = [];
						this.$store.dispatch('getCartCount');//更新数量
					}
				})
			},
			//清空商品
			clear(){
				this.$util.throttle(async ()=>{
					const res = await this.$request('cart', 'removeAll', {}, {
						showLoading: true
					})
					this.$util.msg(res.msg);
					if(res.status === 1){
						this.list = [];
						this.$store.dispatch('getCartCount');//更新数量
					}
				})
			},
			//计算总价
			calcTotalPrice(){
				const checkedList = this.list.filter(item=> item.checked);
				let total = 0;
				checkedList.forEach(item=> {
					console.log(item);
					total += item.price * item.number
				})
				this.totalPrice = total;
			},
			getRouteInfo(){
				uni.request({
				   url: "http://47.102.212.4:8082/search/getrouteSight", //请求接口
				   data:{
					   routeId: this.routeid,
				   },
				   method: 'GET',
				success: (res) => {//请求成功后返回
						if (res.data.code === 20000){
							this.routename = res.data.data.name;
							this.routedes = res.data.data.desc;
							this.list = res.data.data.sights;
							console.log(this.list)
							uni.showToast({
								title: '成功获取路线',
								duration: 2000
							});
						}else{
							uni.showToast({
								title: '路线请求错误',
								duration: 2000
							});		
						}
				   }
				});
			}
		}
	}
</script>

<style scoped lang="scss">
	.app{
		
		/deep/ .uni-swipe{
			padding: 24rpx 0;
		}
	}
	.infocontainer{
		width: 100%;
		background-color: #eaeaea;
		height: 180rpx;
	}
	.infocontainer2{
		width: 95%;
		margin-left: 2.5%;
		background-color: white;
		border-radius: 15rpx;
	}
	.kongbai{
		height: 10rpx; 
	}
	.kongbai2{
		height: 10rpx; 
		background-color: #eaeaea;
		margin-bottom: 20rpx;
	}
	.routename{
		font-size: 44rpx;
		margin-left: 30rpx;
		margin-bottom: 20rpx;
		font-weight: 700px;
	}
	.routedes{
		font-size: 30rpx;
		margin-left: 40rpx;
		margin-bottom: 40rpx;
		color: #8f8f94;
		height: 80rpx;
		
	}
	/*  详情 */
	.detail-desc {
		margin-top: 12rpx;
		background: #fff;
	
		.d-header {
			height: 80rpx;
			font-size: 30rpx;
			color: #333;
	
			text {
				margin: 0 20rpx;
			}
	
			&:before,
			&:after {
				content: '';
				width: 60rpx;
				border-bottom: 1px solid #ccc;
			}
		}
	}
	
	.item{
		width: 100%;
		padding-right: 30rpx;
		overflow: hidden;
		
		.icon-xuanzhong{
			padding: 60rpx 20rpx;
			font-size: 36rpx;
			color: #ddd;
			
			&.active{
				color: $base-color;
			}
		}
		.image-wrapper{
			flex-shrink: 0;
			width: 170rpx;
			height: 170rpx;
			margin-right: 20rpx;
			border-radius: 10rpx;
			overflow: hidden;
			
			image{
				width: 100%;
				height: 100%;
			}
		}
		.right{
			flex: 1;
			overflow: hidden;
		}
		.title{
			font-size: 30rpx;
			line-height: 42rpx;
		}
		.sku{
			min-height: 20rpx;
			margin: 20rpx 0 28rpx;
			font-size: 24rpx;
			color: #999;
		}
		.price{
			margin-bottom: 4rpx;
			font-size: 30rpx;
			color: #333;
		}
		.number-box{
			position: absolute;
			right: 0;
			bottom: 0;
			padding: 20rpx 30rpx 0;
		}
		/deep/{
			.uni-numbox-minus, .uni-numbox-value, .uni-numbox-plus{
				height: 48rpx;
			}
		}
	}
	.invalid{
		
		.item{
			padding: 24rpx 24rpx 24rpx 0;
			opacity: 0.5;
		}
		.invalid-header{
			height: 72rpx;
			padding: 0 24rpx 0 22rpx;
			font-size: 24rpx;
			color: #333;
			background-color: #f5f5f5;
		}
		.icon-tishi{
			margin-right: 10rpx;
			font-size: 32rpx;
			color: $base-color;
		}
		.btn{
			padding: 8rpx 16rpx;
			font-size: 24rpx;
			color: #666;
		}
		.icon-lajitong{
			margin-right: 8rpx;
			font-size: 32rpx;
		}
		.tag{
			padding: 6rpx 18rpx;
			font-size: 24rpx;
			color: #fff;
			border-radius: 100rpx;
			background-color: #555;
		}
	}
	.bot-fill-view{
		width: 100%;
		height: 110rpx;
		box-sizing: content-box;
		padding-bottom: constant(safe-area-inset-bottom);
		padding-bottom: env(safe-area-inset-bottom); 
	}
	.bottom{
		position: fixed;
		left: 0;
		bottom: var(--window-bottom);
		z-index: 90;
		width: 100%;
		height: 100rpx;
		background-color: #fff;
		box-shadow: 0 -2rpx 10rpx 0 rgba(0,0,0,.06);
		box-sizing: content-box;
		padding-bottom: constant(safe-area-inset-bottom);
		padding-bottom: env(safe-area-inset-bottom); 
		
		.icon-xuanzhong{
			margin-left: 20rpx;
			font-size: 48rpx;
			color: #ddd;
			position: relative;
			z-index: 10;
			background-color: #fff;
			border-radius: 100rpx;
			
			&.active{
				color: $base-color;
			}
		}
		.check-tip{
			position: absolute;
			left: 80rpx;
			font-size: 28rpx;
			color: #333;
		}
		.del-btn{
			width: 0rpx;
			height: 44rpx;
			padding-left: 14rpx;
			font-size: 28rpx;
			color: #fff;
			background-color: #C0C4CC;
			border-radius: 0 100rpx 100rpx 0;
			position: relative;
			left: -24rpx;
			transition: width .2s;
			
			&.active{
				width: 110rpx;
			}
		}
		.price{
			margin-right: 30rpx;
			font-size: 34rpx;
			color: $base-color;
			font-weight: 700;
			text-align: right;
		}
		.btn{
			min-width: 180rpx;
			height: 70rpx;
			padding: 0 26rpx;
			margin-right: 20rpx;
			border-radius: 100rpx;
			background-color: $base-color;
			font-size: 30rpx;
			color: #fff;
		}
	}
</style>
