<template>
	<view class="index">
		<musichead title="Music" :icon="false"></musichead>
		<view class="container">
			<scroll-view scroll-y="true">
				<view class="index-search" @tap="handleToSearch">
					<text class="iconfont icon-fangdajing"></text>
					<input type="text" placeholder="搜索歌曲">
				</view>
				<view class="" v-if="isLoading">
					<m-for-skeleton :avatarSize="200" :row="3" :loading="isLoading" isarc="square"
						v-for="(item,key) in 4" :key="key" :titleStyle="{}" :title="false">
					</m-for-skeleton>
				</view>
				<view class="index-list" v-else>
					<view class="index-list-item" v-for="(item,index) in topList" :key="item.id"
						@tap="handleToList(item.listId)">
						<view class="index-list-img">
							<image :src="item.coverImgUrl"></image>
							<text>{{item.updateFrequency}}</text>
						</view>
						<view class="index-list-text">
							<view v-for="(item,index) in item.tracks">
								{{index + 1}}.{{item.first}} - {{ item.second }}
							</view>
						</view>
					</view>


				</view>
			</scroll-view>
		</view>
	</view>
</template>

<script>
	import mForSkeleton from "@/components/m-for-skeleton/m-for-skeleton";
	import "@/common/iconfont.css"
	import musichead from "../../components/musichead/musichead.vue"
	import {
		topList
	} from "../../common/api.js"
	export default {
		data() {
			return {
				topList: [],
				isLoading: true
			}
		},
		components: {
			musichead,
			mForSkeleton
		},
		onLoad() {
			topList().then((res) => {
				console.log(res);
				if (res.length) {
					setTimeout(() => {
						this.topList = res
						this.isLoading = false
					},800)
				}

			})
		},
		methods: {
			handleToList(listId) {
				uni.navigateTo({
					url: "/pages/list/list?listId=" + listId
				})
			},
			handleToSearch() {
				uni.navigateTo({
					url: "/pages/search/search"
				})
			}
		}
	}
</script>

<style scoped>
	.index {}

	.index-search {
		display: flex;
		align-items: center;
		height: 70rpx;
		margin: 70rpx 30rpx 30rpx 30rpx;
		background-color: #f7f7f7;
		border-radius: 50rpx;
	}

	.index-search text {
		font-size: 26rpx;
		margin-right: 26rpx;
		margin-left: 28rpx;
	}

	.index-search input {
		font-size: 28rpx;
		flex: 1;
	}

	.index-list {
		margin: 0 30rpx;
	}

	.index-list-item {
		display: flex;
		margin-bottom: 34rpx;
	}

	.index-list-img {
		position: relative;
		width: 212rpx;
		height: 212rpx;
		border-radius: 30rpx;
		overflow: hidden;
		margin-right: 22rpx;
	}

	.index-list-img image {
		width: 100%;
		height: 100%;
	}

	.index-list-img text {
		position: absolute;
		left: 12rpx;
		bottom: 16rpx;
		color: white;
		font-size: 20rpx;
	}

	.index-list-text {
		flex: 1;
		font-size: 24rpx;
		line-height: 66rpx;
		overflow: hidden;
		white-space: nowrap;
	}
</style>