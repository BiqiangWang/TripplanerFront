<template>
	<view class="home-banner">
		<swiper
			class="swiper" 
			circular 
			autoplay 
			interval="5000"
			previous-margin="50rpx"
			next-margin="50rpx"
			@change="onSwiperChange"
		>
			<swiper-item class="item" v-for="(item, index) in list" :key="index" >
				<image 
					class="pic" 
					:class="{current: current === index}"
					:src="item.image" 
					mode="aspectFill"
					@click="navTo(item.link)"
				></image>
			</swiper-item>
		</swiper>
		<view class="dots row center">
			<view class="dot" :class="{current: current === index}" v-for="(item, index) in list" :key="index"></view>
		</view>
	</view>
</template>

<script>
	export default {
		name: 'HomeBanner',
		data() {
			return {
				current: 0,
				list: [
					{image: 'http://img1.qunarzz.com/sight/p0/1603/4e/4e7a704010c95d2490.water.jpg_280x200_6e3222e6.jpg', 
					link: '/pages/product/detail'},
					{image: 'http://img1.qunarzz.com/sight/p0/1512/96/96d55a05f5b087b090.water.jpg_280x200_48bccc2e.jpg', 
					link: '/pages/product/detail'},
					{image: 'http://img1.qunarzz.com/sight/p0/1711/f4/f4c46492e38479ca3.img.jpg_280x200_40984876.jpg', 
					link: '/pages/product/detail'},
				]
			};
		},
		// props: {
		// 	list: {
		// 		type: Array,
		// 		default(){
		// 			return []
		// 		}
		// 	}
		// },
		methods: {
			onSwiperChange(e){
				this.current = e.detail.current;
			}
		}
	}
</script>

<style scoped lang="scss">
	.home-banner{
		position: relative;
		background-color: #fff;
	}
	.swiper{
		width: 750rpx;
		height: 310rpx;
		padding-top: 10rpx;
	}
	.pic{
		display: block;
		width: 100%;
		height: 100%;
		border-radius: 16rpx;
		transform: scale(0.94, 0.88);
		transition: transform .36s;
		
		&.current{
			transform: scale(1);
		}
	}
	.dots{
		position: absolute;
		left: 0;
		bottom: 12rpx;
		width: 100%;
	}
	.dot{
		width: 32rpx;
		height: 8rpx;
		margin: 0 6rpx;
		background-color: #fff;
		border-radius: 10px;
		
		&.current{
			background-color: $base-color;
		}
	}
</style>
