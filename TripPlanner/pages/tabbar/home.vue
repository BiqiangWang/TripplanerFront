<template>
	<view class="app">
		<!-- 顶部栏 -->
		<!-- #ifndef MP-WEIXIN -->
		<page-header :showFillView="false"></page-header>
		<mescroll-body
			ref="mescrollRef" 
			@init="mescrollInit" 
			:top="headerHeight" 
			@down="downCallback" 
			:up="upOption" 
			@up="loadHotList" 
		>
		<!-- #endif -->
		<!-- #ifdef MP-WEIXIN -->
		<!-- <page-header :showFillView="true"></page-header> -->
		<!-- #endif -->
			<!-- 头部轮播图 -->
			<banner :list="carousel"></banner>
			<view class="hot-wrap" :class="{show: loaded}">
				<view class="floor-header row" @click="navTo('/pages/product/list?isHot=1')" v-show="hasLogin">
					<image class="icon" src="/static/icon/hot.png"></image>
					<view class="column fill">
						<text class="tit">智能推荐</text>
						<text>Intelligent Recommendation</text>
					</view>
					<text class="mix-icon icon-you"></text>
				</view>
				
				<product-list ref="productList" :list="hotList"></product-list>
			</view>
			
		<!-- #ifndef MP-WEIXIN -->
		</mescroll-body>
		<!-- #endif -->
		<!-- #ifdef MP-WEIXIN -->
		<!-- <mix-load-more :status="loadingType"></mix-load-more> -->
		<!-- #endif -->
		
		<!-- 首页优惠券弹窗 -->
		<home-advert-modal></home-advert-modal>
	</view>
</template>

<script>
	import {mapState, mapGetters} from 'vuex'
	import tabbarMixin from './mixin/tabbar' 
	import homeMixin from './mixin/home'
	import pageHeader from './components/page-header.vue'
	import banner from './components/banner.vue'
	import productList from '@/pages/product/components/product-list.vue'
	import homeAdvertModal from './components/home-advert-modal.vue'
	export default {
		components: {
			pageHeader,
			banner,
			productList,
			homeAdvertModal
		},
		mixins: [homeMixin, tabbarMixin],
		data() {
			return {
				navList: [],//导航列表
				advertList: [],//广告列表
				hotList: [{
							thumb: 'http://img1.qunarzz.com/sight/p0/2005/39/3979f1867defec4ea3.water.jpg_280x200_e1b47993.jpg',
							title: '八达岭长城',
							sales: '2307',
							price: '40',
							id: '22'
						},
						{
							thumb: 'http://img1.qunarzz.com/sight/p0/1602/92/920e47352552c1c990.water.jpg_280x200_cf5666bb.jpg',
							title: '天坛公园',
							sales: '3601',
							price: '10.5',
							id: '363'
						},
						{
							thumb: 'http://img1.qunarzz.com/sight/p64/201211/04/9173ff9f33e97f3193835fbb.jpg_280x200_9537b05a.jpg',
							title: '杜甫草堂',
							sales: '542',
							price: '47',
							id:'541'
						},
						{
							thumb: 'http://img1.qunarzz.com/sight/p0/1603/97/97a91e052b51aa9890.water.jpg_280x200_ba2a9853.jpg',
							title: '天涯海角',
							sales: '2280',
							price: '50',
							id:'757'
						}
					],//热门推荐
			}
		},
		computed: {
			...mapState(['userInfo', 'orderCount', 'couponCount']),
			...mapGetters(['hasLogin']),
			midAdvert(){
				if(this.advertList.length === 0) return {};
				const res = this.advertList.filter(item=> item.advert_type === 'middle');
				return res.length > 0 ? res[0]: {};
			},
			carousel(){
				return this.advertList.filter(item=> item.advert_type === 'carousel');
			}
		},
		onLoad() {
			this.loadAdvert();
			this.loadNavList();
			
			setTimeout(()=>{
				//this.navTo('/pages/address/list')
			}, 1000)
		},
		onShow(){
			this.$store.dispatch('getUserInfo'); 
			console.log(this.userInfo);
			console.log(this.hasLogin);
			if (this.userInfo == {}) {
				this.hasLogin = false;
			}
			// this.$store.dispatch('getOrderCount'); //更新订单数量
			// this.$store.dispatch('getCouponCount'); //更新优惠券数量
		},
		methods: {
			//加载广告 缓存10分钟
			async loadAdvert(){
				const res = await this.$request('advert', 'getAdvertList', {}, {
					cache: 10*60
				});
				this.advertList = res.data;
				this.log(res);
			},
			//加载导航 缓存1小时
			async loadNavList(){
				const res = await this.$request('advert', 'getNavList', {}, {
					cache: 60*60*0,
				});
				this.navList = res.data;
			},
		}
	}
</script>

<style>
	page{
		background-color: #f7f7f7;
	}
</style>
<style scoped lang="scss">
	/* 分类 */
	.cate-section {
		display: flex;
		align-items: center;
		flex-wrap:wrap;
		padding: 10rpx 16rpx; 
		background: #fff;
		
		.item {
			display: flex;
			flex-direction: column;
			align-items: center;
			width: 20%;
			padding: 20rpx 0;
			font-size: 24rpx;
			color: #333;
		}
		.icon {
			width: 84rpx;
			height: 84rpx;
			margin-bottom: 14rpx;
			border-radius: 50%;
		}
	}
	.mid-ad{
		width: 100%;
		height: 208rpx;
		padding: 0 20rpx 20rpx;
		background: #fff;
		
		image{
			width:100%;
			height: 100%; 
		}
	}
	.floor-header{
		padding: 6rpx 30rpx 8rpx 24rpx;
		font-size: 24rpx;
		color: #999;
		background-color: #fff;
		
		.icon{
			flex-shrink: 0;
			width: 80rpx;
			height: 80rpx;
			margin-right: 20rpx;
		}
		.tit-box{
			flex: 1;
			display: flex;
			flex-direction: column;
		}
		.tit{
			margin-bottom: 10rpx;
			font-size: 32rpx;
			color: #333;
			font-weight: 700;
		}
		.icon-you{
			font-size: 28rpx;
			color: #999;
		}
	}
	.hot-wrap{
		padding-top: 20rpx;
		background: linear-gradient(to bottom, #fff 10rpx, #f7f7f7);
		//opacity: 0;
		transition: opacity .2s;
		
		&.show{
			opacity: 1;
		}
		.floor-header{
			padding-bottom: 30rpx;
		}
		/deep/ .list-item{
			box-shadow: 4rpx 0 10rpx rgba(0,0,0,.06);
		}
	}
	
</style>
