<template>
	<view class="home">
		<view class="list" v-for="(item, index) in list" :key="index">
			<text class="list-title">{{item.content}}</text>
		</view>
		<uni-load-more :loadingType="loadingType" :status="loadingStatus" :contentText="contentText"></uni-load-more>
	</view>
</template>

<script>
	import uniLoadMore from "@/components/uni-load-more.vue"
	export default {
		data() {
			return {
				list: [],
				page: 1,
				lineShow: false,
				loadingType: 0, //旧
				contentText: {
					contentdown: "上拉显示更多",
					contentrefresh: "正在加载...",
					contentnomore: "- 我是有底线的 -"
				}
			}
		},
		components: {
			uniLoadMore
		},
		onLoad() {
			this.getList()
		},
		onPullDownRefresh() {
			// 下拉刷新
			this.page = 1;
			this.loadingType = 0;
			this.getList()
		},
		onReachBottom() {
			// 上拉加载
			this.getReload();
		},
		methods: {
			getList() {
				var that = this;
				uni.request({
					url: 'https://v.juhe.cn/joke/content/text.php',
					data: {
						page: 1,
						pagesize: 10,
						key: '1462e56de07b06d9995657fa097fd0a5'
					},
					success: (res) => {
						uni.stopPullDownRefresh(); // 停止刷新
						that.list = res.data.result.data
					}
				});
			},
			// 上拉加载
			getReload() {
				var that = this;
				if (that.loadingType !== 0) {
					return;
				}
				that.loadingType = 1;
				uni.request({
					url: 'https://v.juhe.cn/joke/content/text.php',
					data: {
						page: that.page + 1,
						pagesize: 10,
						key: '1462e56de07b06d9995657fa097fd0a5'
					},
					success: (res) => {
						let moreList = res.data.result.data;
						if (moreList.length == 0) {
							that.lineShow = true;
							that.loadingType = 2;
						} else {
							that.page++;
							for (var i = 0; i < moreList.length; i++) {
								that.list.push(moreList[i]);
							}
							that.loadingType = 0;
						}
					}
				});
			}
		},
		onShareAppMessage(res) {
			return {
				title: '我是媛媛呀',
				path: '/pages/index/index'
			}
		}
	}
</script>

<style lang="scss">
	.home {
		width: 100%;
		display: flex;
		flex-direction: column;
		align-items: center;
		background-color: #f8f8f8;

		.list {
			width: 690upx;
			padding: 20upx 30upx;
			background-color: #fff;
			border-radius: 20upx;
			margin-top: 30upx;
			box-sizing: border-box;
		}

		.list-title {
			width: 690upx;
			font-size: 36upx;
			color: #000000;
			box-sizing: border-box;
		}

		.list-img {
			margin-bottom: 50px;
		}
	}
</style>
