<template>
	<view class="app">
		<pageHeader ref="pageHeader"></pageHeader>
		<!-- 轮播图 -->
		<!-- <view class="carousel-section">
			<view class="titleNview-placing"></view>
			<view class="titleNview-background" :style="{backgroundColor:titleNViewBackground}"></view>
			<swiper class="carousel" circular @change="swiperChange">
				<swiper-item v-for="(item, index) in carouselList" :key="index" class="carousel-item">
					<image :src="item.src" />
				</swiper-item>
			</swiper>
			<view class="swiper-dots">
				<text class="num">{{swiperCurrent+1}}</text>
				<text class="sign">/</text>
				<text class="num">{{swiperLength}}</text>
			</view>
		</view> -->
		<mix-swiper v-if="hackReset" :list="data.images"></mix-swiper>
		<view class="introduce column">
			<view class="sight_name">{{ sightname }}</view>
			<view class="info_container">
				<view class="scores">{{ score }}分</view>
				<view class="visitnum">{{ visitnumber }}条评价</view>
				<view class="level">{{ sightlevel }}</view>
			</view>
			<view class="introduction_container">
				<image class="introduction_logo" src="../../static/tplaner/tishi.png"></image>
				<view class="sight_introduction">{{ introduction }}</view>
			</view>
			<view class="address_container">
				<image class="address_logo" src="../../static/tplaner/map.png"></image>
				<view class="sight_address">{{ address }}</view>
			</view>
			<view class="price-wrap row">
				<mix-price-view :price="data.price" :size="40" style="margin-left: 20rpx;"></mix-price-view>
				<text v-if="data.market_price > data.price" class="m-price">￥{{ data.market_price }}</text>
				<text class="tag">人均</text>
				<!-- <view class="particulars">
					<uni-link href="http://piao.qunar.com/ticket/detail_2602391235.html?st=a3clM0QlRTUlQjklQkYlRTUlQjclOUUlMjZpZCUzRDIxMzQ3NCUyNnR5cGUlM0QwJTI2aWR4JTNEMjclMjZxdCUzRHJlZ2lvbiUyNmFwayUzRDIlMjZzYyUzRFdXVyUyNnVyJTNEJUU1JUI5JUJGJUU1JUI3JTlFJTI2bHIlM0QlRTQlQjglOEElRTYlQjUlQjc%3D#from=mpshouye_hotcity" text="查看详情" showUnderLine="false" color="#ff536f" copyTips="这是复制时显示的提示语" :showUnderLine="true" ></uni-link>
				</view> -->
			</view>
			<!-- <view class="bot row">
				<text class="fill">销量: {{ data.sales || 0 }}</text>
				<text class="fill">库存: {{ data.stock || 0 }}</text>
				<text class="fill">浏览量: {{ data.look_num || 0 }}</text>
			</view> -->
		</view>

		<!-- 打分 -->
		<!-- <view class="scoreBox">
			<text class="scoreTitle">为该景点打分：</text>
			<view class="uni-padding-wrap uni-common-mt star">
				<view class="uni-flex uni-row">
					<view :class="{starActive:item}" @click="choise(index)" class="flex-item iconfont"
						v-for="(item,index) in clicked_list">
						<view v-if="item" class="starIcon">
							&#xe601;
						</view>
						<view v-else class="starIcon">
							&#xe602;
						</view>
					</view>
				</view>
			</view>
		</view> -->
		<!-- 		<view class="commentbox">
			<textarea class="mycommentinput" auto-height="true" placeholder="请输入您的评价" v-model="comment"></textarea>
			<textarea class="mycommentinput" placeholder="请输入您的评价" maxlength="50" v-model="comment"></textarea>
			<image class="commitCommentButton" src="../../static/select.png" @click="commitComment"></image>
		</view> -->


		<!-- 评价 -->
		<view id="rating" class="rating-wrap column" :class="{'no-data': !ratingData.data}">
			<view class="e-header" @click="navTo('/pages/rating/rating?id='+data._id)">
				<text class="tit">景点评价</text>
				<text>({{ ratingData.count || 0 }})</text>
				<text class="tip">好评率 {{ data.rating_ratio || 100 }}%</text>
				<text class="mix-icon icon-you"></text>
			</view>
			<rating-item v-if="ratingData.data" :items="ratingData.data"></rating-item>
		</view>
		<view class="detail-desc">
			<view class="d-header center">
				<text>图文详情</text>
			</view>
			<jyf-parser ref="article" :html="data.content" lazy-load show-with-animation></jyf-parser>
			<!-- <rich-text :nodes="data.content"></rich-text> -->
		</view>
		<!-- 底部操作菜单 -->
		<bottom-operation :infoData="data" @onOprationClick="onOprationClick"></bottom-operation>
		<!-- loading -->
		<mix-loading v-if="isLoading" :mask="true"></mix-loading>
	</view>
</template>

<script>
	import jyfParser from '@/components/jyf-parser/jyf-parser.vue'
	import pageHeader from './components/detail-page-header' //页面头
	import mixSwiper from './components/mix-swiper' //轮播图
	import ratingItem from '@/pages/rating/components/rating-item' //评价
	import bottomOperation from './components/bottom-operation' //底部栏
	import sku from './components/sku' //sku面板
	let _anchorList = [];
	export default {
		components: {
			jyfParser,
			pageHeader,
			mixSwiper,
			ratingItem,
			bottomOperation,
			sku
		},
		data() {
			return {
				hackReset:true,
				titleNViewBackground: '',
				swiperCurrent: 0,
				swiperLength: 0,
				carouselList: [{
						src: "../../static/logo.png",
						background: "#ffffff",
					},
					{
						src: "../../static/select.png",
						background: "#ffffff",
					},
					{
						src: "../../static/logo.png",
						background: "#ffffff",
					},
				],
				sightid:1100,
				sightname: '八达岭长城',
				score:4.5,
				visitnumber:366,
				sightlevel:"",
				introduction: '一定要看那块“不到长城非好汉”碑',
				address:'重庆',
				clicked_list: [false, false, false, false, false], //对应星星个数
				currentSku: {},
				data: {
					_id:1,
					price:110,
					market_price:0,
					images: [{
							pic: 'http://img1.qunarzz.com/sight/p0/2005/39/3979f1867defec4ea3.water.jpg_280x200_e1b47993.jpg'
						},
					],
				},
				ratingData: {
					data: {
						user: {
							anonymous: false,
							lv: 300,
							avatar: '/static/icon/default-avatar.png',
							gender: 2,
							nickname: '飞飞',
							username: '飞飞',
						},
						rating: 5,
						content: 'good',
						date: "2021-5-1",
					},
					count: 3,
					rating_ratio: 66,
					Sight:{}
				}, //评价
				comment: '',
			};
		},
		onLoad(options) {
			this.id = options.id;
			this.loadRating(); //加载评价
		},
		onShow() {
			this.loadData();
		},
		onPageScroll(e) {
			this.$refs.pageHeader && this.$refs.pageHeader.pageScroll(e);
		},
		// #ifdef MP-WEIXIN
		onShareAppMessage(res) {
			return {
				title: this.data.title,
				path: '/pages/product/detail?id=' + this.data._id,
				imageUrl: this.data.thumb
			}
		},
		// #endif
		created:function(){
			this.getSightInfo()
			this.getcomment()
		},
		methods: {
			async loadData() {
				this.swiperLength = this.carouselList.length;
				const res = await this.$request('product', 'getDetail', {
					id: this.id
				})
				if (res.status === 0) {
					this.$util.msg(res.msg || '产品不存在或已下架');
					setTimeout(() => {
						uni.navigateBack();
					}, 1000)
					return;
				}
				const data = res.data;
				data.content = data.content.replace(/img src="/g,
					'img style="display:block;width:100%;height:auto" src="');
				this.data = data;
				this.$nextTick(() => {
					this.calcAnchor(); //计算锚点参数
				})
				//添加浏览历史
				this.addProductHistory();
			},
			//轮播图切换修改背景色
			swiperChange(e) {
				const index = e.detail.current;
				this.swiperCurrent = index;
				this.titleNViewBackground = this.carouselList[index].background;
			},
			// starIcon(item) {
			// 	if (item) {
			// 		return '&#xe601;'
			// 	} else {
			// 		return '&#xe602;'
			// 	}
			// },
			// //点击选择
			// choise(num) {
			// 	// num 为点击的星星在数组中的下标
			// 	this.clicked_list = [false, false, false, false, false];
			// 	num = num + 1;
			// 	for (let i = 0; i < num; i++) {
			// 		this.clicked_list[i] = true;
			// 	}
			// 	console.log(num);

			// },
			// commitComment() {
			// 	console.log(this.comment);
			// },
			//加载评价
			async loadRating() {
				const res = await this.$request('rating', 'getDetailRating', {
					product_id: this.id,
				})
				this.ratingData = res;
				this.$nextTick(() => {
					this.calcAnchor(); //计算锚点参数
				})
			},
			//加入购物车
			addToCart() {
				this.$util.throttle(async () => {
					const data = this.getConfirmData();
					if (!data) {
						return;
					}
					const res = await this.$request('cart', 'add', data, {
						showLoading: true,
						login: true
					})
					if (!res) {
						return;
					}
					this.log(res)
					this.$util.msg(res.msg);
					if (res.status === 1) {
						this.hidePopup('skuPopup');
						this.$store.dispatch('getCartCount'); //更新购物车数量
						uni.$emit('refreshCart'); //更新购物车
					}
				})
			},
			//立即购买
			buyNow() {
				const data = this.getConfirmData();
				if (!data) {
					return;
				}
				this.hidePopup('skuPopup');
				this.navTo('/pages/order/createOrder?data=' + JSON.stringify(data), {
					login: true
				})
			},
			//设置当前选择sku
			setCurrentSku(data) {
				this.currentSku = data;
			},
			//获取当前sku 如果没有sku返回默认规格
			getConfirmData() {
				const sku = this.currentSku._id ? this.currentSku : this.data.sku[0];
				if (sku.stock <= 0 || this.data.stock <= 0) {
					this.$util.msg('库存不足');
					return false;
				}
				const data = {
					product_id: this.data._id,
					number: this.$refs.skuPopup.buyNumber || 1,
					sku: this.currentSku._id ? this.currentSku : this.data.sku[0]
				}
				return data;
			},
			//计算锚点参数
			async calcAnchor() {
				const size = await new Promise(res => {
					uni.createSelectorQuery().select('#rating').boundingClientRect(data => {
						res(data);
					}).exec();
				});
				const headerHeight = this.systemInfo.statusBarHeight + this.systemInfo.navigationBarHeight;
				const a1 = (size ? size.top : 0) - headerHeight;
				const a2 = (size ? size.bottom : 0) + uni.upx2px(12) - headerHeight;
				this.$refs.pageHeader.anchorList[1].top = a1;
				this.$refs.pageHeader.anchorList[2].top = a2;
				_anchorList = [0, a1, a2];
			},
			//添加浏览历史
			addProductHistory() {
				const data = this.data;
				let list = uni.getStorageSync('productHistory');
				if (!list) {
					list = [];
				}
				const index = list.findIndex(item => item.id === data._id);
				if (index !== -1) {
					list.splice(index, 1);
				}
				list.unshift({
					id: data._id,
					thumb: data.thumb
				})
				uni.setStorageSync('productHistory', list);
			},
			//删除当前浏览历史
			delHistory() {
				let list = uni.getStorageSync('productHistory');
				if (!list) {
					return;
				}
				const index = list.findIndex(item => item._id === data._id);
				if (index === -1) {
					return;
				}
				list.splice(index, 1);
				uni.setStorageSync('productHistory', list);
			},
			showPopup(key, type) {
				this.$refs[key].open(type);
			},
			onOprationClick(type) {
				this.showPopup('skuPopup', type);
			},
			doAppShare() {
				this.$util.throttle(async () => {
					const data = {
						provider: "weixin",
						scene: 'WXSceneSession',
						type: 5,
						imageUrl: this.data.thumb,
						title: this.data.title,
						miniProgram: {
							id: 'gh_3dada2e0f833',
							path: '/pages/product/detail?id=' + this.data._id,
							type: 0,
							webUrl: 'http://guoyunnet.com'
						},
						success: res => {
							console.log("success:" + JSON.stringify(res));
						},
						fail: err => {
							console.log("fail:" + JSON.stringify(err));
						}
					}
					uni.share(data);
				})
			},
			getSightInfo(){
			   var _self = this;
			   uni.request({
				   url: "http://47.102.212.4:8092/sight/getInfo", //请求接口
				   data:{
					   sightId: this.sightid,
				   },
				   method: 'GET',
				success: (res) => {//请求成功后返回
						console.log(res.data.data.sightInfo.Sight)
						if (res.statusCode === 200){
							this.sightname = res.data.data.sightInfo.Sight.stname;
							this.introduction = res.data.data.sightInfo.Sight.desc;
							this.address = res.data.data.sightInfo.Sight.address;
							this.visitnumber = res.data.data.sightInfo.Sight.visitnum;
							this.score = res.data.data.sightInfo.Sight.starlevel.substr(0,3);
							this.data.price = res.data.data.sightInfo.Sight.price;
							if(this.data.price == ""){
								this.data.price = 0;
							}
							this.sightlevel = res.data.data.sightInfo.Sight.level;
							if(this.sightlevel == "无"){
								this.sightlevel = "普通景区";
							}
							this.data.images[0].pic = res.data.data.sightInfo.Sight.picUrl;
							this.hackReset = false;
							  this.$nextTick(() => {
							       this.hackReset = true;
							  })
						}else{
							uni.showToast({
								title: '信息请求错误',
								duration: 2000
							});		
						}
							
				   }
				});
			},
			getcomment(){
				var _self = this;
				uni.request({
				   url: "http://47.102.212.4:8092/sight/getComment", //请求接口
				   data:{
					   sightId: this.sightid,
				   },
				   method: 'GET',
				success: (res) => {//请求成功后返回
						if (res.statusCode === 200){
							console.log(res.data)
						}else{
							uni.showToast({
								title: '信息请求错误',
								duration: 2000
							});		
						}
							
				   }
				});
			}

		},
	}
</script>

<style>
	page {
		background-color: #f5f5f5;
	}
</style>
<style scoped lang='scss'>
	.app {
		padding-bottom: constant(safe-area-inset-bottom);
		padding-bottom: env(safe-area-inset-bottom);

		&:after {
			content: '';
			display: block;
			width: 100%;
			height: 100rpx;
		}
	}

	/* 标题简介 */
	.sight_name {
		font-size: 44rpx;
		margin-left: 30rpx;
		margin-bottom: 20rpx;
		font-weight: 700px;
	}
	.info_container{
		width: 100%;
		display: flex;
		margin-bottom: 40rpx;
	}
	.scores{
		background-color: #0276f1;
		color: #FFFFFF;
		border-top-left-radius:25rpx;
		border-bottom-left-radius:25rpx;
		margin-left: 40rpx;
		font-size: 28rpx;
		width: 90rpx;
		text-align: center;
	}
	.visitnum{
		background-color: #e5f1fe;
		color: #6faadb;
		border-top-right-radius:25rpx;
		border-bottom-right-radius:25rpx;
		font-size: 28rpx;
		width: 160rpx;
		text-align: center;
	}
	.level{
		background-color: #fdf2e4;
		color: #ecb47b;
		border-radius: 25rpx;
		font-size: 28rpx;
		width: 140rpx;
		text-align: center;
		margin-left: 30rpx;
	}
	.introduction_container{
		width: 100%;
		display: flex;
	}
	.introduction_logo{
		width: 40rpx;
		height: 40rpx;
		margin-left: 20rpx;
	}
	.sight_introduction {
		font-size: 30rpx;
		margin-left: 20rpx;
		margin-bottom: 40rpx;
		color: #8f8f94;
	}
	.address_container{
		width: 100%;
		display: flex;
	}
	.address_logo{
		width: 40rpx;
		height: 40rpx;
		margin-left: 20rpx;
	}
	.sight_address{
		font-size: 30rpx;
		margin-left: 20rpx;
		margin-bottom: 40rpx;
		color: #8f8f94;
		width: 600rpx;
	}
	

/* 	.sight_info {
		height: 200rpx;
		font-size: 32rpx;
		margin-left: 30rpx;
	} */

/* 	.tocomment {
		margin-left: 450rpx;
		top: -375rpx;
		font-size: 32rpx;
		color: #8f8f94;
	} */

	.introduce {
		background: #fff;
		padding: 20rpx 30rpx;

		.title {
			min-height: 44rpx;
			font-size: 32rpx;
			color: #333;
			line-height: 44rpx;
			font-weight: 700;
		}

		.price-wrap {
			min-height: 40rpx;
			margin-top: 28rpx;
			font-size: 26rpx;
		}

		.m-price {
			margin-left: 10rpx;
			margin-right: 16rpx;
			color: #999;
			text-decoration: line-through;
		}

		.tag {
			transform: translateY(4rpx);
			padding: 0 10rpx;
			margin-left: 8rpx;
			background: $base-color;
			font-size: 20rpx;
			color: #fff;
			line-height: 32rpx;
			border-radius: 4rpx;
			position: relative;
			bottom: 8rpx;
		}

		.bot {
			padding: 28rpx 0 10rpx 4rpx;
			font-size: 24rpx;
			color: #999;
		}
	}

	/* ---------------打分-----------	 */
	@font-face {
		font-family: 'iconfont';
		src: url('https://at.alicdn.com/t/font_1254062_cqoky071dbi.eot');
		src: url('https://at.alicdn.com/t/font_1254062_cqoky071dbi.eot?#iefix') format('embedded-opentype'),
			url('https://at.alicdn.com/t/font_1254062_cqoky071dbi.woff2') format('woff2'),
			url('https://at.alicdn.com/t/font_1254062_cqoky071dbi.woff') format('woff'),
			url('https://at.alicdn.com/t/font_1254062_cqoky071dbi.ttf') format('truetype'),
			url('https://at.alicdn.com/t/font_1254062_cqoky071dbi.svg#font_family') format('svg');
	}

	.iconfont {
		//与上方font-family 保持一致
		font-family: "iconfont" !important;
		font-size: 16px;
		font-style: normal;
		-webkit-font-smoothing: antialiased;
		-webkit-text-stroke-width: 0.2px;
		-moz-osx-font-smoothing: grayscale;
	}

	.scoreBox {
		display: flex;
		height: 50rpx;
		background-color: #ffffff;
	}

	.scoreTitle {
		font-size: 30rpx;
		width: 250rpx;
		margin-left: 5%;
	}

	.star.uni-common-mt {
		box-sizing: border-box;
		width: 300rpx;
		overflow: hidden;
		padding: 0.85upx;
		margin-left: -7%;
		margin-top: -1.5%;
	}

	.flex-item {
		display: inline-block;
		width: 20%;
		/* color: #999; */
	}

	.starIcon {
		font-size: 40upx;
	}

	.starActive {
		color: #ff8000;
	}

	/* ----------------------打分	----------------------- */

	.commentbox {
		width: 100%;
		height: auto;
		background-color: #FFFFFF;
	}

	.mycommentinput {
		background-color: #ffffff;
		height: 200rpx;
		width: 90%;
		word-break: break-all;
		margin-left: 5%;
		border-radius: 5px;
		border: solid 1px #ccc;
	}

	.commitCommentButton {
		width: 60rpx;
		height: 60rpx;
		margin-left: 645rpx;
		margin-top: -55rpx;
	}


	/* 分享 */
	.share-wrap {
		background: linear-gradient(to right, #fdf5f6, #fbebf6);
		height: 76rpx;
		padding: 0 30rpx;
		color: $base-color;

		.icon {
			width: 90rpx;
			height: 30rpx;
			border: 1px solid $base-color;
			border-radius: 4rpx;
			position: relative;
			overflow: hidden;
			font-size: 22rpx;

			&:after {
				position: absolute;
				left: -20rpx;
				top: -12rpx;
				content: '';
				width: 50rpx;
				height: 50rpx;
				border-radius: 50%;
				background-color: $base-color;
			}
		}

		.icon-iconfontxingxing {
			position: relative;
			z-index: 1;
			font-size: 24rpx;
			margin-left: 2rpx;
			margin-right: 10rpx;
			color: #fff;
		}

		.tit {
			flex: 1;
			font-size: 26rpx;
			color: #666;
			margin-left: 14rpx;
		}

		.icon-bangzhu1 {
			padding: 10rpx;
			font-size: 30rpx;
			display: none;
		}

		.btn {
			padding-left: 20rpx;
			font-size: 24rpx;
			color: $base-color;
		}

		.icon-you {
			font-size: 22rpx;
			margin-left: 4rpx;
			color: $base-color;
		}
	}

	.c-list {
		font-size: 26rpx;
		color: #888;
		background: #fff;

		.row {
			min-height: 82rpx;
			padding: 16rpx 30rpx;
			position: relative;

			&:after {
				border-color: #eaeaea;
			}

			&:last-child:after {
				border: 0;
			}
		}

		.tit {
			width: 140rpx;
		}

		.con {
			flex: 1;
			color: #333;

			.attr {
				margin-right: 10rpx;
			}

			.service {
				margin-right: 30rpx;

				&:last-child {
					margin: 0;
				}
			}
		}

		.con-list {
			color: #333;

			text {
				line-height: 40rpx;
			}
		}

		.red {
			color: $base-color;
		}

		.icon-you {
			font-size: 24rpx;
			color: #999;
		}
	}

	/* 评价 */
	.rating-wrap {
		padding: 20rpx 30rpx 0rpx;
		background: #fff;
		margin-top: 12rpx;

		&.no-data {
			padding: 10rpx 30rpx 10rpx;
		}

		.e-header {
			display: flex;
			align-items: center;
			height: 70rpx;
			font-size: 28rpx;
			color: #333;
		}

		.tit {
			font-size: 32rpx;
			color: #333;
			font-weight: 700;
			margin-right: 4rpx;
		}

		.tip {
			flex: 1;
			font-size: 26rpx;
			color: #999;
			text-align: right;
		}

		.icon-you {
			margin-left: 8rpx;
			font-size: 24rpx;
			color: #999;
		}

		.mix-rating-item::after {
			border: 0;
		}
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
</style>
