<template>
	<view class="detail">
		<view class="fixbg" :style="{ 'background-image':  'url('+songDetail.al.picUrl+')'}"></view>
		<musichead :title="songDetail.name" :icon="true" color="white"></musichead>
		<view class="container" v-show="!isLoading">
			<scroll-view scroll-y="true">
				<view class="detail-play" @tap="handleToPlay">
					<image :src="songDetail.al.picUrl" :class="{'detail-play-run':isPlayRotate}" mode=""></image>
					<text class="iconfont" :class="iconPlay"></text>
					<view></view>
				</view>
				<view class="detail-lyric">
					<view class="detail-lyric-wrap" :style="{ transform:'translateY('+ -(lyricIndex-1)*82+'rpx)' }">
						<view class="detail-lyric-item" :class="{active:lyricIndex == index}" v-for="(item,index) in songLyric" :key>{{item.lyric}}</view>
					</view>
				</view>
				<view class="detail-like">
					<view class="detail-like-head">
						喜欢这首歌的人也听
					</view>
					<view class="detail-like-item" v-for="(item,index) in songSimi" :key="item.id" @tap="handleToSimi(item.id)">
						<view class="detail-like-img">
							<image :src="item.album.picUrl" mode=""></image>
						</view>
						<view class="detail-like-song">
							<view class="">{{item.name}}</view>
							<view class="">
								<image v-if="item.privilege>60&&item.privilege<70" src="../../static/dujia.png" mode=""></image>
								<image v-if="item.privilege.maxbr==999000" src="../../static/wusunyinzhi.png" mode=""></image>
								<text>{{item.album.artists[0].name}}-{{item.name}}</text>
							</view>
						</view>
						<text class="iconfont icon-bofang"></text>
					</view>
				</view>
				<view class="detail-comment">
					<view class="detail-comment-head">精彩评论</view>
					<view class="detail-comment-item" v-for="(item,index) in songComment" :key="item.commentId">
						<view class="detail-comment-img">
							<image :src="item.user.avatarUrl" mode=""></image>
						</view>
						<view class="detail-comment-content">
							<view class="detail-comment-title">
								<view class="detail-comment-name">
									<view class="">{{item.user.nickname}}</view>
									<view class="">{{item.time|formatTime}}</view>
								</view>
								<view class="detail-comment-like">
									{{item.likedCount|formatCount}}
									<text class="iconfont icon-good"></text>

								</view>
							</view>
							<view class="detail-comment-text">{{item.content}}</view>
						</view>
					</view>
				</view>
			</scroll-view>
		</view>
	</view>
</template>

<script>
	import "@/common/iconfont.css"
	import {
		list,
		songDetail,
		songComment,
		songSimi,
		songLyric,
		songUrl
		
	} from "../../common/api.js"
	import musichead from "../../components/musichead/musichead.vue"
	export default {
		data() {
			return {
				songDetail: {
					al: {
						picUrl:""
					}
				},
				songSimi:[],
				songComment:[],
				songLyric:[],
				lyricIndex:0,
				iconPlay:"icon-zanting",
				isPlayRotate:true,
				isLoading:true
			}
		},
		components: {
			musichead
		},
		onLoad(options) {
			// console.log(options.songId);
			this.getMusic(options.songId)
		},
		onUnload(){
			this.cancelLyricIndex()
			// #ifdef H5
			  this.bgAudioManager.destroy()
			// #endif	
		},
		onHide(){
			this.cancelLyricIndex()
			// #ifdef H5
			  this.bgAudioManager.destroy()
			// #endif	
		},
		methods: {
			getMusic(songId) {
				this.$store.commit('NEXT_ID',songId)
				uni.showLoading({
					title:"加载中..."
				})
				this.isLoading=true
				Promise.all([songDetail(songId),songSimi(songId),songComment(songId),songLyric(songId),songUrl(songId)]).then((res) => {
					console.log(res);
					if (res[0].data.code == 200) {
						this.songDetail = res[0].data.songs[0];
					}
					if(res[1].data.code == 200){
						this.songSimi = res[1].data.songs;
					}
					if(res[2].data.code == 200){
						this.songComment = res[2].data.hotComments;
					}
					if(res[3].data.code == 200){
						let lyric=res[3].data.lrc.lyric
						let re=/\[(.*)\](.*)/g
						var result=[]
						lyric.replace(re,($0,$1,$2)=>{
							result.push({"time":this.formatTimeToSec($1),"lyric":$2})
						})
						this.songLyric=result
					}
					if(res[4].data.code==200 ){
						// #ifdef MP-WEIXIN
						this.bgAudioManager = uni.getBackgroundAudioManager();
						this.bgAudioManager.title = this.songDetail.name;
						// #endif
						// #ifdef H5
						if(!this.bgAudioManager){
						this.bgAudioManager = uni.createInnerAudioContext();
						}

						this.iconPlay='icon-bofang'
						this.isPlayRotate=false
						// #endif

						this.bgAudioManager.src = res[4].data.data[0].url||"";
						this.listenLyricIndex()
						this.bgAudioManager.onPlay(()=>{
							this.iconPlay="icon-zanting";
							this.isPlayRotate=true
							this.listenLyricIndex()
						})
						this.bgAudioManager.onPause(()=>{
							this.iconPlay="icon-bofang";
							this.isPlayRotate=false
						})
						this.bgAudioManager.onEnded(()=>{
							this.getMusic(this.$store.state.nextId)
						})
					}
					this.isLoading=false;
					uni.hideLoading()
				})
			},
			formatTimeToSec(value){
				let arr=value.split(":")
				return (parseFloat(arr[0])*60 + parseFloat(arr[1])).toFixed(1);
			},
			handleToPlay(){
				if(this.bgAudioManager.paused){
					this.bgAudioManager.play();
				}else{
					this.bgAudioManager.pause()
					this.cancelLyricIndex()
				}
			},
			listenLyricIndex(){
				clearInterval(this.timer);
				this.timer=setInterval(()=>{
					if(this.bgAudioManager.currentTime>this.songLyric[this.songLyric.length-1].time){
						this.lyricIndex=this.songLyric.length-1
						return;
					}
					for(var i=0;i<this.songLyric.length;i++){
						if(this.bgAudioManager.currentTime>this.songLyric[i].time && this.bgAudioManager.currentTime<this.songLyric[i+1].time){
							this.lyricIndex=i;
						}
					}
					console.log(this.lyricIndex);
				},500)
				
			},
			cancelLyricIndex(){
				clearInterval(this.timer)
			},
			handleToSimi(songId){
				this.getMusic(songId)
			}
		}
	}
</script>

<style>
	.detail-play {
		width: 580rpx;
		height: 580rpx;
		margin: 214rpx auto 0 auto;
		position: relative;
	}

	.detail-play image {
		width: 500rpx;
		height: 500rpx;
		border-radius: 50%;
		position: absolute;
		left: 0;
		top: 0;
		right: 0;
		bottom: 0;
		margin: auto;
        animation: 10s linear move infinite;
		animation-play-state: paused;
	}
    .detail-play .detail-play-run{
		   animation-play-state: running;
	} 
	   @keyframes move{
		   from{
			   transform: rotate(0deg);
		   }
		   to{
			   transform: rotate(360deg);
		   }
	   }
	.detail-play text {
		width: 100rpx;
		height: 100rpx;
		font-size: 100rpx;
		color: white;
		position: absolute;
		left: 0;
		top: 0;
		right: 0;
		bottom: 0;
		margin: auto;
	}

	.detail-play view {}

	.detail-lyric {
		font-size: 32rpx;
		line-height: 82rpx;
		height: 246rpx;
		text-align: center;
		overflow: hidden;
	}

	.detail-lyric-wrap {
		transition: 0.5s ;
	}

	.detail-lyric-item {
		min-height: 82rpx;
	}

	.detail-lyric-item.active {
		color: white;
	}

	.detail-like {
		margin: 0 30rpx;
	}

	.detail-like-head {
		font-size: 36rpx;
		color: white;
		margin: 50rpx 0;
	}

	.detail-like-item {
		display: flex;
		align-items: center;
		margin-bottom: 28rpx;
	}

	.detail-like-img {
		width: 82rpx;
		height: 82rpx;
		border-radius: 20rpx;
		overflow: hidden;
		margin-right: 20rpx;
	}

	.detail-like-img image {
		width: 100%;
		height: 100%;
	}

	.detail-like-song {
		flex: 1;
		color: #c6c2bf;
	}

	.detail-like-song view:nth-child(1) {
		font-size: 28rpx;
		color: white;
		margin-bottom: 12rpx;
	}

	.detail-like-song view:nth-child(2) {
		display: flex;
		align-items: center;

	}

	.detail-like-song view:nth-child(2) text {
		font-size: 14rpx;
	}


	.detail-like-song image:nth-last-of-type(1) {
		width: 34rpx;
		height: 30rpx;
		margin-right: 10rpx;
	}

	.detail-like-song image:nth-last-of-type(2) {
		width: 40rpx;
		height: 34rpx;
		margin-right: 10rpx;
	}

	.detail-like-item text {
		font-size: 50rpx;
		color: #c6c2bf;
	}

	.detail-comment {
		margin: 0 30rpx;
	}

	.detail-comment-head {
		font-size: 36rpx;
		color: white;
		margin: 50rpx 0;
	}

	.detail-comment-item {
		margin-bottom: 28rpx;
		display: flex;
	}

	.detail-comment-img {
		width: 64rpx;
		height: 64rpx;
		border-radius: 50%;
		overflow: hidden;
		margin-right: 18rpx;
	}

	.detail-comment-img image {
		width: 100%;
		height: 100%;

	}

	.detail-comment-content {
		flex: 1;
		color: #cbcacf;
	}

	.detail-comment-title {
		display: flex;
		justify-content: space-between;
	}

	.detail-comment-name {
		color: #cbcacf;
	}

	.detail-comment-name view:nth-child(1) {
		font-size: 26rpx;
	}

	.detail-comment-name view:nth-child(2) {
		font-size: 20rpx;
	}

	.detail-comment-like {
		font-size: 28rpx;
		display: flex;
		align-items: center;
	}
.detail-comment-like text{
	padding-left: 6rpx ;
}
	.detail-comment-text {
		font-size: 28rpx;
		line-height: 40rpx;
		color: white;
		margin-top: 20rpx;
		border-bottom: 1px #e0e0e0 solid;
		padding-bottom: 40rpx;
	}
</style>