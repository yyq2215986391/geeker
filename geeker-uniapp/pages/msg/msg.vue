<template>
	<view v-if="update">
		<template v-if="list.length > 0">
		<block v-for="(item, index) in list" :key="index"><!-- 遍历消息 -->
			<msg-list :item="item" :index="index" :key='value'></msg-list>
		</block>
		</template>
		<template v-else>
			<Nothing></Nothing>
		</template>
		
		<!-- 弹出层 -->
		<uni-popup ref="popup" type="top">
			<view class="flex align-center justify-center font-md border-bottom py-2" hover-class="bg-light" @click="popupEvent('friend')">
				<text class="iconfont icon-sousuo mr-2"></text> 添加好友
			</view>
			<view class="flex align-center justify-center font-md py-2" hover-class="bg-light"  @click="popupEvent('clear')">
				<text class="iconfont icon-shanchu mr-2"></text> 清除列表
			</view>
		</uni-popup>
	</view>
</template>

<script>
	const demo = [
					{
						user_id:13,
						avatar:"../../static/bgimg/4.jpg", //头像
						username:"昵称",
						update_time:1644891000606,
						data:"内容内容内容内容内容内容内容内容内容内容内容内容内容", //内容
						noread:20, //未读数
					},{
						user_id:16,
						avatar:"../../static/bgimg/4.jpg", //头像
						username:"昵称",
						update_time:1644891000606,
						data:"内容内容内容内容内容内容内容内容内容内容内容内容内容", //内容
						noread:20, //未读数
					},{
						user_id:19,
						avatar:"../../static/bgimg/4.jpg", //头像
						username:"昵称",
						update_time:1644891000606,
						data:"内容内容内容内容内容内容内容内容内容内容内容内容内容", //内容
						noread:20, //未读数
					},{
						user_id:22,
						avatar:"../../static/bgimg/4.jpg", //头像
						username:"昵称",
						update_time:1644891000606,
						data:"内容内容内容内容内容内容内容内容内容内容内容内容内容", //内容
						noread:20, //未读数
					}
				]
	
	import msgList from "@/components/msg/msg-list.vue"
	import Nothing from "@/components/common/Nothing.vue"
	import uniPopup from "@/components/uni-ui/uni-popup/uni-popup.vue"
	import { mapState } from 'vuex'
	export default {
		components:{
			msgList,
			Nothing,
			uniPopup
		},
		data() {
			return {
				list:[],
				update:true,
				update2:true,
				value:0
			}
		},
		onLoad(){
			this.$store.dispatch('getChatList').then(res=>{
				this.list = res
				this.update = false
				this.update = true
				console.log('getChatList', this.list)
			})
			
		},
		computed:{
		}
		,
		watch:{
			'$store.state.chatList'(){
				console.log("改变")
				/* this.chatList = this.$store.state.chatList
				this.value++;
				console.log(this.value)
				 
				 重新push
				 */
				let pages = getCurrentPages();
				if (pages.length == 1) {
					uni.reLaunch({
					    url: '../msg/msg'
					});
				}
			}
		}
		,
		onPullDownRefresh(){
			this.refresh();
		},
		// 监听原生导航栏按钮事件
		onNavigationBarButtonTap(e){
			switch (e.index){
				case 0: //left button
					break;
				case 1: //right button
					this.$refs.popup.open() //打开弹出层
					break;
			}
		},
		methods: {
			refresh(){
				setTimeout(()=>{
						uni.stopPullDownRefresh(); //停止刷新
					}, 2000
				)
				uni.reLaunch({
				    url: '../msg/msg'
				});
			},
			popupEvent(e){
				switch (e){
					case "friend":
					console.log("添加好友");
						break;
					case "clear":
					console.log("清除列表");
						break;
				}
				// 关闭弹出层
				this.$refs.popup.close()
			}
		}
	}
</script>

<style>

</style>
