<template>
	<view>
		<view class="warp">
			<view class="name">个人信息</view>
			<u-form labelPosition="left" label-width="70" :model="form" ref="uForm">
				<u-form-item prop="nickname" borderBottom label="用户名">
					<u-input v-model="form.nickName" placeholder="请输入用户"/>
				</u-form-item>
				<u-form-item prop="mobile" borderBottom label="手机号">
					<u-input v-model="form.phone" placeholder="请输入手机号"/>
				</u-form-item>
				<u-form-item prop="mobile" borderBottom label="性别">
					<u-radio-group v-model="form.sex" >
						<u-radio v-for="(item, index) in [{name:'1',label:'男'},{name:'0',label:'女'}]"
						  :key="index"
						  :label="item.label"
						  :name="item.name" shape="circle">
						</u-radio>
					</u-radio-group>
				</u-form-item>
				<u-form-item prop="mobile" borderBottom label="头像">
					<view class="u-flex u-direction-row u-p-l-25 u-border-bottom u-p-b-25">
				          <u-image border-radius="15" @tap="ChooseImage()" class="u-image-shadow" height="180rpx"  
					width="180rpx" :src="getPic(form.pic)" mode="aspectFill">
					           <view slot="error" style="font-size: 24rpx;" class="u-text-center">
						             <u-icon size="30" name="plus"></u-icon><br />选择图片
						            </view>
					           <view slot="loading" style="font-size: 24rpx;" class="u-text-center">
						             <u-icon size="30" name="reload"></u-icon><br />加载中
						            </view>
					          </u-image>      
					     </view>
				</u-form-item>
				<u-button type="primary" @click="sub">提交</u-button>
			</u-form>
			<view class="margin-t padding-t">
				<u-button type="" @click="changePwd">修改密码</u-button>
			</view>
			
		</view>
		
		<u-modal :show="showPwd"  title="修改密码" confirm-text="提交" @confirm="sub2">
			<view class="slot-content" style="width: 100%;">
				<view class="padding-10">
				<u--input
					placeholder="输入原密码"
					type="password"
					v-model="pwd.pass"
				  ></u--input>
				  </view>
				  <view class="padding-10">
				  <u--input
				  	placeholder="输入新密码"
				  	type="password"
				  	v-model="pwd.passN"
				    ></u--input>
					</view>
					<view class="padding-10">
					<u--input
						placeholder="重复新密码"
						type="text"
						v-model="pwd.passCopy"
					  ></u--input>
				  </view>
			</view>
		</u-modal>
		
	</view>
</template>

<script>
	
	import appRequest from "@/common/appRequestUrl.js"
	import base64 from "@/common/base64.js"
	import ygyUploadImg from '../../components/ygy-upload-img/ygy-upload-img.vue'		
	export default {
		components: {
			ygyUploadImg
		},
		data() {
			return {
				showPwd:false,
				pwd:{
					pass:"",
					passN:"",
					passCopy:""
				},
				userInfo:{
					nickname:"",
					mobile:""
				},
				login:false,
				array: [],
				index:0,
				parkInfo:[],
				selItem:{},
				form:{
					pic:"",
					mobile:"",
					nickname:"",
					gender:"1",
					plateNumber:"",
					sex:"1",
					orgId:0,
					orgName:"",
					parkManageId:0,
					parkManageName:0,
					pass:"",
					pass2:""
				},
				show:true,
				allUserInfo:{}
			};
		},
		onShow(){
			let info = appRequest.getUserInfo();
			if(!info){
				uni.clearStorage()
				this.userInfo.nickname = "游客"
				this.userInfo.mobile = "登录后查看"
				uni.navigateTo({
					url:"/pages/user/index"
				})
			}else{
				let _this = this;
				this.userInfo = info;
				this.login = true;
				if(!this.form.uid){
					this.form = info;
					this.form.sex = this.form.sex+"";
					this.$forceUpdate();
				}
				this.form["pass3"] = "";
				this.form["pass2"] = "";
				this.form["pass1"] = "";
				this.getAllInfo()
			}
		},
		onLoad(){
			// this.getParkInfo();
			
		},
		methods:{
			changePwd(){
				if(this.userInfo.openId){
					this.$api.msg("微信登录用户免密，无需修改")
				}else{
					this.showPwd = true;
				}
			},
			getPic(pic){
				if(pic.length < 20){
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
			getImgList(arr) {
				this.form.pic = arr[0];
				console.log(arr[0]);
			},
			radioGroupChange(e){
				this.form.gender = e;
			},
			sub2(){
				let _this = this;
				let base64en = new base64();
				let enPass = base64en.encode(this.pwd.pass);
				let user = this.allUserInfo;
				if(enPass == this.allUserInfo.password){
					console.log(JSON.stringify(this.pwd))
					if(this.pwd.passN == this.pwd.passCopy && this.pwd.passN.length >= 6 ){
						let item = {
							uid:user.uid,
							password:base64en.encode(this.pwd.passN )
						}
						
						appRequest.request({
							method: "POST",
							data:item,
							url: appRequest.urlMap.editSysUser,
							success: function(res) {
								if(res.data.code == 200){
									uni.clearStorage();
									uni.showModal({
										title:"信息",
										content:"修改成功,变更信息后需重新登录，将前往登录页面",
										showCancel:false,
										success:function(e){
											uni.redirectTo({
												url:"/pages/simple/login"
											})
										}
									})
								}else{
									uni.showModal({
										title:"信息",
										content:"修改失败，退回个人中心",
										showCancel:false,
										success:function(e){
											uni.switchTab({
												url:"/pages/user/index"
											})
										}
									})
								}
								
							},
							fail: function(res) {
								_this.$api.msg("请求异常");
								_this.userInfo.password = "";
							}
						})
					}else{
						this.$api.msg("新密码两次不一致或长度不足6位")
						return
					}
					
				}else{
						this.$api.msg("原密码错误")
						return
					}
				
				
				
			},
			sub(){
				let _this = this;
				let base64en = new base64();
				if(_this.form.nickname =="" || _this.form.phone == ""){
					uni.showToast({
						title:"请填写完整",
						icon:"none"
					})
					
					return;
				}
				
				if( !_this.$u.test.mobile(_this.form.phone) ){
					uni.showToast({
						title:"请填写正确的手机号格式",
						icon:"none"
					})
					return;
				}
				appRequest.request({
					method: "POST",
					data: _this.form,
					url: appRequest.urlMap.changeUserInfo,
					success: function(res) {
						if(res.data.code == 200){
							uni.clearStorage();
							uni.showModal({
								title:"信息",
								content:"修改成功,变更信息后需重新登录，将前往登录页面",
								showCancel:false,
								success:function(e){
									uni.redirectTo({
										url:"/pages/simple/login"
									})
								}
							})
						}else{
							uni.showModal({
								title:"信息",
								content:"修改失败，退回个人中心",
								showCancel:false,
								success:function(e){
									uni.switchTab({
										url:"/pages/user/index"
									})
								}
							})
						}
					},
					fail: function(res) {
						_this.$api.msg("请求异常");
					}
				})
				
			},
			changeCar(e){
				this.form.plateNumber = e;
			},
			getAllInfo(){
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
						_this.allUserInfo = res.data.data;
					},
					fail: function(res) {
						
						uni.showToast({
							title:"网络异常",
							icon:"none"
						})
						
					}
				})
				
			},ChooseImage() {
				let _this = this;
				uni.chooseImage({
					count: 1,
					sizeType: ["compressed"],
					success: function(res) {
						let src = res.tempFiles[0].path;
						uni.compressImage({
							src: src,
							width: 450,
							quality: 70,
							success: function(path) {
								let filePath = path.tempFilePath;
								wx.getFileSystemManager().readFile({
									filePath: filePath,
									encoding: 'base64',
									fail: function(errMsg) {
										console.log(errMsg);
									},
									success: function(res) {
										_this.form.pic =
											"data:image/jpeg;base64," + res.data;
									}
								});
							}
						});
					},
					fail: function(errMsg) {
						console.log(errMsg);
					}
				});
			}
		},onReady() {
			//this.$refs.uForm.setRules(this.rules);
		}
	}
</script>

<style lang="scss">
	.warp {
		width: 750rpx;
		padding: 160rpx 70rpx 30rpx 70rpx;
		background: #fff;
		
		.name {
			width: 100%;
			text-align: center;
			font-size: 38rpx;
			color: #3366FF;
			padding-bottom: 80rpx;
		}
		
		.u-form-item {
			padding-bottom: 50rpx;
		}
		
		.buttonBox {
			width: 100%;
			padding-top: 30rpx;
			
			.u-button {
				width: 100%;
			}
		}
	}
</style>
