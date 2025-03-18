<template>
	<view class="warp">
		
		<view class="all_title_box">
			<view class="title">停车场信息</view>
		</view>
		
		<!-- <view class="car_park u-m-t-30" style="width: 100%;" >
			<picker @change="car_confirm" :value="index" :range="columns" style="width: 100%;">
				<view class="uni-input">{{columns[index]}}</view>
			</picker>
			<u-icon name="arrow-down" color="#999" size="16"></u-icon>
		</view> -->
		
		<!-- <u-subsection :list="tagList" :current="current" @change="sectionChange"></u-subsection> -->
		
		<view v-for="(item,index) in parkInfo" :key="index" :class="'card-bg'+(index%2==0?'':2) " class="view-shadow padding-15 margin-t text-shadow text-bold" style="border-radius: 30rpx;width:100%;">
			<view style="width: 100%;max-width: 80vw;">
				<view class="v-flex v-r-between padding-2 text-shadow">
					 <view class="v-flex v-r-left padding-2">
						 <u-icon name="car-fill" color="#2b85e4" size="30"></u-icon>
						 <view class="text-bold text-middle " style="padding-left: 10rpx;">{{item.name}}</view>
					 </view>
					 <view class="v-flex v-c-center" style="width: 20%;"><u-icon color="#ff9900" size="20" name="star-fill"></u-icon><text style="margin-left: 10rpx;">{{item.star||0}}</text></view>
				</view>
				<view class="v-flex v-r-between padding-2" style="width: 90vw;">
					<view class="v-flex v-r-left v-c-center">
						<view style="min-width: 90rpx;"  >位置：</view>
						<text class="text-ellipsis" >{{item.address}}</text>
					</view>
					
				</view>
				<view class="v-flex v-r-between padding-2" style="width: 90vw;">
					<view class="v-flex v-r-between" style="min-width: 300rpx;flex-wrap: nowrap;">距离：{{item.distance}}</view>
				</view>
				<view class="v-flex v-r-between padding-2">
					<view>空位：{{getSpace(item.parkItem)}}/{{item.num}}</view>
				</view>
				<view class="v-flex v-r-between padding-2 text-bold">
					<view class="text-blue" style="text-shadow: none;"> {{ item.price <= 0 ?'免费停车':(item.freeTime+"分免费，"+(item.price * 60)+"元/时") }}</view>
					<view class=" v-flex v-r-center v-c-center" style="text-shadow: none;"> 
						<view class="margin-r v-flex v-r-center v-c-center" @click="collect()"><u-icon color="#f6ad55" name="star-fill"></u-icon>收藏</view>
						<view class="margin-l v-flex v-r-center v-c-center" @click="toMap"><u-icon color="#409EFF" name="map-fill"></u-icon>导航</view>
					</view>
					
				</view>
			</view>
			
		</view>
		<view class="all_title_box">
			<view class="title">车位信息</view>
		</view>
		<view class="lattice_box" style="width: 100%;">
			<view class="park-content">
				<view @click="order(item)" v-for="(item,index) in carParkItems" :key="index" class="demo-layout" :class="item.stat == 1?'parking':''">
					<view>{{item.itemName}}</view>
					<view>{{item.stat == 0?'空置':(item.stat==1?'使用中':'已预约')}}</view>
				</view>
			</view>
		</view>	
		<view class="all_title_box">
			<view class="title">评价</view>
		</view>
		<view>
			<u-cell-group>
					<u-cell v-for="(item,index) in comment" :key="index" icon="chat-fill" :title="item.content" :value="item.star+'分'" :label="item.createTime"></u-cell>
				</u-cell-group>
		</view>
		<u-popup :show="showCars" mode="center" :round="10" :closeable="true" @close="showCars=false">
			<view style="width: 500rpx;margin-top: 100rpx;">
				<u-cell-group>
					<u-cell v-for="(item,index) in allUserInfo.userItem" :key="index" icon="car-fill" :title="item.carNo" value="选择" @tap="setCar(item)"></u-cell>
				</u-cell-group>
			</view>
		</u-popup>
		<u-datetime-picker
		:minHour="get20Min().getHours()"
		:minMinute="get20Min().getMinutes()"
		                :show="showTime"
		                v-model="selTime"
		                mode="time" @confirm="subTime" @cancel="showTime=false"></u-datetime-picker>
	</view>
</template>

<script>
	
	import appRequest from "@/common/appRequestUrl.js"
	// import QQMapWX from "@/common/qqmap-wx-jssdk.js"
	// let qqmapsdk;
	// 	const mapInit = () => {
	// 		qqmapsdk = new QQMapWX({
	// 			key: "26WBZ-CQG35-75FIB-ICLTD-UUP5E-E2FQX"
	// 		})
	// 	}
	export default {
		data() {
			return {
				selTime:"",
				showTime:false,
				carNo:"",
				showCars:false,
				parkInfo:[],
				notice:"",
				index:0,
				selObj:{},
				commonIcon:"/common/car.png",
				list1: [
					'/static/coolc/banner_03.jpg'
				],
				text1: '请您尽快处理；',
				car_number: "",
				show: false,
				columns: [
					[]
				],
				table: [{
						time: '刚刚',
						title: '总校公共停车场: 粤M12345 入场'
					}
				],
				carParkItems:[],
				userInfo:"",
				tagList:[{
                    name: '距离↑'}, {
                    name: '价格↑'}, {
                    name: '免费↑'},{
                    name: '评分↓'}],
				current:0,
				sortObj:{type:0,asc:true},
				position:{
					lat:0,
					lng:0
				},
				parkId:0,
				allUserInfo:{},
				nowH:0,
				nowM:0,
				res:"",
				selItem:"",
				comment:[],
				orderFlag:false,
				bookFlag:false
			}
		},
		onShow:function(){
			let user = appRequest.getUserInfo();
			if(!user.uid){
				uni.clearStorage()
				this.userInfo=false;
			}else{
				this.userInfo = user;
			}
			let _this = this;
		},
		onLoad:function(options){
			let _this = this;
			this.parkId = options.id;
			wx.getLocation({
			 type: 'wgs84',
			 success (res) {
			   _this.position.lat = res.latitude
			   _this.position.lng = res.longitude
			   
			   console.log("地址"+JSON.stringify(res));
			   _this.getParkInfo();
			 },fail:function(res){
			 	uni.showToast({
			 		title:"获取定位失败",
			 		icon:"none"
			 	})
			 }
			})
			
			this.getNotice();
			
		},
		methods: {
			collect(){
				
				let _this = this;
				appRequest.request({
					method: "POST",
					url: appRequest.urlMap.addNmCarCollect,
					data:{
						parkName:this.parkInfo[0].name,
						parkFk:this.parkInfo[0].id,
						validFlag:1,
						creater:_this.userInfo.uid
					},
					success: function(res) {
						uni.showToast({
							title:res.data.msg,
							icon:"none"
						})
					},
					fail: function(res) {
						
						uni.showToast({
							title:"网络异常",
							icon:"none"
						})
						
					}
				})
			},
			toMap(){
				wx.openLocation({
				  latitude:this.parkInfo[0].addX,
				  longitude: this.parkInfo[0].addY, // 经度
				  name:this.parkInfo[0].address,
				  scale: 10, // 缩放比例          
				})
			},
			getComment(){
				let _this = this;
				appRequest.request({
					method: "POST",
					url: appRequest.urlMap.findNmCarParkCommentList,
					data:{
						parkFk:this.parkInfo[0].id,
						validFlag:1
					},
					success: function(res) {
						_this.comment = res.data.data;
					},
					fail: function(res) {
						
						uni.showToast({
							title:"网络异常",
							icon:"none"
						})
						
					}
				})
				
			},
			setCar(item){
				
				console.log(JSON.stringify(item))
				
				this.carNo=item.carNo
				this.showCars = false;
				this.chooseOrder(item);
			},
			get20Min(){
				let date = new Date();
				date.setMinutes(new Date().getMinutes() + 20);
				return date;
			},
			order(item){
				
				let _this = this;
				if(!this.userInfo){
					uni.showToast({
						title:"操作前请登录",
						icon:"none"
					})
					uni.clearStorage();
					uni.navigateTo({
						url:"/pages/simple/login"
					})
					return;
				}
				
				_this.getAllInfo();
				_this.selItem = item;
				if(item.stat != 0){
					uni.showToast({
						title:"车位已被占用或预约",
						icon:"none"
					})
					return;
				}
				
				uni.showModal({
					title:"操作",
					content:"请问您要停车还是预约？",
					confirmText:"停车",
					cancelText:"预约",
					success:function(res){
						if(_this.allUserInfo.userItem.length == 0){
							uni.showToast({
								title:"您未添加车辆车牌，无法使用",
								icon:"none"
							})
							return;
						}
						_this.showCars = true;
						_this.orderFlag = false;
						_this.bookFlag = false;
						_this.orderFlag = res.confirm;
						_this.bookFlag = !res.confirm;
					}
				})
				
				
			},
			chooseOrder(item){
				console.log(JSON.stringify(item))
				if(this.orderFlag){
					//设定停车
					this.subCarNo(this.selItem,this.carNo);
					
				}else if(this.bookFlag){
					this.selTime = "";
					this.showTime = true;
				}
			},
			subTime(time){
				let _this = this;
				_this.selTime = time.value;
				uni.showModal({
					title:"停车",
					content:"是否预约"+_this.selObj.name+","+_this.selItem.itemName+"，"+_this.selTime+"停车?",
					success:function(res){
						if(res.confirm){
							appRequest.request({
								method: "POST",
								url: appRequest.urlMap.addNmCarParkBooking,
								data:{
									validFlag:1,
									itemFk:_this.selItem.itemId,
									itemIndex:_this.selItem.itemIndex,
									itemName:_this.selItem.itemName,
									parkName:_this.selObj.name,
									state:0,
									ot:_this.selTime,
									carNo:_this.carNo
								},
								success: function(res) {
									console.log(JSON.stringify(res))
									if(res.data.code == 200){
										uni.showToast({
											title:"预约成功",
											icon:"none"
										});
										_this.getParkItem(_this.selItem.pid);
										_this.showTime = false;
										_this.selTime = "";
									}else{
										uni.showModal({
											title:"提醒",
											content:res.data.msg,
											showCancel:true
										});
										_this.showTime = false;
										_this.selTime = "";
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
				})
			},
			click(e) {
				let urls = '';
				if(e == 0) {
					urls = "";
				}
				if(e== 1) {
					urls = "/pages/site/price_details"
				}
				if(e== 2) {
					urls = "/pages/site/car_log"
				}
				
				//跳转页面
				uni.navigateTo({
					url: urls
				});
			},
			car_confirm(e) {
				this.selObj = this.parkInfo[e.detail.value];
				this.getParkItem(this.selObj.id);
			},
			toNotice(){
				uni.navigateTo({
					url:"/pages/site/notice"
				})
			},
			getNotice(){
				let _this = this;
				appRequest.request({
					method: "POST",
					url: appRequest.urlMap.findSysNoticeList,
					data:{
						validFlag:1
					},
					success: function(res) {
						var data = res.data.data;
						_this.notice = data[data.length - 1].content;
						
					},
					fail: function(res) {
						
						uni.showToast({
							title:"网络异常",
							icon:"none"
						})
						
					}
				})
			},
			getParkItem(parkFk,item){
				let _this = this;
				appRequest.request({
					method: "POST",
					url: appRequest.urlMap.findNmParkItemList,
					data:{
						pid:parkFk,
						validFlag:1
					},
					success: function(res) {
						console.log(res.data.data[0].itemId)
						_this.carParkItems = res.data.data;
						if(item){
							item['parkItem'] = _this.carParkItems;
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
			getParkInfo(){
				let _this = this;
				appRequest.request({
					method: "POST",
					url: appRequest.urlMap.findNmCarParkList,
					data:{
						id:this.parkId
					},
					success: function(res) {
						_this.parkInfo = res.data.data;
						if(_this.parkInfo && typeof(_this.parkInfo) == 'object'){
							_this.columns = [];
							_this.parkInfo.map(function(item,index){
								_this.columns.push(item.name)
								let dis = _this.getDistance(_this.position.lat,_this.position.lng,item.addX,item.addY);
								item['distance'] = dis;
								_this.getParkItem(item.id,item);
							})
							_this.selObj = _this.parkInfo[0];
							_this.car_number = _this.columns[0];
							_this.getComment();
						}
					},
					fail: function(res) {
						
						uni.showToast({
							title:"网络异常",
							icon:"none"
						})
						
					}
				})
				
			},checkIndex(){
				let _this = this;
				_this.parkInfo.map(function(item,index){
					if((item.name+"("+item.orgName+")") == _this.car_number){
						_this.selObj = item;
					}
				})
			},getDistance(lat1, lng1, lat2, lng2) {
				function rad(d) {
					return d * Math.PI / 180.0;
				}
				var radLat1 = rad(lat1);
				var radLat2 = rad(lat2);
				var a = radLat1 - radLat2;
				var b = rad(lng1) - rad(lng2);
				var s = 2 * Math.asin(Math.sqrt(Math.pow(Math.sin(a / 2), 2) +
					Math.cos(radLat1) * Math.cos(radLat2) * Math.pow(Math.sin(b / 2), 2)));
				s = s * 6378.137; // EARTH_RADIUS;
				s = Math.round(s * 10000) / 10000; //输出为公里

				var distance = s;
				var distance_str = "";

				if (parseInt(distance) >= 1) {
					distance_str = distance.toFixed(1) + "km";
				} else {
					distance_str = (distance * 1000).toFixed(1) + "m";
				}
				return distance_str;
			},
			getSpace(item){
				let num = 0;
				//console.log(JSON.stringify(item))
				if(item){
					item.map(function(item,index){
						if(item.stat == 0){
							num ++;
						}
					})
				}
				
				return num;
			},subCarNo(item,carNo){
				let _this = this;
				appRequest.request({
				  data: {
					carNo: carNo,
					itemFk: _this.selItem.itemId,
					parkFk: _this.parkInfo[0].id,
					creater: _this.userInfo.uid,
					actType:1
				  },
				  url: appRequest.urlMap.subCarNoInfo,
				  success: function(res) {
					if (res.data.code == 200) {
					  uni.showToast({
					  	title:"车牌号："+carNo +"，"+ res.data.msg,
					  	icon:"none"
					  })
					  _this.itemFk = "";
					  _this.parkFk = "";
					  _this.carNO = "";
					  _this.getParkInfo();
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
	.warp {
		width: 100vw;
		padding: 0 30rpx 30rpx 30rpx;
		background: #fff;
	}
	
	.tbody {
		height: 500rpx;
		overflow-y: auto;
		overflow-x: hidden;
	}
	.w1 {
		width: 28%;
		padding-left: 0rpx!important;
		padding-right: 0rpx!important;
	}
	.w2 {
		width: 72%;
		text-align: left;
	}
	.wrap{
		width:100%;
	}
	.demo-layout{
		//box-shadow: 1px 1px 20px #19be6b;
		box-shadow: 1px 1px 15px rgba(0, 0, 0, 0.3);
		height: 80rpx;
		width: 140rpx;
		padding: 20rpx;
		// display: flex;
		// flex-direction: column;
		// justify-content: center;
		// align-items: center;
		border-radius: 10rpx;
		font-size: 20rpx;
		// color: #19be6b;
		margin: 15rpx 15rpx;
	}
	.parking{
		color: #fa3534;
		box-shadow: 1px 1px 15px rgba(250, 53, 52, 0.6);
	}
	.park-content{
		width: 100%;
		display: flex;
		flex-direction: row;
		flex-wrap: wrap; 
		justify-content: space-around;
		align-items: center;
		padding: 10rpx;
		margin-top: 30rpx;
	}
	.card-bg{
		background: linear-gradient(135deg, #6CACFF, #8DEBFF);
		color: #fff;
	}
	.card-bg2{
		background: linear-gradient(135deg, #41C7AF, #54E38E);
		color: #fff;
	}
</style>
