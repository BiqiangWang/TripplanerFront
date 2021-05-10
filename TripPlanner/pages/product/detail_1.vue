<template>
	<view class="content">

		<!-- 头部轮播 -->
		<view class="carousel-section">
			<view class="titleNview-placing"></view>
			<view class="titleNview-background" :style="{backgroundColor:titleNViewBackground}"></view>
			<swiper class="carousel" circular @change="swiperChange">
				<swiper-item v-for="(item, index) in carouselList" :key="index" class="carousel-item" @click="navToDetailPage({title: '轮播广告'})">
					<image :src="item.src" />
				</swiper-item>
			</swiper> 
			<view class="swiper-dots">
				<text class="num">{{swiperCurrent+1}}</text>
				<text class="sign">/</text>
				<text class="num">{{swiperLength}}</text>
			</view>
		</view>
		
		<!-- <mix-swiper :list="data.images"></mix-swiper> -->
		
		<view class="text-area">
			<view class="sight_name">{{sightname}}</view>
			<view class="sight_introduction">{{title}}</view>
			<view class="sight_info">{{info}}</view>
		</view>
		<view class="scoreBox">
			<view class="scoreTitle">为该景点打分：</view>
			<view class="uni-padding-wrap uni-common-mt star">
				<view class="uni-flex uni-row">
					<view :class="{starActive:item}" @click="choise(index)" class="flex-item iconfont" v-for="(item,index) in clicked_list">
						<view v-if="item" class="starIcon">
							&#xe601;
						</view>
						<view v-else class="starIcon">
							&#xe602;
						</view>
					</view>
			
				</view>
			</view>
		</view>
		<textarea class="commentInput" auto-height="true" placeholder="请输入您的评价" v-model="comment"></textarea>
		<image class="commitCommentButton" src="../../static/select.png" @click="commitComment"></image>
		<view class="allComment">
			<view class="singlecomment" v-for="(item,index) in all_comment">
				<view class="avatar"></view>
				<view class="commenttitle">{{item.commenttitle}}</view>
				<view class="commentintro">{{item.commentintro}}</view>
				<view class="commentdate">{{item.date}}</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				titleNViewBackground: '',
				swiperCurrent: 0,
				swiperLength: 0,
				carouselList: [
					{
						src:"../../static/logo.png",
						background:"#ffffff",
					},
					{
						src:"../../static/select.png",
						background:"#ffffff",
					},
					{
						src:"../../static/logo.png",
						background:"#ffffff",
					},
				],
				sightname:'景点名称',
				title: '这里是一段简单的景点介绍',
				info:'景点信息啊',
				clicked_list:[false,false,false,false,false], //对应星星个数
				all_comment:[
					{
						commenttitle:"1",
						commentintro:"简介1",
						date:"2021-4-16",
					},{
						commenttitle:"2",
						commentintro:"简介2",
						date:"2021-4-16",
					},{
						commenttitle:"3333333333333333333333333333333333333",
						commentintro:"简介哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈33",
						date:"2021-4-16",
					},{
						commenttitle:"3333333333333333333333333333333333333",
						commentintro:"简介哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈33",
						date:"2021-4-16",
					},{
						commenttitle:"3333333333333333333333333333333333333",
						commentintro:"简介哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈33",
						date:"2021-4-16",
					},
				],
				comment:'', 
			}
		},
		props:{
			
		},
		onLoad() {
			this.loadData();
		},
		methods: {
			async loadData() {
				this.swiperLength = this.carouselList.length;
			},
			//轮播图切换修改背景色
			swiperChange(e) {
				const index = e.detail.current;
				this.swiperCurrent = index;
				this.titleNViewBackground = this.carouselList[index].background;
			},
			starIcon(item){
				if(item){
					return  '&#xe601;'
				}else{
					return  '&#xe602;'
				}
			},
			//点击选择
			choise(num){
				// num 为点击的星星在数组中的下标
				this.clicked_list=[false,false,false,false,false];
				num=num+1;
				for(let i= 0 ; i < num ; i++){
					this.clicked_list[i]=true;
				}				
				console.log(num);
				
			},
			commitComment(){
				console.log(this.comment);
			}

		}
	}
</script>


<style lang="scss" scoped>
	.content {
		width: 100%;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

// ----------------------轮播图-------------------------------
	.carousel-section {
		position: relative;
		padding-top: 10rpx;
	
		.titleNview-placing {
			height: var(--status-bar-height);
			padding-top: 44rpx;
			box-sizing: content-box;
		}
	
		.titleNview-background {
			position: absolute;
			top: 0;
			left: 0;
			width: 100%;
			height: 426rpx;
			transition: .4s;
		}
	}
	.carousel {
		width: 800rpx;
		height: 350upx;
	
		.carousel-item {
			width: 100%;
			height: 100%;
			overflow: hidden;
		}
	
		image {
			width: 100%;
			height: 100%;
			border-radius: 10upx;
		}
		
	}
	.swiper-dots {
		display: flex;
		position: absolute;
		left: 60upx;
		bottom: 15upx;
		width: 72upx;
		height: 36upx;
		background-image: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAMgAAABkCAYAAADDhn8LAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAyZpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuNi1jMTMyIDc5LjE1OTI4NCwgMjAxNi8wNC8xOS0xMzoxMzo0MCAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wTU09Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9tbS8iIHhtbG5zOnN0UmVmPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvc1R5cGUvUmVzb3VyY2VSZWYjIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtcE1NOkRvY3VtZW50SUQ9InhtcC5kaWQ6OTk4MzlBNjE0NjU1MTFFOUExNjRFQ0I3RTQ0NEExQjMiIHhtcE1NOkluc3RhbmNlSUQ9InhtcC5paWQ6OTk4MzlBNjA0NjU1MTFFOUExNjRFQ0I3RTQ0NEExQjMiIHhtcDpDcmVhdG9yVG9vbD0iQWRvYmUgUGhvdG9zaG9wIENDIDIwMTcgKFdpbmRvd3MpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6Q0E3RUNERkE0NjExMTFFOTg5NzI4MTM2Rjg0OUQwOEUiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6Q0E3RUNERkI0NjExMTFFOTg5NzI4MTM2Rjg0OUQwOEUiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz4Gh5BPAAACTUlEQVR42uzcQW7jQAwFUdN306l1uWwNww5kqdsmm6/2MwtVCp8CosQtP9vg/2+/gY+DRAMBgqnjIp2PaCxCLLldpPARRIiFj1yBbMV+cHZh9PURRLQNhY8kgWyL/WDtwujjI8hoE8rKLqb5CDJaRMJHokC6yKgSCR9JAukmokIknCQJpLOIrJFwMsBJELFcKHwM9BFkLBMKFxNcBCHlQ+FhoocgpVwwnv0Xn30QBJGMC0QcaBVJiAMiec/dcwKuL4j1QMsVCXFAJE4s4NQA3K/8Y6DzO4g40P7UcmIBJxbEesCKWBDg8wWxHrAiFgT4fEGsB/CwIhYE+AeBAAdPLOcV8HRmWRDAiQVcO7GcV8CLM8uCAE4sQCDAlHcQ7x+ABQEEAggEEAggEEAggEAAgQACASAQQCCAQACBAAIBBAIIBBAIIBBAIABe4e9iAe/xd7EAJxYgEGDeO4j3EODp/cOCAE4sYMyJ5cwCHs4rCwI4sYBxJ5YzC84rCwKcXxArAuthQYDzC2JF0H49LAhwYUGsCFqvx5EF2T07dMaJBetx4cRyaqFtHJ8EIhK0i8OJBQxcECuCVutxJhCRoE0cZwMRyRcFefa/ffZBVPogePihhyCnbBhcfMFFEFM+DD4m+ghSlgmDkwlOgpAl4+BkkJMgZdk4+EgaSCcpVX7bmY9kgXQQU+1TgE0c+QJZUUz1b2T4SBbIKmJW+3iMj2SBVBWz+leVfCQLpIqYbp8b85EskIxyfIOfK5Sf+wiCRJEsllQ+oqEkQfBxmD8BBgA5hVjXyrBNUQAAAABJRU5ErkJggg==);
		background-size: 100% 100%;
	
		.num {
			width: 36upx;
			height: 36upx;
			border-radius: 50px;
			font-size: 24upx;
			color: #fff;
			text-align: center;
			line-height: 36upx;
		}
	
		.sign {
			position: absolute;
			top: 0;
			left: 50%;
			line-height: 36upx;
			font-size: 12upx;
			color: #fff;
			transform: translateX(-50%);
		}
	}
//--------------------------------------------------------------------

	.text-area {
		display: flex;
/* 		justify-content: center;
		flex: 1; */
		flex-direction: column;
		margin-left: -40%;
	}
	
	.sight_name{
		font-size: 36rpx;
		margin-bottom: 50rpx;
	}

	.sight_introduction {
		font-size: 30rpx;
		color: #8f8f94;
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
	
	.iconfont{ //与上方font-family 保持一致
	    font-family:"iconfont" !important;
	    font-size:16px;font-style:normal;
	    -webkit-font-smoothing: antialiased;
	    -webkit-text-stroke-width: 0.2px;
	    -moz-osx-font-smoothing: grayscale;
	}
	
	.scoreBox{
		display: flex;
	}
	
	.scoreTitle{
		width: 100%;
		margin-left: -10%;
	}
	
	.star.uni-common-mt{
		box-sizing: border-box;
		width: 100%;
		overflow: hidden;
		padding: 0.85upx;
		margin-left: -20%;
	}
	.flex-item{
		display: inline-block;
		width: 20%;
		color: #999;
	}
	.starIcon{
		font-size: 52upx;
	}
	.starActive{
		color: #ff8000;
	}
/* ----------------------打分	----------------------- */
	

	.commentInput{
		width: 80%;
		height: auto;
		// word-break: break-all;
		margin-top: 24upx;
		padding: 24upx 24upx;
		background-color: #F5F5F5;
		border-radius: 15px;			

	}
	.commitCommentButton{
		width: 60rpx;
		height: 60rpx;
		margin-left: 500rpx;
		margin-top: -75rpx;
	}
	
	.allComment{
		width: 100%;
		margin-top: 30rpx;
	}
	.singlecomment{
		display: flex;
		width: 90%;
		margin-left: 5%;
		margin-top: 18rpx;
		background-color: #F5F5F5;
		border-radius: 15px;
	}
	.avatar{
		width: 140rpx;
		height: 105rpx;
		background-color: #55557f;
		border-radius: 15px;
	}
	.commenttitle{
		width: 250rpx;
		margin-left: 30rpx;
		font-family: "微软雅黑";
		font-size: 40rpx;
		overflow: hidden;
		white-space: nowrap; //文本强制不换行；
		text-overflow: ellipsis;  /* 超出部分省略号 */

	}
	.commentintro{
		width: 170px;
		margin-top:60rpx;
		margin-left: -260rpx;
		color: #8f8f94;
		font-size: 35rpx;
		overflow: hidden;
		white-space: nowrap; //文本强制不换行；
		text-overflow: ellipsis;  /* 超出部分省略号 */
	}
	.commentdate{
		margin-top: 60rpx;
		font-size: 30rpx;
		color: #8f8f94;
	}
</style>
