<template>
	<view class="app column">
		<!-- 顶部栏 -->
		<page-header></page-header>

		<view class="content row">
			<scroll-view class="left-side" scroll-y>
				<view class="item center" :class="{active: item._id === current._id}" v-for="item in list"
					:key="item._id" @click="changeCate(item)">
					<text>{{ item.name }}</text>
				</view>
			</scroll-view>
			<scroll-view class="right-side" scroll-y>
				<!-- <image class="cate-image" :src="current.image" mode="aspectFill"></image>
				<view class="wrap">
					<view class="item column" v-for="item in current.child" :key="item._id" @click="navToList(item)">
						<image class="icon" :src="item.image" mode="aspectFill"></image>
						<text class="tit">{{ item.name }}</text>
					</view>
				</view> -->
				<product-list ref="productList" :list="hotList" listType='row'></product-list>
			</scroll-view>
			
		</view>
	</view>
</template>

<script>
	import tabbarMixin from './mixin/tabbar'
	import pageHeader from './components/page-header.vue'
	import productList from '@/pages/product/components/product-list.vue'
	export default {
		components: {
			pageHeader,
			productList
		},
		mixins: [tabbarMixin],
		data() {
			return {
				list: [{
					_id: 1,
					name: '文化古迹'
				}, {
					_id: 2,
					name: '城市观光'
				}, {
					_id: 3,
					name: '演出'
				}, {
					_id: 4,
					name: '农家度假'
				}, {
					_id: 5,
					name: '游乐场'
				}, {
					_id: 6,
					name: '自然风光'
				}, {
					_id: 7,
					name: '餐饮'
				}],
				current: {},
				sightList: [],
				hotList: [{
							thumb: 'http://img1.qunarzz.com/sight/p0/2005/39/3979f1867defec4ea3.water.jpg_280x200_e1b47993.jpg',
							title: '八达岭长城',
							sales: '2307',
							price: '40'
						},
						{
							thumb: 'http://img1.qunarzz.com/tuan/team2/1507/2c/83e0e0e7ae082a.jpg_280x200_8c8e548a.jpg',
							title: '东方明珠',
							sales: '2307',
							price: '69.9'
						},
						{
							thumb: 'http://img1.qunarzz.com/sight/p64/201211/04/9173ff9f33e97f3193835fbb.jpg_280x200_9537b05a.jpg',
							title: '杜甫草堂',
							sales: '2307',
							price: '47'
						}
					],//热门推荐
			}
		},
		onLoad() {
			this.loadData();
		},
		methods: {
			async loadData() {
				//获取分类 缓存1小时
				const res = await this.$request('product', 'getCategory', {}, {
					cache: 60 * 60 * 0,
				});
				this.current = res[0];
				this.list = res;
			},
			getTagSight() {
				uni.request({
					url: "http://47.102.212.4:8080/search/tag", //请求接口
					data: {
						tag: this.item.name,
					},
					header:{'content-type':'application/x-www-form-urlencoded'},
					method: 'POST',
					success: (res) => { //请求成功后返回
						console.log(res.data)
						if (res.statusCode === 200) {
							
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
			},
			changeCate(item) {
				this.current = item;
				
			},
			navToList(item) {
				const arr = [];
				this.current.child.forEach(cate => {
					arr.push({
						_id: cate._id,
						name: cate.name
					})
				})
				this.navTo(`/pages/product/list?cateId=${item._id}&firstCateId=${item.parent_id}`)
			}
		}
	}
</script>

<style>
	page {
		height: 100%;
	}
</style>
<style scoped lang="scss">
	.app {
		height: 100%;
	}

	.content {
		flex: 1;
		padding-top: 12rpx;
		overflow: hidden;
	}

	.left-side {
		flex-shrink: 0;
		width: 180rpx;
		height: 100%;
		overflow-y: hidden;
		background-color: #f2f2f2;

		.item {
			height: 90rpx;
			font-size: 26rpx;
			color: #555;
		}

		.active {
			font-size: 28rpx;
			color: $base-color;
			font-weight: 700;
			background-color: #fff;
			position: relative;

			&::before {
				content: '';
				position: absolute;
				left: 0;
				top: 30rpx;
				width: 6rpx;
				height: 30rpx;
				background-color: $base-color;
				border-radius: 0 4rpx 4rpx 0;
			}
		}
	}

	.right-side {
		flex: 1;
		height: 100%;

		.cate-image {
			width: calc(100% - 40rpx);
			height: 200rpx;
			margin: 0 20rpx;
			border-radius: 8rpx;
		}

		.wrap {
			display: flex;
			flex-wrap: wrap;
			padding: 0 20rpx 20rpx;
		}

		.item {
			flex-shrink: 0;
			justify-content: flex-start;
			align-items: center;
			width: 30%;
			padding: 30rpx 0 0;

			&:nth-child(3n-1) {
				width: 40%;
			}
		}

		.icon {
			width: 108rpx;
			height: 108rpx;
			margin-bottom: 16rpx;
		}

		.tit {
			width: 140rpx;
			font-size: 24rpx;
			color: #333;
			text-align: center;
			line-height: 1.4;
		}
	}
</style>
