<template>
	<view>
		<view class="warp_user">
			<view class="top_user_box">
				<image :src="getPic(userInfo.pic)"></image>
				<view class="user_info">
					<view class="username">{{userInfo.nickName}}({{!userInfo.uid?'请登录':(vipInfo == 0?'普通会员':(vipInfo==1?'包月会员':'包年会员'))}})</view>
					<view class="carNo" v-if="userInfo.uid">积分：{{allUserInfo.vipInfo[0].vipScore}}</view>
					<view class="carNo" v-if="userInfo.uid">余额：{{allUserInfo.vipInfo[0].balance}}</view>
				</view>
			</view>

			<view class="icon_boxs">
				<navigator class="i" hover-class="none" @click="showCarInfo">
					<view class="icon_box">
						<u-icon name="grid-fill" color="#fff" size="32"></u-icon>
					</view>
					<text class="n">车辆信息</text>
				</navigator>
				<navigator class="i" hover-class="none" @click="showUserInfo">
					<view class="icon_box">
						<u-icon name="email-fill" color="#fff" size="32"></u-icon>
					</view>
					<text class="n">个人信息</text>
				</navigator>
			</view>
		</view>
		<view>
			<u-cell-group :border="false">
				<u-cell icon="setting" style="margin: 10rpx 0;" @click="info" :isLink="true" title="应用信息"></u-cell>
				<u-cell icon="chat" style="margin: 10rpx 0;" @click="setFeedBack" :isLink="true" title="意见满意度留言"></u-cell>
				<u-cell @click="toCommont()" icon="bell" style="margin: 10rpx 0;" :isLink="true" title="通知"></u-cell>
				<u-cell @click="rmb" icon="rmb" style="margin: 10rpx 0;" :isLink="true" title="充值"></u-cell>
				<u-cell @click="collectBtn" icon="star" style="margin: 10rpx 0;" :isLink="true" title="收藏"></u-cell>
				<u-cell @click="jump" icon="trash" style="margin: 10rpx 0;" :isLink="true" :title="login?'退出登录':'前往登录'"></u-cell>
			</u-cell-group>
		</view>

		
		
		
		<u-popup :show="showCars" mode="center" :round="10" :closeable="true" @close="showCars=false">
			<view style="width: 500rpx;">
				<u-cell-group>
					<u-cell v-for="(item,index) in allUserInfo.userItem" :key="index" icon="car-fill" :title="item.carNo" value="删除" @click="delCar(item)"></u-cell>
					<u-cell @click="addCar" title="新增" :titleStyle="{color:'#3f80de'}"></u-cell>
				</u-cell-group>
			</view>
		</u-popup>
		
		<u-modal :show="showModel" title="添加车牌" @confirm="subCarNo" >
			<view class="slot-content" @click="showKey=true">
				<u--input :readonly="true"
				    placeholder="请输入车牌"
				    border="surround"
				    v-model="carNo" 
				  ></u--input>
			</view>
		</u-modal>
		<u-modal :show="showRmb"  title="充值" confirm-text="支付" :showCancelButton="true" @confirm="bianqiang" @cancel="showRmb=false;">
			<view class="slot-content">
				<u-tabs :list="tagList" @click="sectionChange" scrollable="false"></u-tabs>
				
				<view v-if="curr==0">
					<view class="padding-2 margin-t">
						<view class="text-blue text-middle">充值</view>
						<view class="v-flex v-r-left v-c-center">金额：
							<view class="padding-10"><u--input
								placeholder="请输入充值金额"
								type="number"
								v-model="balance"
								
							  ></u--input></view>
						  </view>
						<view class="padding-10">当前余额：{{allUserInfo.vipInfo[0].balance||0}}<text class="text-gray text-light"></text></view>
					</view>
				</view>
				<view v-if="curr==1">
					<view class="padding-2 margin-t">
						<view class="text-blue text-middle">包月</view>
						<view class="v-flex v-r-left v-c-center">金额：
							<view class="padding-10">100元</view>
						  </view>
					</view>
				</view>
				<view v-if="curr==2">
					<view class="padding-2 margin-t">
						<view class="text-blue text-middle">包年</view>
						<view class="v-flex v-r-left v-c-center">金额：
							<view class="padding-10">1000元</view>
						  </view>
					</view>
				</view>
				<view class="padding-2 margin-t">
					<view class="text-blue text-middle">支付方式</view>
					<view>
						<view style="border-radius: 20rpx;" class=" margin-l padding-10 text-middle v-flex v-r-left v-c-center"><u-icon name="weixin-fill" size="50" color="#53c21d"></u-icon><text class="margin-l">微信支付</text></view>
					</view>
					
				</view>
			</view>
		</u-modal>
		<u-modal :show="showCollect" mode="center" @confirm="showCollect=false" confirm-text="关闭">
		
			<view class="slot-content padding-10">
				
				<view v-for="(item,index) in collect" :key="index" class="view-shadow v-flex v-r-between v-c-center padding-20 margin-b" style="width: 600rpx;height: 50rpx;border-radius: 30rpx;">
					<view class="text-ellipsis" style="width: 70%;">{{item.parkName}}</view>
					<view style="width: 15%;" @tap="toItem(item)">详情</view>
					<view style="width: 15%;color: red;" @tap="delCo(item)">删除</view>
				</view>
				<view v-if="collect.length == 0" class="v-flex v-r-between">无收藏</view>
			</view>
		</u-modal>
		<u-keyboard @confirm="showKey = false" @cancel="showKey=false" @change="valChange" @backspace="backspace" ref="uKeyboard" mode="car" :show="showKey"></u-keyboard>
	</view>
</template>

<script>
	import appRequest from "@/common/appRequestUrl.js"
	export default {
		data() {
			return {
				showKey:false,
				carNo:"",
				content:"测试",
				showModel:false,
				title:"操",
				showCars:false,
				feed:{
					title:"",
					content:""
				},
				userInfo:{
					nickName:"",
					carNo:""
				},
				login:false,
				allUserInfo:{},
				showCollect:false,
				collect:[],
				showRmb:false,
				balance:0,
				list:["充值","包月","包年"],
				curr:0,
				tagList:[{name:"充值"},{name:"包月"},{name:"包年"}],
				vipInfo:0
			}
		},
		onShow(){
			let info = appRequest.getUserInfo();
			if(!info.uid){
				uni.clearStorage()
				this.userInfo.nickName = "游客"
				this.userInfo.carNo = "登录后查看"
			}else{
				this.userInfo = info;
				this.login = true;
				this.getAllInfo();
				this.getCollect();
			}
		},
		methods: {
			rmb(){
				if(!this.userInfo.uid){
					this.$api.msg("请登录")
				}else{
					this.showRmb = true;
				}
				
			},
			collectBtn(){
				if(!this.userInfo.uid){
					this.$api.msg("请登录")
				}else{
					this.showCollect = true;
				}
			},
			
			toLogin(){
				if(!this.userInfo.uid){
					uni.clearStorage();
					uni.navigateTo({
						url:"/pages/simple/login"
					})
				}
			},
			getPic(pic){
				if(!pic || pic.length < 20){
					return "/static/coolc/park.png";
				}
				if(/^http/.test(pic)){
					return pic
				}
				if(pic.length >25){
					return appRequest.urlMap.getPic+pic
				}
				return pic;
			},
			sectionChange(item){
				let index = item.index;
				this.curr = index;
			},
			bianqiang(){
				
				let _this = this
				let info = this.allUserInfo.vipInfo[0];
				
				if(this.curr == 0){
					let reg = /^([1-9]\d*|0)(\.\d{1,2})?$/;
					if(this.balance<=0 || !reg.test(this.balance)){
					 uni.showToast({
							title:"金额异常",
							icon:"none"
						})
						
						return;
					}
					let vipInfo = this.allUserInfo.vipInfo[0].balance;
					vipInfo =  vipInfo + Number(this.balance);
					this.allUserInfo.vipInfo[0].balance =vipInfo;
				}else if(this.curr==1){
					
					info.vipType = 1;
					info.freeTime = 9999;
				}else if(this.curr==2){
					info.vipType = 2;
					info.freeTime = 9999;
				}
				if((this.vipInfo == 1 ||this.vipInfo == 2) && info.freeTime == 9999){
					uni.showToast({
						title:"已经是会员了，请到期后续费",
						icon:"none"
					})
					
					return;
				}
				appRequest.request({
					method: "POST",
					url: appRequest.urlMap.editNmCarUserVip,
					data:info,
					success: function(res) {
						_this.getAllInfo();
						_this.showRmb = false;
						uni.showToast({
							title:"充值完成",
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
			delCo(item){
				let _this = this
				item.validFlag=0;
				appRequest.request({
					method: "POST",
					url: appRequest.urlMap.editNmCarCollect,
					data:item,
					success: function(res) {
						_this.getCollect();
					},
					fail: function(res) {
						
						uni.showToast({
							title:"网络异常",
							icon:"none"
						})
						
					}
				})
			},
			toItem(item){
				uni.navigateTo({
					url:"/pages/site/parkItem?id="+item.parkFk
				})
			},
			getCollect(){
				let _this = this;
				appRequest.request({
					method: "POST",
					url: appRequest.urlMap.findNmCarCollectList,
					data:{
						creater:this.userInfo.uid,
						validFlag:1
					},
					success: function(res) {
						_this.collect = res.data.data;
					},
					fail: function(res) {
						
						uni.showToast({
							title:"网络异常",
							icon:"none"
						})
						
					}
				})
			},
			toCommont(){
				if(!this.userInfo.uid){
					this.$api.msg("请登录")
				}else{
					uni.navigateTo({
						url:"/pages/user/push"
					})
				}
				
			},
			cancleCollect(){
				
			},
			valChange(val) {
				// 将每次按键的值拼接到value变量中，注意+=写法
				this.carNo += val;
				console.log(this.carNo);
			},
			// 退格键被点击
			backspace() {
				// 删除value的最后一个字符
				if(this.carNo.length) {
					this.carNo = this.carNo.substr(0, this.carNo.length - 1);
				}					
				console.log(this.carNo);
			},
			subCarNo(){
				if(!uni.$u.test.carNo(this.carNo)){
					uni.showToast({
						title:"车牌号格式错误",
						icon:"none"
					})
				}else{
					let _this = this;
					appRequest.request({
						method: "POST",
						url: appRequest.urlMap.addNmCarUserItem,
						data:{
							userFk:this.userInfo.uid,
							carNo:this.carNo
						},
						success: function(res) {
							uni.showToast({
								title:"添加成功",
								icon:"none"
							})
							_this.getAllInfo();
							_this.showModel = false;
							_this.showKey = false;
						},
						fail: function(res) {
							
							uni.showToast({
								title:"网络异常",
								icon:"none"
							})
							
						}
					})
				}
				
				
			},
			addCar(){
				this.carNo = "";
				this.showModel = true;
			},
			delCar(item){
				let _this = this;
				uni.showModal({
					title:"提醒",
					content:"确认删除？",
					success:function(res){
						if(res.confirm){
							item.validFlag = 0;
							appRequest.request({
								method: "POST",
								url: appRequest.urlMap.editNmCarUserItem,
								data:item,
								success: function(res) {
									uni.showToast({
										title:"删除成功",
										icon:"none"
									})
									_this.getAllInfo();
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
			setFeedBack(){
				if(!this.userInfo.uid){
					this.$api.msg("请登录")
				}else{
					uni.navigateTo({
						url: "/pages/user/feed/feed"
					});
				}
				
			},
			navi(url) {
				uni.navigateTo({
					url: url
				});
			},jump(){
				if(this.login){
					this.login = false;
					uni.clearStorage();
					uni.switchTab({
						url:"/pages/site/index"
					})
				}else{
					uni.clearStorage();
					uni.navigateTo({
						url:"/pages/simple/login"
					})
				}
			},info(){
				uni.showModal({
					title:"应用信息",
					content:"智能停车场App，版本0.808",
					showCancel:false
				})
			},showCarInfo(){
				if(this.login){
					
					this.showCars = true;
					
				}else{
					uni.showToast({
						title:"请登录后操作",
						icon:"none"
					})
				}
			},showUserInfo(){
				if(this.login){
					uni.navigateTo({
						url:"/pages/user/userInfo"
					})
				}else{
					uni.showToast({
						title:"请登录后操作",
						icon:"none"
					})
				}
				
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
						_this.vipInfo = _this.checkVip();
					},
					fail: function(res) {
						
						uni.showToast({
							title:"网络异常",
							icon:"none"
						})
						
					}
				})
				
			},checkVip(){
				if(this.userInfo.nickName == "游客"){
					this.vipInfo = -1;
				}
				if(this.allUserInfo.vipInfo){
					let info = this.allUserInfo.vipInfo[0];
					if(info.vipType == 1 || info.vipType == 2){
						if(new Date().getTime() < new Date(info.vipEnd).getTime()){
							return info.vipType;
						}
					}
				}
				return 0;
			}
		}
	}
</script>

<style lang="scss">
	.warp_user {
		width: 750rpx;
		padding: 30rpx;
		background: linear-gradient(180deg, rgba(61, 126, 255, 1) 0%, rgba(61, 126, 255, 1) 30%, rgba(255, 255, 255, 1) 90%);
	}
	
	.top_user_box {
		width: 100%;
		@include flex;
		justify-content: flex-start;
		align-items: center;
		padding-bottom: 30rpx;
		
		image {
			width: 100rpx;
			height: 100rpx;
			border-radius: 100rpx;
			border: 6rpx solid #fff;
		}
		.user_info {
			padding: 24rpx;
			
			.username {
				font-size: 34rpx;
				color: #fff;
				padding-bottom: 10rpx;
			}
			.mobile {
				font-size: 26rpx;
				color: #fff;
			}
		}
	}
	.icon_boxs {
		background: #fff;
		border-radius: 20rpx;
		padding: 60rpx 0 20rpx 0;
		@include flex;
		justify-content: space-between;
		align-items: center;
		
		.i {
			width: 50%;
			text-align: center;
			
			.icon_box {
				width: 86rpx;
				height: 86rpx;
				line-height: 86rpx;
				border-radius: 86rpx;
				background: #9ab6ff;
				@include flex;
				justify-content: center;
				align-items: center;
				margin: 0 auto;
			}
			
			.n {
				padding-top: 20rpx;
				font-size: 24rpx;
				color: #333;
			}
		}
	}
	
	/* 遮罩层 */
		.modal {
			position: fixed;
			top: 0;
			left: 0;
			width: 100%;
			height: 100%;
			background-color: rgba(0, 0, 0, 0.5);
			display: flex;
			justify-content: center;
			align-items: center;
			font-size: 33rpx;
		}
		/* 窗口 */
		.modal-content {
			background-color: white;
			/* padding: 20px; */
			width: 600rpx;
			height: 800rpx;
			border-radius: 8rpx;
			position: relative;
			//modal-content下的第一个view
			view:first-child{
				padding:30rpx;
				font-size:33rpx;
				font-weight:bold;
			}
			//modal-content下的第二个view
			view:nth-child(2){
				padding:40rpx;
				font-size:33rpx;
				color:red
			}
		}
		/* 按钮 */
		.modal-buttons {
			width: 100%;
			display: flex;
			bottom: 0;
			position: absolute;
			font-size:33rpx;
		}
		.modal-buttons button {
			width: 100%;
			border: none;
		}
		.modal-buttons button:first-child {
			background-color: #74bfe7;
			color: #fff;
			border-radius: 0;
		}
		.modal-buttons button:last-child {
			width: 100%;
			border: 2rpx solid #74bfe7;
			border-radius: 0px;
			background-color: #fff;
			color: #74bfe7;
		}
	
</style>
