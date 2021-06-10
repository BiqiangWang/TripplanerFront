<template>
	<view class="app">
		<view class="nav-wrap">
			<mix-nav-bar :navs="navs" :counts="navCounts" :current="navCurrent" @onChange="onNavBarChange"></mix-nav-bar>
		</view>
		
<!-- 		<mescroll-body
			ref="mescrollRef" 
			@init="mescrollInit" 
			:top="84" 
			@down="downCallback" 
			:up="upOption" 
			@up="loadList" 
		>
			<rating-item 
				v-for = "(item, index) in list" 
				:key = "index"
				:items = "item"
			></rating-item>
			
		</mescroll-body> -->
		<view class="allcomment">
			<rating-item
				v-if="navReset"
				v-for = "(item, index) in list" 
				:key = "index"
				:items = "item.Comment"
			></rating-item>
		</view>
		
		<!-- <mix-loading v-if="isLoading" :mask="true"></mix-loading> -->
	</view>
</template>

<script>
	import ratingItem from './components/rating-item'
	import MescrollMixin from "@/components/mescroll-uni/mescroll-mixins.js";
	export default {
		components: {
			ratingItem
		},
		mixins: [MescrollMixin], 
		data() {
			return {
				navReset:true,
				navs: [{name: '最新'}, {name: '好评'}, {name: '中评'}, {name: '差评'}],
				navCurrent: 0, //当前tab
				upOption:{
					auto: false, // 是否自动加载
					page: {
					 	num: 0, // 当前页码,默认0,回调之前会加1,即callback(page)会从1开始
					 	size: 15 // 每页数据的数量
					},
					noMoreSize: 5,
				},
				navCounts: [], //数量
				list: [
					// {
					// 	user: {
					// 		anonymous: false,
					// 		lv: 300,
					// 		avatar: '/static/icon/default-avatar.png',
					// 		gender: 1,
					// 		nickname: '飞飞',
					// 		username: '飞飞',
					// 	},
					// 	rating: 5,
					// 	content: 'good',
					// 	date: "2021-5-1",
					// },
					// {
					// 	user: {
					// 		anonymous: true,
					// 		lv: 0,
					// 		avatar: '/static/icon/default-avatar.png',
					// 		gender: 2,
					// 		nickname: '冲冲',
					// 		username: '冲冲',
					// 	},
					// 	rating: 5,
					// 	content: '这也太好玩了吧',
					// 	date: "2021-5-1",
					// },
					// {
					// 	user: {
					// 		anonymous: false,
					// 		lv: 3,
					// 		avatar: '/static/icon/default-avatar.png',
					// 		gender: 2,
					// 		nickname: '瑶瑶公主',
					// 		username: '瑶瑶公主',
					// 	},
					// 	rating: 2,
					// 	content: '就那样吧',
					// 	date: "2021-5-1",
					// },
				],
			}
		},
		onLoad(options) {
			this.id = options.id;
			this.loadCount();
			this.getallcomment();
		},
		methods: {
			async loadList(e){
				this.mescroll && this.mescroll.removeEmpty();
				const res = await this.$request('rating', 'get', {
					product_id: this.id,
					offset: (e.num - 1) * e.size,
					limit: e.size,
					type: this.navCurrent
				});
				const curList = res.data;
				if(e.num === 1){
					//第一页清空数据重载
					this.list = [];
				} 
				this.list = this.list.concat(curList); //追加新数据
				this.mescroll.endSuccess(curList.length); //结束加载状态
			},
			async loadCount(){
				const res = await this.$request('rating', 'count', {
					product_id: this.id
				})
				this.navCounts = res;
				console.log(res);
			},
			mescrollInit(mescroll){
				this.isLoading = true;
				this.mescroll = mescroll;
				setTimeout(()=>{
					this.mescroll.resetUpScroll(false)
				}, 10)
			},
			//tab切换
			onNavBarChange(current){
				if(this.navCurrent === current){
					return;
				}
				this.navCurrent = current;
				// this.isLoading = true;
				// this.mescroll && this.mescroll.resetUpScroll(false)
				console.log(this.navCurrent)
				if(this.navCurrent==1){
					this.getgoodcomment()
				}else if(this.navCurrent==2){
					this.getmidcomment()
				}else if(this.navCurrent==3){
					this.getbadcomment()
				}else{
					this.getallcomment()
				}
			},
			getallcomment(){
				uni.request({
				   url: "http://47.102.212.4:8092/sight/getComment", //请求接口
				   data:{
					   sightId: this.id,
				   },
				   method: 'GET',
				success: (res) => {//请求成功后返回
						if (res.statusCode === 200){
							console.log(res.data.data.commentlist.commentlist)
							this.list.length=0;
							this.list = res.data.data.commentlist.commentlist;
							this.navReset = false;
							  this.$nextTick(() => {
							       this.navReset = true;
							  })
						}else{
							uni.showToast({
								title: '评论请求错误',
								duration: 2000
							});		
						}
							
				   }
				});
			},
			getgoodcomment(){
				uni.request({
				   url: "http://47.102.212.4:8092/sight/getgoodcomment?", //请求接口
				   data:{
					   sightId: this.id,
				   },
				   method: 'GET',
				success: (res) => {//请求成功后返回
						if (res.statusCode === 200){
							console.log(res.data.data.commentlist.goodcommentlist);
							this.list.length=0;
							this.list = res.data.data.commentlist.goodcommentlist;
							this.navReset = false;
							  this.$nextTick(() => {
							       this.navReset = true;
							  })
						}else{
							uni.showToast({
								title: '评论请求错误',
								duration: 2000
							});		
						}
							
				   }
				});
			},
			getmidcomment(){
				uni.request({
				   url: "http://47.102.212.4:8092/sight/getmidcomment?", //请求接口
				   data:{
					   sightId: this.id,
				   },
				   method: 'GET',
				success: (res) => {//请求成功后返回
						if (res.statusCode === 200){
							console.log(res.data.data.commentlist.midcommentlist) 
							this.list.length=0;
							this.list = res.data.data.commentlist.midcommentlist;
							this.navReset = false;
							  this.$nextTick(() => {
							       this.navReset = true;
							  })
						}else{
							uni.showToast({
								title: '评论请求错误',
								duration: 2000
							});		
						}
							
				   }
				});
			},
			getbadcomment(){
				uni.request({
				   url: "http://47.102.212.4:8092/sight/getbadcomment?", //请求接口
				   data:{
					   sightId: this.id,
				   },
				   method: 'GET',
				success: (res) => {//请求成功后返回
						if (res.statusCode === 200){
							console.log(res.data.data.commentlist.badcommentlist) 
							this.list.length=0;
							this.list = res.data.data.commentlist.badcommentlist;
							this.navReset = false;
							  this.$nextTick(() => {
							       this.navReset = true;
							  })
						}else{
							uni.showToast({
								title: '评论请求错误',
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
		padding: 0 30rpx;
		
		/deep/ {
			.mescroll-body-content{
				padding-top: 10rpx;
			}
		}
	}
	.allcomment{
		margin-top: 100rpx;
	}
</style>
