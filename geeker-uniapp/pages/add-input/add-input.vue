<template>
	<view>
		<!-- 自定义导航 -->
		<!-- #ifdef MP -->
		<uni-nav-bar :border="false">
			<view class="flex align-center justify-center w-100"
			@click="changeIsopen">
				{{isopenText}}<text class="iconfont icon-shezhi"></text>
			</view>
		</uni-nav-bar>
		<!-- #endif -->
		<!-- #ifndef MP -->
		<uni-nav-bar left-icon="back" statusBar :border="false" @click-left="goBack">
			<view class="flex align-center justify-center w-100"
			@click="changeIsopen">
				{{isopenText}}<text class="iconfont icon-shezhi"></text>
			</view>
		</uni-nav-bar>
		<!-- #endif -->
		<!-- 文本域 -->
		<textarea v-model="content" placeholder="说一句话吧" class="uni-textarea px-2"/>
		
		
		<!-- 选中的分类 -->
		<view class="font-md px-2 py-1 flex">
			<view class="border px-3 py main-color main-border-color flex align-center justify-center" style="border-radius: 50rpx;">
				<text class="iconfont icon-huati mr-1"></text>
				{{post_class_name ? "所属分类："+post_class_name : "请选择分类"}}
			</view>
		</view>
		
		<!-- 选中的标签 -->
		<view class="font-md px-2 py-1 flex">
			<view class="border px-3 py main-color main-border-color flex align-center justify-center" style="border-radius: 50rpx;">
				<text class="iconfont icon-huati mr-1"></text>
				{{tags ? "所属技术栈："+ tags : "请选择技术栈"}}
			</view>
		</view>
		
		<!-- 选中的话题 -->
		<view class="font-md px-2 py-1 flex">
			<view class="border px-3 py main-color main-border-color flex align-center justify-center" style="border-radius: 50rpx;">
				<text class="iconfont icon-huati mr-1"></text>
				{{topic.title ? "所属话题："+topic.title : "请选择话题"}}
			</view>
		</view>
		
		<!-- 选择技术栈 -->
		<view class="uni-list" v-if="showChooseTags">
					<checkbox-group @change="checkboxChange">
						<label class="uni-list-cell uni-list-cell-pd" v-for="item in items" :key="item.value">
							<view>
								<checkbox :value="item.value" :checked="item.checked" />
							</view>
							<view>{{item.name}}</view>
						</label>
					</checkbox-group>
		</view>
		
		<!-- 多图上传 -->
		<upload-image :show="show" ref="uploadImage" :list="imageList" @change="changeImage"></upload-image>
		
		<!-- 底部操作条 -->
		<view class="fixed-bottom bg-white flex align-center" style="height: 85rpx;">
			<!-- 选择分类 -->
			<picker mode="selector" :range="range" 
			@change="choosePostClass">
				<view class="iconfont icon-caidan footer-btn animate__animated"
				hover-class="animate__jello"></view>
			</picker>
			<!-- 选择标签 -->
			<view @click="choosePostTag">
				<view class="iconfont icon-zengjia1 footer-btn animate__animated"
				hover-class="animate__jello"></view>
			</view>
			
			<view class="iconfont icon-huati footer-btn animate__animated"
			hover-class="animate__jello" @click="chooseTopic"></view>
			<view class="iconfont icon-tupian footer-btn animate__animated"
			hover-class="animate__jello" @click="iconClickEvent('uploadImage')"></view>
			
			<view class="bg-main text-white ml-auto flex justify-center align-center rounded mr-2 animated" hover-class="jello" style="width: 140rpx;height: 60rpx;" @click="submit">发送</view>
		</view>
		
	</view>
</template>

<script>
	const isOpenArray = ['仅自己可见','所有人可见'];
	import uniNavBar from '@/components/uni-ui/uni-nav-bar/uni-nav-bar.vue';
	import uploadImage from '@/components/common/upload-image.vue';
	export default {
		components: {
			uniNavBar,
			uploadImage
		},
		data() {
			return {
				showChooseTags:false,
				tags:"",
				content:"",
				imageList:[],
				// 是否已经弹出提示框
				showBack:false,
				isopen:1,
				topic:{
					id:0,
					title:""
				},
				post_class_list:[],
				post_class_index:-1,
				list:["Java", "Python", "Go"],
				list_index:-1,
				items: [{
							value: 'Java',
							name: 'Java'
						},
						{
							value: 'Python',
							name: 'Python'
						},
						{
							value: 'Go',
							name: 'Go'
						},
						{
							value: 'C++',
							name: 'C++'
						},
						{
							value: '算法',
							name: '算法'
						},
						{
							value: '前端',
							name: '前端'
						},
						{
							value:"后端",
							name:"后端"
						}
					],
				
			}
				
		},
		computed: {
			show() {
				return this.imageList.length >= 0 
			},
			isopenText(){
				return isOpenArray[this.isopen]
			},
			// 文章分类可选项
			range(){
				return this.post_class_list.map(item=>{
					return item.classname
				})
			},
			post_class_id(){
				if(this.post_class_index !== -1){
					return this.post_class_list[this.post_class_index].id
				}
			},
			post_class_name(){
				if(this.post_class_index !== -1){
					return this.post_class_list[this.post_class_index].classname
				}
			},
			post_tag(){
				if(this.list_index !== -1){
					return this.list[this.list_index]
				}
			},
			imgListIds(){
				return this.imageList.map(item=>{
					return {
						id:item.id
					}
				})
			}
		},
		// 监听返回
		onBackPress() {
			if ((this.content !== '' || this.imageList.length > 0) && !this.showBack ) {
				uni.showModal({
					content: '是否要保存为草稿？',
					showCancel: true,
					cancelText: '不保存',
					confirmText: '保存',
					success: res => {
						// 点击确认
						if (res.confirm) {
							this.store()
						} else { // 点击取消，清除缓存
							uni.removeStorage({
								key:"add-input"
							})
						}
						// 手动执行返回
						uni.navigateBack({ delta: 1 });
					},
				});
				this.showBack = true
				return true
			}
		},
		// 页面加载时
		onLoad() {
			uni.getStorage({
				key:"add-input",
				success:(res)=>{
					if (res.data) {
						let result = JSON.parse(res.data)
						this.content = result.content
						this.imageList = result.imageList
					}
				}
			})
			// 监听选择话题事件
			uni.$on('chooseTopic',(e)=>{
				this.topic.id = e.id
				this.topic.title = e.title
			})
			// 获取所有分类
			this.getPostClass()
		},
		beforeDestroy() {
			uni.$off('chooseTopic',(e)=>{})
		},
		methods: {
			
			checkboxChange: function (e) { 
				// 选择技术栈  以字符串存储在this.tags中 以':'分隔 
				// 必须至少选择一个技术栈
				var items = this.items,
				values = e.detail.value;
				var str = "";
				for (var i = 0, lenI = items.length; i < lenI; ++i) {
					const item = items[i]
					if(values.includes(item.value)){
						this.$set(item,'checked',true)
						str = str + item.value + ":";
					}else{
						this.$set(item,'checked',false)
					}
				}
				if (str!==""){
					this.tags = str.slice(0,-1);
				}
			},
						
			// 发布
			submit(){
				if(this.topic.id == 0){
					return uni.showToast({
						title: '请选择话题',
						icon: 'none'
					});
				}
				if(this.post_class_id == 0){
					return uni.showToast({
						title: '请选择分类',
						icon: 'none'
					});
				}
				uni.showLoading({
					title: '发布中...',
					mask: false
				});
				console.log(this.imageList);
				console.log(this.tags);
				var item = {
					titlepic : this.imageList.length === 0 ? "" : this.imageList[0].url,
					title:this.content,
					isopen:this.isopen,
					language:this.tags,
					topic_id:this.topic.id,
					post_class_id:this.post_class_id,
					share_id:0,
					content:this.content,
					type:0,
					sharenum:0,
					path: this.$store.state.user.path ? this.$store.state.user.path : "未知",
					user_id:this.$store.state.user.id,
				}
				console.log(item);
				this.$H.post('/addpost', item).then(res=>{
					uni.hideLoading()
					uni.$emit('updateIndex')
					uni.showToast({
						title: '发布成功',
						icon:"none"
					});
					console.log(res);
					this.showBack = true
					//update是可以自定义的事件名,触发全局的自定义事件，附加参数都会传给监听器回调函数。
					
					uni.navigateBack({
						delta: 1
					});
					uni.$emit('update',{msg:'页面更新'})
				}).catch(err=>{
					uni.hideLoading()
				})
			},
			// 获取所有文章分类
			getPostClass(){
				this.$H.get('/pc').then(res=>{
					this.post_class_list = res.data;
					this.post_class_list.splice(1,1); // 删除推荐分类
				})
			},
			// 选择文章分类
			choosePostClass(e){
				this.post_class_index = e.detail.value
			},
			// 选择文章标签
			choosePostTag(){
				this.showChooseTags = !this.showChooseTags;
			},
			// 选择话题
			chooseTopic(){
				uni.navigateTo({
					url: '../topic-nav/topic-nav?choose=true',
				});
			},
			// 切换可见性
			changeIsopen(){
				uni.showActionSheet({
				    itemList: isOpenArray,
				    success: (res)=> {
				        this.isopen = res.tapIndex
				    }
				});
			},
			// 底部图片点击事件
			iconClickEvent(e){
				switch (e){
					case 'uploadImage':
					this.$refs.uploadImage.chooseImage()
						break;
				}
			},
			// 返回上一步
			goBack(){
				uni.navigateBack({ delta: 1 });
			},
			// 选中图片
			changeImage(e){
				this.imageList = e
				console.log(e);
				console.log(this.imageList);
			},
			// 保存操作
			store(){
				// 保存为本地存储
				let obj = {
					content:this.content,
					imageList:this.imageList
				}
				uni.setStorage({
					key:'add-input',
					data:JSON.stringify(obj)
				})
			}
		}
	}
</script>

<style>
.footer-btn{
	width: 86rpx;
	height: 86rpx;
	display: flex;
	justify-content: center;
	align-content: center;
	font-size: 50rpx;
}
</style>
