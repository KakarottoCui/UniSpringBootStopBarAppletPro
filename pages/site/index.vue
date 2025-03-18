<template>
	<view class="warp">
		<!-- <u-swiper
			:list="list1"
			indicator
			height="368rpx"
			indicatorMode="dot"
			@click="click"
		></u-swiper>
		<view class="u-m-15"></view> -->
		
		<u-search placeholder="目的地/停车场搜索" v-model="keyword" @custom="getParkLocation" @search="getParkLocation"></u-search>
		
		<view class="margin-t" style="border-radius: 30rpx;">
			<map @markertap="tapMap" style="width: 100%; height: 50vh;" :latitude="position.lat" :longitude="position.lng" :markers="covers"></map>
		</view>
		<u-notice-bar @click="toNotice" :text="notice" speed="40" color="#3f80de" bgColor="#ffffff"></u-notice-bar>
		
		<view class="all_title_box">
			<view class="title">停车场列表</view>
		</view>
		
		<!-- <view class="car_park u-m-t-30" style="width: 100%;" >
			<picker @change="car_confirm" :value="index" :range="columns" style="width: 100%;">
				<view class="uni-input">{{columns[index]}}</view>
			</picker>
			<u-icon name="arrow-down" color="#999" size="16"></u-icon>
		</view> -->
		
		<!-- <u-subsection :list="tagList" :current="current" @change="sectionChange"></u-subsection> -->
		<view class="view-shadow v-flex v-r-center" style="padding: 10rpx; border-radius: 20rpx;text-align: center;">
			<u-tabs :list="tagList" @click="sectionChange" scrollable="false"></u-tabs>
		</view>
		
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
					<view class="v-flex v-r-left v-c-center" style="width: 80%;max-width: 65vw;">
						<view style="min-width: 90rpx;font-size: 28rpx;"  >位置：</view>
						<text class="text-ellipsis" style="font-size: 28rpx;max-width: 300rpx;" >{{item.address}}</text>
					</view>
					<view class="margin-l v-flex v-r-between" style="min-width: 300rpx;font-size: 28rpx;flex-wrap: nowrap;">距离：{{item.distance}}</view>
				</view>
				<view class="v-flex v-r-between padding-2">
					<view>空位：{{getSpace(item.parkItem)}}/{{item.num}}</view>
				</view>
				<view class="v-flex v-r-between padding-2 text-bold">
					<view class="text-blue" style="text-shadow: none;"> {{ item.price <= 0 ?'免费停车':(item.freeTime+"分免费，"+(item.price * 60)+"元/时") }}</view>
					<view @click="toItem(item)">详情>></view>
				</view>
			</view>
			<!-- <view class="lattice_box" style="width: 100%;">
				<view class="park-content">
					<view @click="order(item)" v-for="(item,index) in carParkItems" :key="index" class="demo-layout" :class="item.stat == 1?'parking':''">
						<view>{{item.itemName}}</view>
						<view>{{item.stat == 0?'空置':(item.stat==1?'使用中':'已预约')}}</view>
					</view>
				</view>
			</view>	 -->
		</view>
		
		
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
				covers:[],
				keyword:"",
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
				}
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
			console.log("我就看看地址行不行");
			let _this = this;
			
		},
		onLoad:function(){
			let _this = this;
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
			
			this.getOrderItem();
			this.getNotice();
		},
		methods: {
			getParkLocation(name){
				let _this = this;
				appRequest.request({
					method: "POST",
					url: appRequest.urlMap.findNmCarParkList,
					data:{
						name:name
					},
					success: function(res) {
						if(res.data.code == 200){
							let data = res.data.data;
							if(data && data.length > 0){
								_this.position = {lat:data[0].addX,lng:data[0].addY};
								uni.showToast({
									title:"主地图已定位到："+data[0].name+",请查看",
									duration:3000,
									icon:"none"
								})
							}else{
								_this.search(name);
							}
							
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
			search(value){
				let _this = this;
				appRequest.request({
					method: "POST",
					url: appRequest.urlMap.getLocationWithName,
					data:{
						name:value
					},
					header: {
						'content-type': 'application/x-www-form-urlencoded'
					},
					success: function(res) {
						if(res.data.code == 200){
							console.log(res.data)
							let data = res.data.data;
							if(data){
								_this.position = data.location;
								uni.showToast({
									title:"主地图已定位到："+data.title+",请查看周边推荐停车场",
									duration:3000,
									icon:"none"
								})
								_this.covers.push({
									id:"",
									latitude:_this.position.lat,
									longitude:_this.position.lng,
									label:{content:"目的地"+data.title,padding:2}
								})
							}else{
								uni.showToast({
									title:"地址无法定位，请输入完整地址",
									duration:3000,
									icon:"none"
								})
								
							}
							
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
			tapMap(data){
				if(data.markerId){
				this.toItem({id:data.markerId});	
				}else{
					uni.showToast({
						title:"未选择停车场",
						icon:"none"
					})
				}
			},
			toItem(item){
				uni.navigateTo({
					url:"/pages/site/parkItem?id="+item.id
				})
			},
			sectionChange(item){
				let index = item.index;
				let _this = this;
				let val = _this.tagList[index].name.indexOf("↑") > -1;
				if(index == this.current){
					this.sortObj = {
						type:index,
						asc: !val
					}
					_this.tagList[index].name = item.name.replace(!val?'↓':'↑',val?'↓':'↑');
					console.log(_this.tagList[index].name)
				}else{
					this.sortObj = {
						type:index,
						asc: val
					}
				}
				this.current = index;
				this.getParkInfo();
			},
			order(item){
				let _this = this;
				if(!this.userInfo){
					uni.showToast({
						title:"操作前请登录",
						icon:"none"
					})
					return;
				}
				if(item.stat != 0){
					uni.showToast({
						title:"车位已被占用或预约",
						icon:"none"
					})
					return;
				}
				
				uni.showModal({
					title:"预约停车",
					content:"是否预约"+_this.selObj.name+","+item.itemName+"今日停车?",
					success:function(res){
						if(res.confirm){
							appRequest.request({
								method: "POST",
								url: appRequest.urlMap.addNmCarParkBooking,
								data:{
									validFlag:1,
									itemFk:item.itemId,
									itemIndex:item.itemIndex,
									itemName:item.itemName,
									parkName:_this.selObj.name,
									state:0
								},
								success: function(res) {
									if(res.data.code == 200){
										uni.showToast({
											title:"预约成功",
											icon:"none"
										});
										_this.getParkItem(item.pid);
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
					success: function(res) {
						_this.parkInfo = res.data.data;
						if(_this.parkInfo && typeof(_this.parkInfo) == 'object'){
							_this.columns = [];
							_this.covers = [];
							_this.parkInfo.map(function(item,index){
								_this.columns.push(item.name)
								let dis = _this.getDistance(_this.position.lat,_this.position.lng,item.addX,item.addY);
								item['distance'] = dis;
								_this.getParkItem(item.id,item);
								
								_this.covers.push({
									id:item.id,
									latitude:item.addX,
									longitude:item.addY,
									label:{content:item.name,padding:2,bgColor:"#3f80de",borderRadius:5,color:"#fff"
									},
									iconPath:"/static/park.png"
								})
							})
							_this.selObj = _this.parkInfo[0];
							_this.car_number = _this.columns[0];
							_this.sortData()
							
							//_this.getParkItem(_this.parkInfo[0].id);
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
			},
			sortData(){
				let _this = this;
				console.log(JSON.stringify(_this.sortObj))
				
				_this.parkInfo.sort(function(a,b){
					console.log(JSON.stringify(a))
					if(_this.sortObj.type == 0 && a.distance && b.distance){
						var data1 = parseFloat(a.distance.replace(/[km|m]/g,''));
						var data2 =  parseFloat(b.distance.replace(/[km|m]/g,''))
						return _this.sortObj.asc ?(data1-data2):(data2-data1);
					}
					if(_this.sortObj.type == 1 && a.price && b.price){
						return _this.sortObj.asc ?(a.price - b.price):(b.price - a.price);
					}
					if(_this.sortObj.type == 2 && a.freeTime && b.freeTime){
						return _this.sortObj.asc ?(b.freeTime - a.freeTime):(a.freeTime - b.freeTime);
					}
					if(_this.sortObj.type == 3 && a.star && b.star){
						return _this.sortObj.asc ?(a.star - b.star):(b.star - a.star);
					}
				})
				
			},getOrderItem(){
				let _this = this;
				appRequest.request({
					method: "POST",
					url: appRequest.urlMap.getUserParkItemCheck,
					success: function(res) {
						if(res.data.code == 200){
							if(res.data.data){
								uni.showModal({
									title:"车位预警",
									content:"你最常停放的"+res.data.data.name+"，车位可能不足3个，请注意哦",
									showCancel:false
								})
							}
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
		justify-content: space-between;
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
