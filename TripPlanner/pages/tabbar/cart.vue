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
			<!-- 列表 -->
			
			<view class="user-wrapper" v-if="!hasLogin">
				<view class="login-box" @click="navTo('/pages/auth/login')">
					<image src="/static/empty/address.png" style="margin-left: 30px; margin-top:50px;"></image>
				</view>
				<text style="margin-left: 150px; margin-top: 20px; text-align: center; color: #8F8F94;">请登录</text>
			</view>
			<uni-swipe-action>
				<uni-swipe-action-item v-if="hackReset" v-for="(item, index) in list" :key="item._id" :options="options" @click="remove($event,index)" >
					<view class="item row" style="margin-top: 10px;">
						<text class="mix-icon icon-xuanzhong" :class="{active: item.checked}" @click.stop.prevent="checkRow(item)"></text>
						<view class="image-wrapper lazyload lazypic" :class="{loaded: item.loaded}" @click="navTo('/pages/route/routeInfo?id='+item.id)">
							<image :src="item.sights[0].picUrl" mode="aspectFill" lazy-load="true" @load="imageOnLoad(item)" ></image>
						</view>
						<view class="right column">
							<text class="title clamp">{{item.name}}</text>
							<!-- <text class="sku">{{item.sku.name}}</text> -->
							<!-- <text class="price">¥{{item.price || ''}}</text> -->
							<!-- <view class="number-box" @click.stop.prevent="stopPrevent">
								<mix-number-box :min="1" :max="item.stock" :value="item.number" :isMax="item.number >= item.stock" :isMin="item.number === 1" :index="index" @eventChange="onNumberChange" ></mix-number-box>
							</view> -->
						</view>
					</view>
				</uni-swipe-action-item>
			</uni-swipe-action>
			<!-- 失效商品 -->
			<!-- <view v-if="invalidList.length > 0" class="invalid">
				<view class="invalid-header row" style=" height: 72rpx; padding: 0 24rpx 0 22rpx; font-size: 24rpx; color: #333; background-color: #f5f5f5; ">
					<text class="mix-icon icon-tishi"></text>
					<text class="fill">商品调整、库存不足等原因会导致商品失效</text>
					<view class="btn center" @click="clearInvalid">
						<text class="mix-icon icon-lajitong"></text>
						<text>清除</text>
					</view>
				</view>
				<view v-for="(item, index) in invalidList" :key="item._id" class="item row">
					<text class="mix-icon icon-xuanzhong"></text>
					<view class="image-wrapper lazyload lazypic" :class="{loaded: item.loaded}">
						<image :src="item.image" mode="aspectFill" lazy-load="true" @load="imageOnLoad(item)" ></image>
					</view>
					<view class="right column">
						<text class="title clamp">{{item.title}}</text>
						<text class="sku">{{item.sku.name}}</text>
						<view class="row">
							<text class="price fill">¥{{item.price || ''}}</text>
							<view class="tag">失效</view>
						</view>
					</view>
				</view>
			</view> -->
			<!-- 底部栏 -->
			<view class="bot-fill-view"></view>
		<!-- </mescroll-body> -->
		
		<!-- <view v-if="loaded && (list.length > 0 || invalidList.length > 0)" class="bottom row"> -->
		<view class="bottom row">
			<!-- <text class="mix-icon icon-xuanzhong" :class="{active: allChecked}" @click="checkAll"></text>
			<text v-if="!allChecked" class="check-tip">全选</text>
			<view class="del-btn center" :class="{active: allChecked}" @click="showPopup('mixModal')">
				<text>清空</text>
			</view> -->
			<text class="price fill"></text>
			<view class="btn center" @click="navTo('/pages/address/limit')">
				<text>新路线</text>
			</view>
		</view>
		<!-- 缺省 -->
		<!-- <mix-empty v-else-if="loaded" type="cart"></mix-empty> -->
		<!-- 加载 -->
		<!-- <mix-loading v-if="isLoading" :type="loaded ? 1 : 2" :mask="true"></mix-loading> -->
		<!-- 确认对话框 -->
		<mix-modal ref="mixModal" title="删除提示" text="真的要狠心清空吗" confirmText="清空" @onConfirm="clear"></mix-modal>
	</view>
</template>

<script>	
	import {mapState, mapGetters} from 'vuex'
	import tabbarMixin from './mixin/tabbar' 
	import MescrollMixin from "@/components/mescroll-uni/mescroll-mixins.js";
	export default {
		mixins: [tabbarMixin, MescrollMixin], 
		data() {
			return {
				hackReset:true,
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
						onShow: true,
					},
					noMoreSize: 50,
				},
				list: [],
				invalidList: [],
				totalPrice: 0,
				userid:"",
			}
		},
		computed: {
			...mapState(['userInfo', 'orderCount', 'couponCount']),
			...mapGetters(['hasLogin']),
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
		onLoad() {
			uni.$on('refreshCart', ()=>{
				this.list = [];
				this.invalidList = [];
				this.mescroll.resetUpScroll(false)
			})
		},
		onShow() {
			
			this.list = [];
			this.$store.dispatch('getUserInfo');
			this.userid = this.userInfo.id;
			console.log(this.userid)
			this.getRouteList();
		},
		// onShow() {
		// 	this.mescroll && this.mescroll.resetUpScroll(false)
		// },
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
			getRouteList() {
				console.log("333");
				console.log(this.userid);
				uni.request({
					url: "http://47.102.212.4:8082/search/getroute", //请求接口
					data: {
						userId: this.userid,
					},
					header:{'content-type':'application/x-www-form-urlencoded'},
					method: 'GET',
					success: (res) => { //请求成功后返回
						console.log(res.data.data);
						console.log(res.data.data[0].sights[0].picUrl);
						if (res.statusCode === 200) {
							this.list = res.data.data;
							this.hackReset = false;
							this.$nextTick(() => {
								this.hackReset = true;
							})
						} else {
							uni.showToast({
								title: '信息请求错误',
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
