<template>
	<view class="app">
		<view class="cell row b-b">
			<text class="tit">路线名称</text>
			<input class="input" type="text" maxlength="10" v-model="data.name" placeholder="请输入路线名称" placeholder-class="placeholder" />
		</view>
		<view class="cell row b-b">
			<text class="tit">路线描述</text>
			<input class="input" type="text" maxlength="11" v-model="data.mobile" placeholder="请输入描述(选填)" placeholder-class="placeholder" />
		</view>
		<view class="cell row b-b">
			<text class="tit">景点数量</text>
			<!-- <input class="input" type="number" maxlength="11" v-model="data.mobile" placeholder="请输入描述(选填)" placeholder-class="placeholder" /> -->
			<view class="numchoose">
				<image class="subtract" src="../../static/tplaner/minus.png" @click="sub"></image>
				<view class="sightnum"> {{ sightnum }} </view>
				<image class="add" src="../../static/tplaner/add.png" @click="add"></image>
			</view>
		</view>
		<view class="cell row b-b" @click="chooseAddress">
			<text class="tit">地址</text>
			<text class="input clamp" :class="{placeholder: !data.address.address}">
				{{ data.address.address ? 
					data.address.address + ' ' + data.address.room :
					'请在地图中选择出发地点' 
				}}
			</text>
			<text class="mix-icon icon-you"></text>
		</view>
		<view class="cell row">
			<text class="tit">预算金额：(元)</text>
		</view>
		<slider value="300" @change="sliderChange" min="0" max="2000" show-value step="100" />
		
		<view class="cell row">
			<text class="tit">交通方式</text>
		</view>
		<view>
			<checkbox-group @change="checkboxChange">
				<label class="checkbox" v-for="item in checkboxItems" :key="item.name">
					<view class="box-container">
						<checkbox :value="item.name" :checked="item.checked"></checkbox>
					</view>
					<view class="row">
						<view class="tit">{{item.value}}</view>
					</view>
				</label>
			</checkbox-group>
		</view>
<!-- 		<view class="cell row b-b">
			<text class="tit fill">设为默认地址</text>
			<switch :checked="data.is_default" color="#FF536F" @change="onSwitchChange" />
		</view> -->
		<mix-button ref="confirmBtn" text="开始规划" marginTop="60rpx" @onConfirm="submit"></mix-button>
	</view>
</template>

<script>
	import {mapState, mapGetters} from 'vuex'
	import {checkStr} from '@/common/js/util'
	export default {
		computed: {
			...mapState(['userInfo', 'orderCount', 'couponCount']),
			...mapGetters(['hasLogin']),
		},
		onShow() {
			this.$store.dispatch('getUserInfo');
			this.userid = this.userInfo.id;
			console.log(this.userid)
			this.getUserInfo();
		},
		data() {
			return {
				// num:1,
				userid:"",
				budget:300,
				sightnum:3,
				city:'',
				is_default: true,
				data: {
					address: {}
				},
				checkboxItems: [
					{
						name: 'Taxi',
						value: '打车（推荐使用滴滴打车哦）',
						checked: false,
					},
					{
						name: 'Self-driving',
						value: '自驾（更自由）',
						checked: true,
					},
					{
						name: 'public-transit',
						value: '公共交通(包括公交，地铁等)',
						checked: false,
					}
				],
			}
		},
		onLoad(options) {
			if(options.data){
				this.data = JSON.parse(options.data)
			}
		},
		methods: {
			async submit(){
				const data = this.data;
				if(!data.name){
					this.$util.msg('请输入路线名称');
					this.$refs.confirmBtn.stop();
					return;
				}
				// if(!checkStr(data.mobile, 'mobile')){
				// 	this.$util.msg('请输入路线描述');
				// 	this.$refs.confirmBtn.stop();
				// 	return;
				// }
				if(!data.address.address){
					this.$util.msg('请选择出发地点');
					this.$refs.confirmBtn.stop();
					return;
				}
				this.routePlan()
			},
			//选择地址
			chooseAddress(){
				let url = '/pages/chooseAddress/index';
				if(this.data.address.title){
					url += `?data=${JSON.stringify(this.data.address)}`; 
				}
				this.navTo(url);
			},
			//选择地址回调
			setAddress(e){
				console.log(JSON.stringify(e));
				this.city = e.ad_info.city.substr(0,2);
				this.data.address = e;
			},
			onSwitchChange(e){
				this.data.is_default = e.detail.value;
			},
			sliderChange(e) {
				console.log('value 发生变化：' + e.detail.value)
				this.budget=e.detail.value;
			},
			checkboxChange: function(e) {
				console.log(e)
				var checked = e.target.value
				var changed = {}
				for (var i = 0; i < this.checkboxItems.length; i++) {
					if (checked.indexOf(this.checkboxItems[i].name) !== -1) {
						changed['checkboxItems[' + i + '].checked'] = true
					} else {
						changed['checkboxItems[' + i + '].checked'] = false
					}
				}
			},
			routePlan(){
				uni.request({
				   url: "http://47.102.212.4:8082/search/route", //请求接口
				   data:{
					   name: this.data.name,
					   desc: this.mobile,
					   city: this.city,
					   cost: this.budget,
					   snum: this.sightnum,
					   trans: 1,
					   user: this.userid,
					
				   },
				   method: 'POST',
				   header:{'content-type':'application/x-www-form-urlencoded'},
				success: (res) => {//请求成功后返回
						if (res.data.code === 20000){
							console.log(res.data.data);
							uni.showToast({
								title: '路线规划成功',
								duration: 2000
							});
							this.navTo('/pages/route/routeInfo?')
						}else{
							uni.showToast({
								title: '路线规划请求错误',
								duration: 2000
							});		
						}
				   }
				});
			},
			sub(){
				if(this.sightnum>0){
					this.sightnum--;
				}
			},
			add(){
				if(this.sightnum<6)
					this.sightnum++;
			},
		}
	}
</script>

<style scoped lang="scss">
	.app{
		padding: 10rpx 44rpx 0;
	}
	.cell{
		height: 106rpx;
		
		.tit{
			min-width: 130rpx;
			font-size: 30rpx;
			color: #333;
		}
		.input{
			flex: 1;
			font-size: 30rpx;
			color: #333;
		}
		.icon-you{
			flex-shrink: 0;
			margin-right: 8rpx;
			margin-left: 40rpx;
			font-size: 24rpx;
			color: #aaa;
		}
		switch{
			transform: scale(0.8) translateX(10rpx);
			transform-origin: center right;
		}
	}
	
	.checkbox{
		display: flex;
	}
	.box-container{
		width: 100rpx;
		margin-right: 0;
		margin-left: 50rpx;
		margin-bottom: 20rpx;
	}
	.text-container{
		width: 200rpx;

	}
	
	.numchoose{
		display: flex;
	}
	.subtract{
		width: 40rpx;
		height: 40rpx;
		margin-left: 20rpx;
		top: 10rpx;
	}
	.sightnum{
		margin-left: 20rpx;
		width: 30rpx;
		font-size: 40rpx;
	}
	.add{
		width: 40rpx;
		height: 40rpx;
		margin-left: 20rpx;
		top: 10rpx;
	}
</style>
