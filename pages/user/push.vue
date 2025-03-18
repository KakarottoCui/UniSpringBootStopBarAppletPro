<template>
	<view>
		<view class="padding-20">
			<u-cell-group>
				<u-cell v-for="(item,index) in pushList" :key="index" icon="email-fill" :title="item.title" :label="item.content" value="详情" @click="toItem(item)"></u-cell>
			</u-cell-group>
			
			<view v-if="pushList.length == 0" class="v-flex v-r-between text-middle">暂无消息通知</view>
			
		</view>
		
		<u-modal :show="showComment"  title="消息" confirm-text="关闭" @confirm="showComment=false">
			<view class="slot-content" style="width: 100%;">
				<view class="padding-10">
					<view class="v-flex v-flex-column margin-b">
						<view class="text-middle text-bold">
							{{item.title}}
						</view>
						<view class="margin-t margin-b">
							{{item.content}}
						</view>
						<view class="v-flex v-r-left text-gray text-18">
							{{item.createTime}}
						</view>
					</view>
				</view> 
			</view>
		</u-modal>
		
	</view>
</template>

<script>
	import appRequest from "@/common/appRequestUrl.js"
	export default {
		data() {
			return {
				pushList:[],
				userInfo:"",
				item:{},
				showComment:false
			}
		},
		onShow(){
			let info = appRequest.getUserInfo();
			if(!info.uid){
				uni.clearStorage()
			}else{
				this.userInfo = info;
				this.getData();
			}
		},
		methods: {
			toItem(item){
				this.item = item;
				this.showComment = true;
			},
			getData(){
				let _this = this;
				appRequest.request({
					method: "POST",
					url: appRequest.urlMap.findNmCarParkPushList,
					data:{
						validFlag:1,
						creater:this.userInfo.uid
					},
					success: function(res) {
						_this.pushList = res.data.data;
						
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

</style>
