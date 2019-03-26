<template>
	<view class="home">
		<view class="list" v-for="(item, index) in list" :key="index" v-if="item.videouri==''">
			<text class="list-title">{{item.text}}</text>
			<!-- <video id="myVideo" :src="item.videouri" :poster='item.cdn_img' v-if="item.videouri!==''"></video> -->
			<image class="list-img" mode='widthFix' :src="item.cdn_img" @click="previewImage(item.cdn_img)"></image>
			<!-- <image class="list-img" mode='widthFix' :src="item.cdn_img" v-if='item.cdn_img'></image>
			<image class="list-img" mode='widthFix' :src="item.image0" v-else></image> -->
		</view>
		<!-- <view class="mask" v-if="btnShow">
			<view class="fixed-box">
				<image class="logo" src="../../static/logo.jpg"></image>
				<button class="login-btn" type="warn" open-type="getUserInfo" @getuserinfo='openInfo()'>请点击我</button>
			</view>
		</view> -->
		<uni-load-more :loadingType="loadingType" :status="loadingStatus" :contentText="contentText"></uni-load-more>
	</view>
</template>

<script>
	import uniLoadMore from "@/components/uni-load-more.vue"
	export default {
		data() {
			return {
				btnShow: false,
				list: [],
				maxtime: '',
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
			this.getList();
			// this.getInfo()
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
			getInfo() {
				var that = this;
				// 获取用户信息
				uni.getUserInfo({
					provider: 'weixin',
					success: function(infoRes) {
						that.btnShow = false;
					},
					fail: function(ret) {
						that.btnShow = true;
					}
				});
			},
			getList() {
				var that = this;
				uni.request({
					url: 'https://api.budejie.com/api/api_open.php',
					data: {
						a: 'newlist',
						c: 'data',
						page: 1,
						count: 10,
						// 上一页的maxtime作为加载下一页的条件，
						// maxtime: that.maxtime,
						type: '10',
					},
					success: (res) => {
						uni.stopPullDownRefresh(); // 停止刷新
						that.maxtime = res.data.info.maxtime;
						that.list = res.data.list;
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
					url: 'https://api.budejie.com/api/api_open.php',
					data: {
						a: 'list',
						c: 'data',
						// 上一页的maxtime作为加载下一页的条件，
						maxtime: that.maxtime,
						type: '10',
						page: that.page + 1,
					},
					success: (res) => {
						let moreList = res.data.list;
						that.maxtime = res.data.info.maxtime;
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
			},
			// 预览图片(类型必须是数组)
			previewImage(img) {
				var that = this;
				var imgList = [];
				imgList.push(img);
				uni.previewImage({
					urls: imgList
				});
			},
			openInfo() {
				var that = this;
				uni.login({
					provider: 'weixin',
					success: function(loginRes) {
						that.btnShow = false;
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

		.mask {
			position: fixed;
			top: 0;
			left: 0;
			right: 0;
			bottom: 0;
			width: 750upx;
			display: flex;
			align-items: center;
			justify-content: center;
			background-color: rgba($color: #000000, $alpha: 0.4)
		}

		.fixed-box {
			width: 500upx;
			height: 500upx;
			display: flex;
			flex-direction: column;
			align-items: center;
			justify-content: space-around;
			border-radius: 20upx;
			background-color: #fff;
		}

		.logo {
			width: 325upx;
			height: 250upx;
		}

		.list {
			width: 690upx;
			display: flex;
			flex-direction: column;
			align-items: center;
			background-color: #fff;
			border-radius: 20upx;
			margin-bottom: 20upx;
			margin-top: 30upx;
		}

		.list-title {
			font-size: 36upx;
			color: #000000;
			padding: 20upx 30upx;
			box-sizing: border-box;
		}

		.list-img {
			margin-bottom: 50px;
		}
	}
</style>
