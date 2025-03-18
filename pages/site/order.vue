<template>
	<view style="padding: 20rpx;">
		
		<view style="display: flex;flex-direction: row; justify-content: flex-end;width: 100%;padding: 20rpx;">
			<u-button @click="getOrderItem" type="primary">预约单</u-button>
		</view>
		
		<view v-if="!show">
			<view class="item-view" style="margin-top: 30rpx; " v-for="(item,index) in allOrder">
				<view class="order">
					<view>单号：{{item.id}}</view>
					<view>{{item.park.name}}</view>
					<view>{{stateNameArr[item.state]}}</view>
				</view>
				<view class="order">
					<view>{{item.item.itemName}}</view>
					<view>免费期:{{item.park.freeTime}}分钟</view>
					<view>费率:{{item.park.price}}元/分钟</view>
				</view>
				<view class="order">
					<view>停车:{{countTime(item)}}分钟</view>
					
					<view>总价:{{item.payType && item.payType == 2 ?'会员免单':((item.price||getPrice(item))+"元") }}</view>
				</view>
				<view class="order">
					<view>车牌:{{item.carNo}}</view>
					<view>入场:{{item.beginTime}}</view>
				</view>
				<view class="order">
					<view >离场时间:<text class="margin-l" :style="{color:(!item.state == 0)?'black':'red'}">{{item.endTime||"未离场"}}</text></view>
					<view v-if="item.state == 0"><u-button @click="check(item)" type="success" shape="circle" text="离场结算"></u-button></view>
					<view v-if="item.state == 1 && !item.comment"><u-button @click="commentShow(item)" type="primary" shape="circle" text="评价"></u-button></view>
					<view v-if="item.state == 1"><view class="text-green" @click="setCode(item)">开票</view></view>
				</view>
			</view>
		</view>
		
		<view v-if="show" class="item-view" style="position: fixed;top: 140rpx;left:20rpx;width: 710rpx;height: auto;">
			
			<view @click="showOr" style="color: red;text-align: right;padding: 15rpx;">关闭</view>
			<view class="item-view" style="margin-top: 30rpx;" v-for="(item,index) in allOrderItem">
				<view class="order">
					<view>ID：{{item.id}}</view>
					<view>停车场：{{item.parkName}}</view>
				</view>
				<view class="order">
					<view>{{item.itemName}}</view>
					<view>时间：{{item.orderTime}}</view>
				</view>
				<view class="v-flex v-r-center margin-t">
					<view class="v-flex" style="justify-content: space-around;width: 80%;" v-if="item.state==0">
						<u-button style="width: 100rpx;"  @click="book(item)" type="success" shape="circle" text="停车入场"></u-button>
						<view class="margin-l margin-r"></view>
						<u-button style="width: 100rpx;" @click="canCelBook(item)" type="primary" shape="circle" text="取消预约"></u-button>
					</view>
					<view v-else>{{
						
						["已预定","已完成","已取消","违约取消"][item.state||0]
						
					}}</view>
				</view>
			</view>
		</view>
		
		<u-modal :show="showCheck"  title="结算" confirm-text="支付" @confirm="subOrder">
			<view class="slot-content">
				<view class="padding-2 ">时长：{{countTime(selOrder)}}分钟</view>
				<view class="padding-2 ">免费时长：{{selOrder.park.freeTime}}分钟</view>
				<view class="padding-2 ">费率:{{selOrder.park.price}}元/分钟</view>
				<view class="padding-2 ">停车费:{{getPrice(selOrder)}}元</view>
				<view class="padding-2 margin-t">
					<view class="text-blue text-middle">支付方式</view>
					<view v-if="vipFlag==0">
						<view :class="payType==0?'selTab':''" @click="payType=0" style="border-radius: 20rpx;" class=" margin-l padding-10 text-middle v-flex v-r-left v-c-center"><u-icon name="weixin-fill" size="50" color="#53c21d"></u-icon><text class="margin-l">微信支付</text></view>
						<view :class="payType==1?'selTab':''" @click="payType=1" style="border-radius: 20rpx;" class="margin-l padding-10 text-middle v-flex v-r-left v-c-center"><u-icon name="rmb-circle-fill" size="45" color="#f1a532"></u-icon><text class="margin-l"></text>余额支付<text class="text-gray text-light">(剩余{{allUserInfo.vipInfo[0].balance}}元)</text></view>
					</view>
					<view v-else>
						{{ allUserInfo.vipInfo[0].vipType==1?'包月':'包年'}}会员免单
					</view>
				</view>
				<view class="padding-2 margin-t" v-if="vipFlag==0">
					<view class="text-blue text-middle">优惠</view>
					<view class="v-flex v-r-left v-c-center">使用积分：
						<view class="padding-10"><u--input
							placeholder="请输入积分数"
							type="number"
							v-model="vipScore"
							@change="change"
						  ></u--input></view>
					  </view>
					<view class="padding-10">可用积分：{{allUserInfo.vipInfo[0].vipScore}}<text class="text-gray text-light">(10分抵扣1分钟停车时)</text></view>
				</view>
				<view class="padding-2 margin-t">
					<view class="text-middle">合计：{{ vipFlag == 0 ? getPrice2(selOrder):0}}元</view>
				</view>	
			</view>
		</u-modal>
		<u-modal :show="showComment"  title="评价" confirm-text="提交评价" @confirm="subComment">
			<view class="slot-content" style="width: 100%;">
				<view class="padding-10">
				<u--input
					placeholder="请输入评价"
					type="text"
					v-model="comment.content"
				  ></u--input>
				  </view>
				  <view class="padding-10">
					  <u-radio-group v-model="comment.star" >
					  	<u-radio shape="circle" label="1分" name="1"></u-radio>
						<u-radio shape="circle" label="2分" name="2"></u-radio>
						<u-radio shape="circle" label="3分" name="3"></u-radio>
						<u-radio shape="circle" label="4分" name="4"></u-radio>
						<u-radio shape="circle" label="5分" name="5"></u-radio>
					  </u-radio-group>
				  </view>
				  
			</view>
		</u-modal>
		
		<u-modal :show="showCode"  title="电子发票" confirm-text="关闭" @confirm="showCode=false">
			<view class="slot-content" style="width: 100%;">
				<view class="canvas">
					<view class="canvas">
						<!-- 二维码插件 width height设置宽高 -->
						<canvas canvas-id="qrcode" :style="{width: `${qrcodeSize}px`, height: `${qrcodeSize}px`}" />
					</view>
					<view class="padding-2">订单号:{{selOrder.id}}</view>
					<view class="padding-2">发票编号:{{"24312000000214"+selOrder.id}}</view>
					<view class="padding-2">发票金额:{{selOrder.price}}</view>
					<view class="padding-2">开票时间:{{selOrder.updateTime}}</view>	
					<view class=" text-gray">微信扫描二维码获取发票</view>	
					<view class=" text-gray">包月订单以实际应缴金额开票</view>
				</view>  
			</view>
		</u-modal>
		
	</view>
</template>

<script>
	import appRequest from "@/common/appRequestUrl.js"
	import uQRCode from '@/common/uqrcode.js'
	export default {
		data() {
			return {
				showCode:false,
				// 二维码标识串
				qrcodeText: 'eoruw20230528',
				// 二维码尺寸
				qrcodeSize: 136,

				// 最终生成的二维码图片
				qrcodeSrc: '',

				myFormatData: {
					'yyh': 'eoruw20230528',
					'bsdmc': '窗口1',
					'Yylxmc': '租金缴纳',
					'yyrq': '预约日期'
				},
				showComment:false,
				show:false,
				allOrder:[],
				userInfo:"",
				stateNameArr:['已下单','已离场','已取消'],
				current: 0,
				allOrderItem:[],
				showCheck:false,
				allUserInfo:"",
				selOrder:"",
				vipScore:"",
				payType:0,
				totalPrice:0,
				vipFlag:0,
				comment:{
					content:"",
					star:1,
					orderFk:"",
					parkFk:""
				}
			};
		},onShow:function(){
			let user = appRequest.getUserInfo();
			if(!user.uid){
				uni.clearStorage()
				this.userInfo=false;
				uni.showModal({
					title:"信息",
					content:"登录后访问",
					showCancel:false,
					success:function(){
						uni.clearStorage();
						uni.navigateTo({
							url:"/pages/simple/login"
						})
					}
				})
			}else{
				this.userInfo = user;
				this.getOrder();
				this.getAllInfo();
			}
		},
		onLoad:function(){
			let user = appRequest.getUserInfo();
			if(!user.uid){
				uni.clearStorage()
				this.userInfo=false;
			}else{
				this.userInfo = user;
			}
			
		},methods:{
			setCode(item){
				if(item.price<=0){
					this.$api.msg("免费订单，无需开票")
				}else{
					this.showCode = true;
					this.selOrder = item;
					this.setQrcode(item);
				}
			},
			getCode(item){
				if(item.updateTime){
					let date = item.updateTime.replace(/-/g,"");
					return "01,32,,24312000000214"+item.id+","+(item.price||0)+","+date.slice(0,9)+",,88B3";
				}
				return item.id||"";
			},
			setQrcode(item) {
				uni.showLoading({
					title: '发票生成中',
					mask: true
				})
				let date = item.updateTime.replace(/-/g,"");
				uQRCode.make({
					canvasId: 'qrcode',
					text: this.getCode(item),
					size: this.qrcodeSize,
					margin: 10,
					success: res => {
						this.qrcodeSrc = res
						console.log('qrcodeSrc = ' + this.qrcodeSrc);
					},
					complete: () => {
						uni.hideLoading()
					}
				})
			},
			subComment(){
				console.log(JSON.stringify(this.comment))
				if(!this.comment.star || !this.comment.content){
					uni.showToast({
						title:"请填写完整",
						icon:"none"
					})
					return;
				}
				this.comment.orderFk = this.selOrder.id;
				this.comment.parkFk = this.selOrder.parkFk;
				let _this = this;
				appRequest.request({
				  data:this.comment,
				  url: appRequest.urlMap.addNmCarParkComment,
				  success: function(res) {
					if (res.data.code == 200) {
					  uni.showToast({
					  	title:res.data.msg,
					  	icon:"none"
					  })
					  _this.getOrder();
					  _this.showComment = false;
					  _this.comment = {
							content:"",
							star:1,
							orderFk:"",
							parkFk:""
						}
					} else {
					  uni.showToast({
					  	title:res.data.msg,
					  	icon:"none"
					  })
					}
				  },
				  fail: function(res) {
					uni.showToast({
						title:"网络异常",
						icon:"none"
					})
				  }
				});
			},
			commentShow(item){
				this.showComment = true;
				this.selOrder = item;
			},
			book(item){
				
				if(!this.checkTime(item)){
					this.$api.msg("超期未进站，请等待系统结算取消，将扣除10积分。")
					return;
				}
				
				let _this = this;
				appRequest.request({
				  data: {
					carNo: item.carNo,
					itemFk:item.itemFk,
					parkFk: "",
					creater: _this.userInfo.uid,
					bookId:item.id,
					actType:3
				  },
				  url: appRequest.urlMap.subCarNoInfo,
				  success: function(res) {
					if (res.data.code == 200) {
					  uni.showToast({
					  	title:res.data.msg,
					  	icon:"none"
					  })
					  _this.getOrderItem();
					} else {
					  uni.showToast({
					  	title:res.data.msg,
					  	icon:"none"
					  })
					}
				  },
				  fail: function(res) {
					uni.showToast({
						title:"网络异常",
						icon:"none"
					})
				  }
				});
			},
			subOrder(){
				if(this.payType == 1 && this.totalPrice > this.allUserInfo.vipInfo[0].balance){
					uni.showToast({
						title:"余额不足,请更换支付方式",
						icon:"none",
						success:function(){
							_this.vipScore = _this.allUserInfo.vipInfo[0].vipScore;
						}
					})
				}else{
					this.out(this.selOrder);
				}
				
			},
			change(val){
				let _this = this;
				let reg = /^[0-9]\d*$/;
				if(!reg.test(val)){
				 uni.showToast({
						title:"只能输入正整数",
						icon:"none",
						success() {
							_this.vipScore = "";
						}
					})
					
					return;
				}
				let score = parseInt(val);
				if(score > this.allUserInfo.vipInfo[0].vipScore){
					
					uni.showToast({
						title:"使用积分达到上限",
						icon:"none",
						success:function(){
							_this.vipScore = _this.allUserInfo.vipInfo[0].vipScore;
						}
					})
				}
				this.getPrice2(this.selOrder)
				if(this.totalPrice < 0){
					let _this = this;
					uni.showToast({
						title:"使用积分过多",
						icon:"none",
						success:function(){
							_this.vipScore = "";
						}
					})
				}
				
			},
			getPrice2(item){
				let price = this.getPrice(item);
				let count = this.vipScore>0 ? (price - (this.vipScore/10*item.park.price).toFixed(2)).toFixed(2) : price;
				this.totalPrice = count < 0 ? price:count;
				return this.totalPrice;
			},
			getPrice(item){
				if(!item){
					return 0;
				}
				item.park.freeTime = item.park.freeTime ||0;
				let price = ( (this.countTime(item) - item.park.freeTime)*item.park.price).toFixed(2);
				return price < 0 ?0:price;
			},
			check(item){
				this.selOrder = item;
				this.showCheck = true;
				this.totalPrice = 0;
				this.vipScore = "";
				this.payType = 0;
				this.balance = 0;
				if(this.allUserInfo.vipInfo[0].balance < 10){
					uni.showModal({
						title:"温馨提醒",
						content:"余额小于10元，建议充值哦"
					})
				}
			},
			out(item){
				let _this = this;
				let type = this.allUserInfo.vipInfo[0].vipType;//判定会员
				appRequest.request({
				  data: {
					id: item.id,
					carNo: item.carNo,   
					itemFk: item.itemFk,
					parkFk: item.parkFk,
					creater: _this.userInfo.uid,
					price:this.totalPrice,
					vipScore:this.vipScore,
					payType: !type ? this.payType:2,
					balance:this.payType == 1 ?this.totalPrice:0,
					actType:2
				  },
				  url: appRequest.urlMap.subCarNoInfo,
				  success: function(res) {
					if (res.data.code == 200) {
					  _this.showCheck = false;
					  uni.showToast({
					  	title: res.data.msg,
					  	icon:"none"
					  })
					  _this.carNO = "";
					  _this.getOrder();
					  _this.getAllInfo();
					} else {
					  uni.showToast({
					  	title:res.data.msg,
					  	icon:"none"
					  })
					}
				  },
				  fail: function(res) {
					uni.showToast({
						title:"网络异常",
						icon:"none"
					})
				  }
				});
			},
			showOr(){
				this.show = false;
				this.getOrder();
			},
			getOrderItem(){
				
				let _this = this;
				appRequest.request({
					method: "POST",
					url: appRequest.urlMap.findNmCarParkBookingList,
					data:{
						validFlag:1,
						creater:this.userInfo.uid},
					success: function(res) {
						_this.allOrderItem = res.data.data;
						if(_this.allOrderItem.length == 0){
							uni.showModal({
								title:"信息",
								content:"暂无预约",
								showCancel:false,
								success:function(){
									
								}
							})
						}else{
							_this.show = true;
						}
					},
					fail: function(res) {
						
						uni.showToast({
							title:"网络异常",
							icon:"none"
						})
						
					}
				})
			},
			countTime(row){
				if(!row){
					return 0;
				}
			  row.beginTime = row.beginTime.replace(/-/g,"/");
			  if(row.endTime){
				  row.endTime = row.endTime.replace(/-/g,"/");
			  }
			  
			  let begin = new Date(row.beginTime).getTime();
			  let end = row.endTime ? new Date(row.endTime).getTime() : new Date().getTime();
				let diff = end - begin;
				return (diff/(1000*60)).toFixed(2);
			},
			getOrder(){
				let _this = this;
				appRequest.request({
					method: "POST",
					url: appRequest.urlMap.findNmCarParkOrderList,
					data:{
						validFlag:1,
						creater:this.userInfo.uid,
					},
					success: function(res) {
						_this.allOrder = res.data.data;
						if(_this.allOrder.length == 0){
							uni.showModal({
								title:"信息",
								content:"暂无订单",
								showCancel:false,
								success:function(){
									
								}
							})
						}
					},
					fail: function(res) {
						
						uni.showToast({
							title:"网络异常",
							icon:"none"
						})
						
					}
				})
			},getAllInfo(){
				let _this = this;
				appRequest.request({
					method: "POST",
					url: appRequest.urlMap.getUserAllInfo,
					header: {
						'content-type': 'application/x-www-form-urlencoded'
					},
					data:{
						uid:_this.userInfo.uid
					},
					success: function(res) {
						_this.allUserInfo = res.data.data
						_this.vipFlag = _this.checkVip();
					},
					fail: function(res) {
						
						uni.showToast({
							title:"网络异常",
							icon:"none"
						})
						
					}
				})
				
			},checkVip(){
				if(this.allUserInfo.vipInfo){
					let info = this.allUserInfo.vipInfo[0];
					if(info.vipType == 1 || info.vipType == 2){
						if(new Date().getTime() < new Date(info.vipEnd).getTime()){
							return info.vipType;
						}
					}
				}
				return 0;
			},checkTime(item,type){
				let now = new Date();
				let order = new Date(item.orderTime);
				if(type){
					return now.getTime() <= order.getTime();
				}else{
					return order.getTime() - now.getTime() > 10*60*1000;
				}
			},canCelBook(item){
				
				if(!this.checkTime(item,1)){
					this.$api.msg("预约时间到达前10分钟才可取消")
					return;
				}
				
				let _this = this;
				item.state = 2;
				appRequest.request({
					method: "POST",
					url: appRequest.urlMap.editNmCarParkBooking,
					data:item,
					success: function(res) {
						if(res.data.code == '200'){
							uni.showToast({
								title:"取消成功",
								icon:"none"
							})
							_this.getOrder();
						}
					},
					fail: function(res) {
						
						uni.showToast({
							title:"网络异常",
							icon:"none"
						})
						
					}
				})
			}
		}
	}
</script>

<style lang="scss">
	.order{
		display: flex;
		flex-direction: row;
		justify-content: space-between;
		align-items: center;
	}
	.item-view {
		padding: 30rpx;
		border-radius: 15px;
		box-shadow: 3px 3px 20px lightgray;
		font-size: 30rpx;
	}
	.selTab{
		background-color: rgba(0, 0, 0, 0.1);
	}
</style>
