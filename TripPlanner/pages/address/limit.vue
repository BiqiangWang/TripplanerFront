<template>
	<view class="app">
<!-- 		<view class="update-count">
			<view class="reduce" @tap="reduce(num)">
			   <text class="add-reduce">-</text>
			</view>
			<view class="cart-count">
				{{num}}
			</view>
			<view class="add" @tap="add(num)">
				<text class="add-reduce">+</text>
			</view>
		</view> -->


		<view class="cell row b-b">
			<text class="tit">联系人</text>
			<input class="input" type="text" maxlength="10" v-model="data.name" placeholder="请输入联系人姓名" placeholder-class="placeholder" />
		</view>
		<view class="cell row b-b">
			<text class="tit">手机号</text>
			<input class="input" type="number" maxlength="11" v-model="data.mobile" placeholder="请输入联系人手机号码" placeholder-class="placeholder" />
		</view>
<!-- 		<view class="cell row b-b">
			<text class="tit">预算金额</text>
			<input class="input" type="number" maxlength="11" v-model="data.mobile" placeholder="请输入旅行预算金额" placeholder-class="placeholder" />
		</view> -->
		<view class="cell row b-b" @click="chooseAddress">
			<text class="tit">地址</text>
			<text class="input clamp" :class="{placeholder: !data.address.address}">
				{{ data.address.address ? 
					data.address.address + ' ' + data.address.room :
					'请在选择出发地点' 
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
	import {checkStr} from '@/common/js/util'
	export default {
		data() {
			return {
				num:1,
				budget:300,
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
					this.$util.msg('请输入联系人姓名');
					this.$refs.confirmBtn.stop();
					return;
				}
				if(!checkStr(data.mobile, 'mobile')){
					this.$util.msg('请输入正确的手机号码');
					this.$refs.confirmBtn.stop();
					return;
				}
				if(!data.address.address){
					this.$util.msg('请选择出发地点');
					this.$refs.confirmBtn.stop();
					return;
				}
				const operation = data._id ? 'update' : 'add';
				const res = await this.$request('address', operation, data);
				this.$util.msg(res.msg);
				if(res.status === 1){
					this.$util.prePage().loadData();
					setTimeout(()=>{
						uni.navigateBack();
					}, 1000)
				}
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
			
			// add(id) {		
			// 	id++;
			// 	this.num=id;
			// },
			// // 减少商品数量
			// reduce(id) {
			// 	if(id>1){
			// 		if(this.num==id){
			// 			this.num--
			// 		}
			// 	}
			// 	else{
			// 		uni.showToast({
			// 			title:'至少购买一件商品',
			// 			icon:'none'
			// 		})
			// 	}
			// },
			
			
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
	
	// .update-count{
	// 	margin-top: 40rpx;
	// 	display: flex;
	// 	.reduce{
	// 		width: 40rpx;
	// 		height: 40rpx;
	// 		background: #F1ECE7;
	// 		border-radius: 21.6rpx;
	// 		border-radius: 21.6rpx;
	// 		color: #979797;
	// 		font-size: 50rpx;
	// 		text-align: center;
	// 	}
	// 	.add{
	// 		width: 40rpx;
	// 		height: 40rpx;
	// 		background: #F1ECE7;
	// 		border-radius: 21.6rpx;
	// 		border-radius: 21.6rpx;
	// 		color: #979797;
	// 		font-size: 40rpx;
	// 		text-align: center;
	// 	}
	// 	.cart-count{
	// 		width: 72rpx;
	// 		height: 40rpx;
	// 		background: #F1ECE7;
	// 		border-radius: 21.6rpx;
	// 		border-radius: 21.6rpx;
	// 		margin-left: 18rpx;
	// 		margin-right: 18rpx;
	// 		text-align: center;
	// 		line-height: 40rpx;
	// 	}
	// }
	// .add-reduce{
	// 	margin-top: 30rpx;
	// }
	
</style>
