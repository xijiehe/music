<template>
	<view class="list">
		<view class="fixbg" :style="{ 'background-image':`url(${playlist.coverImgUrl})`}"></view>
		<musichead title="Music" :icon="true" color="white"></musichead>
		<view class="container" v-show="!isLoading">
			<scroll-view scroll-y="true" >
				<view class="list-head">
					<view class="list-head-img">
						<image :src="playlist.coverImgUrl" mode=""></image>
						<text class="iconfont icon-yousanjiao">{{playlist.playCount|formatCount}}</text>
					</view>
					<view class="list-head-text">
						<view class="">{{playlist.name}}</view>
						<view class="">
							<image :src="playlist.creator.avatarUrl" mode=""></image>
							{{playlist.creator.nickname}}
						</view>
						<view class="">{{playlist.description}}</view>
					</view>
				</view>
				<!-- #ifdef MP-WEIXIN -->
				<button open-type="share" class="list-share">
					<text class="iconfont icon-fenxiang"></text>
					分享给好友
				</button>
				<!-- #endif -->
				<view class="list-music">
					<view class="list-music-head">
						<text class="iconfont icon-bofang1"></text>
						<text>播放全部</text>
						<text>(共{{playlist.trackCount}}首)</text>
					</view>
					<view class="list-music-item" @tap="handleToDetail(item.id)" v-for="(item,index) in playlist.tracks">
						<view class="list-music-top">{{index+1}}</view>
						<view class="list-music-song">
							<view class="">{{item.name}}</view>
							<view>
								<image  src="../../static/dujia.png" mode=""></image>
								<image v-if="privileges[index].maxbr===999000" src="../../static/wusunyinzhi.png" mode=""></image>
								{{item.ar[0].name}} - {{item.name}}
							</view>
						</view>
						
						<text class="iconfont icon-bofang"></text>
					</view>
				</view>
			</scroll-view>
		</view>
	</view>
</template>

<script>
	import "@/common/iconfont.css"
	import {list} from "../../common/api.js"
	import musichead from "../../components/musichead/musichead.vue"
	export default {
		data() {
			return {
				playlist:{
					coverImgUrl:"",
					creator:{
						avatarUrl:"",
					},
					trackCount:"",
					privileges:[],
				},
				isLoading:true
			}
		
		},
		methods: {
			handleToDetail(songId){
				uni.navigateTo({
					url:"/pages/detail/detail?songId="+songId
				})
			}
		},
		components:{
			musichead
		},
		onLoad(options) {
			uni.showLoading({
				title:'正在加载'
			})
			list(options.listId).then((res)=>{
				if(res.data.code==200){
					this.playlist=res.data.playlist
					this.privileges=res.data.privileges
					this.$store.commit("INIT_TOPLISTIDS",res.data.playlist.trackIds)
                    this.isLoading=false;
					uni.hideLoading()
				}
			})
			
		}
	}
</script>

<style scoped>
.list-head{
	display: flex;
	margin: 30rpx;
}
.list-head-img{
	width: 264rpx;
	height: 264rpx;
	border-radius: 30rpx;
	overflow: hidden;
	position: relative;
	margin-right: 42rpx;
}
.list-head-img image{
	width: 100%;
	height: 100%;
}
.list-head-img text{
	position: absolute;
	color: white;
	top: 8rpx;
	right: 8rpx;
	font-size: 26rpx;
}
.list-head-text{
	flex: 1;
	color:#f0f2f7
}
.list-head-text view:nth-child(1){
	color: white;
	font-size: 34rpx;
}
.list-head-text view:nth-child(2){
	display: flex;
	margin: 20rpx 0;
	align-items: center;
	font-size: 24rpx;
}
.list-head-text view:nth-child(2) image{
	width: 54rpx;
	height: 54rpx;
	border-radius: 50%;
	margin-right: 14rpx;
}
.list-head-text view:nth-child(3){
	line-height: 34rpx;
	font-size: 22rpx;
}
.list-share{
	width: 330rpx;
	height: 74rpx;
	margin: 0 auto;
	background: rgba(0, 0,0, .4);
	border-radius: 37rpx;
	color: white;
	text-align: center;
	line-height: 74rpx;
	font-size: 28rpx;
}
list-share text{
	margin-right: 16rpx;
}
.list-music{
	background-color: white;
	border-radius: 50rpx 50rpx 0 0;
	margin-top: 40rpx;
	overflow: hidden;
}
.list-music-head{
	height: 50rpx;
	margin: 30rpx 0 70rpx 22rpx;
}
.list-music-head text:nth-child(1){
	height: 50rpx;
	font-size: 50rpx;
}
.list-music-head text:nth-child(2){
	font-size: 30rpx;
	margin: 0 10rpx 0 26rpx;
}
.list-music-head text:nth-child(3){
	font-size: 26rpx;
	color: #b2b2b2;
}
.list-music-item{
	display: flex;
	margin: 0 32rpx 66rpx 46rpx;
	align-items: center;
	color: #959595;
}
.list-music-top{
	width: 58rpx;
	font-size: 30rpx;
	line-height: 30rpx;
}
.list-music-song{
	flex: 1;
}
.list-music-song view:nth-child(1){
	font-size: 28rpx;
	color: black;
	width: 70vw;
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;
}
.list-music-song view:nth-child(2){
	font-size: 20rpx;
	align-items: center;
	width: 70vw;
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;
	display: flex;
	align-items: center;
}

.list-music-song view:nth-child(2) image{
	width: 40rpx;
	height: 35rpx;
	margin-right: 10rpx;
}
.item-music-item text{
	font-size: 50rpx;
	color: #c7c7c7;
}
</style>
