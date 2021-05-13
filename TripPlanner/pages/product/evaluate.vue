<template>
	<view class="app">
		<view class="zhanwei"></view>
		<view class="btn center" @click="navTo('/pages/product/detail')">
			<text class="mix-icon icon-xiangzuo"></text>
		</view>
		<view class="headernote">- 写下真实体验，帮助万千用户选择 -</view>
		<view class="sight_name">{{sight_name}}</view>
		<!-- 打分 -->
		<view class="scoreBox">
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
		</view>
		<view class="commentbox">
			<!-- <textarea class="mycommentinput" auto-height="true" placeholder="请输入您的评价" v-model="comment"></textarea> -->
			<textarea class="mycommentinput" placeholder="你可以从交通,游玩路线,推荐活动等方面进行评价,分享你的体验 ~ ~" maxlength="100" v-model="comment"></textarea>
			<!-- <image class="commitCommentButton" src="../../static/select.png" @click="commitComment"></image> -->
			<button class="commitCommentButton2" @click="commitComment()">发布</button>
		</view>
	</view>
</template>

<script>
	export default {
		data(){
			return{
				sight_name:'八达岭长城',
				clicked_list: [false, false, false, false, false], //对应星星个数
				comment:'', 
			}
		},
		methods:{
			navBack(){
				uni.navigateBack();
			},
			choise(num) {
				// num 为点击的星星在数组中的下标
				this.clicked_list = [false, false, false, false, false];
				num = num + 1;
				for (let i = 0; i < num; i++) {
					this.clicked_list[i] = true;
				}
				console.log(num);
	
			},
			commitComment() {
				console.log(this.comment);
			},
		}
	}
</script>

<style scoped>
	.app{
		padding-bottom: constant(safe-area-inset-bottom);
		padding-bottom: env(safe-area-inset-bottom);
		
		&:after {
			content: '';
			display: block;
			width: 100%;
			height: 100rpx;
		}
	}
	.btn{
		width: 80rpx;
	}
	.icon-xiangzuo{
		width: 52rpx;
		height: 52rpx;
		border-radius: 100rpx;
		font-size: 36rpx;
		color: #333;
		text-align: center;
		line-height: 52rpx;
		
	}
	
	.zhanwei{
		height:150rpx;
		width: 100%;
	}
	.headernote{
		width: 100%;
		text-align: center;
		color: #ff536f;
		margin-top: 20rpx;
		font-size: 32rpx;
	}
	.sight_name{
		font-size: 50rpx;
		margin-top: 30rpx;
		margin-bottom: 30rpx;
		margin-left: 5%;
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
		margin-bottom: 20rpx;
	}
	
	.scoreTitle {
		font-size: 36rpx;
		width: 250rpx;
		margin-left: 5%;
	}
	
	.star.uni-common-mt {
		box-sizing: border-box;
		width: 300rpx;
		overflow: hidden;
		padding: 0.85upx;
		margin-left: -2%;
		margin-top: -1.5%;
	}
	
	.flex-item {
		display: inline-block;
		width: 20%;
		/* color: #999; */
	}
	
	.starIcon {
		font-size: 48upx;
	}
	
	.starActive {
		color: #ff536f;
	}
	
	/* ----------------------打分	----------------------- */
	
	.commentbox{
		width: 100%;
		height: auto;
		background-color: #FFFFFF;
	}
	.mycommentinput{
		background-color: #ffffff;
		height: 400rpx;
		width: 90%;
		word-break: break-all;
		margin-left: 5%;
		border-radius: 5px;
		border:solid 1px #ccc;
	}
	.commitCommentButton{
		width: 60rpx;
		height: 60rpx;
		margin-left: 645rpx;
		margin-top: -55rpx;
	}
	.commitCommentButton2{
		margin-left: 550rpx;
		margin-top: -100rpx;
		color: #ff536f;
	}
	
</style>
