<template>
	<view :style="colorStyle">
		<view class="promoter-order">
			<view class='promoterHeader bg-color'>
				<view class='headerCon acea-row row-between-wrapper'>
					<view>
						<view class='name'>累积推广订单</view>
						<view><text class='num'>{{count || 0}}</text>单</view>
					</view>
					<view class='iconfont icon-2'></view>
				</view>
			</view>
			<view class='list' v-if="recordList.length>0">
				<block v-for="(item,index) in recordList" :key="index">
					<view class='item'>
						<view class='title acea-row row-column row-center'>
							<view class='data'>{{item.time}}</view>
							<view>本月累计推广订单：{{item.count || 0}}单</view>
						</view>
						<view class='listn'>
							<block v-for="(child,indexn) in item.child" :key="indexn">
								<view class='itenm'>
									<view class='top acea-row row-between-wrapper'>
										<view class='pictxt acea-row row-between-wrapper'>
											<view class='pictrue'>
												<image :src='child.avatar'></image>
											</view>
											<view class='text line1'>{{child.nickname}}</view>
										</view>
										<view class='money' v-if="child.type == 'brokerage'">返佣：<text
												class='font-color'>￥{{child.number}}</text></view>
										<view class='money' v-else>暂未返佣：<text
												class='font-color'>￥{{child.number}}</text></view>
									</view>
									<view class='bottom'>
										<view><text class='name'>订单编号：</text>{{child.order_id}}</view>
										<view v-if="child.type == 'brokerage'"><text
												class='name'>返佣时间：</text>{{child.time}}</view>
										<view v-else><text class='name'>下单时间：</text>{{child.time}}</view>
										<view class="more" v-if="child.children && child.children.length" @click="open(child)">
											{{child.open?"收起":"更多"}}
											<text class="iconfont" :class="child.open?'icon-xiangshang':'icon-xiangxia'"></text>
										</view>
									</view>
									<view class="more-record" v-if="child.open">
										<view class="more-record-list" v-for="(sp,indexs) in child.children" :key="indexs">
											<view class="more-record-box">
												<view><text class='name'>单号：</text>{{sp.order_id}}</view>
												<view class='money' v-if="sp.type == 'brokerage'">返佣：<text
														class='font-color'>￥{{sp.number}}</text></view>
												<view class='money' v-else>暂未返佣：<text
														class='font-color'>￥{{sp.number}}</text></view>
											</view>
										</view>
									</view>
								</view>
							</block>
						</view>
					</view>
				</block>
			</view>
			<view v-if="recordList.length == 0">
				<emptyPage title="暂无推广订单～"></emptyPage>
			</view>
		</view>
		<!-- #ifdef MP -->
		<!-- <authorize @onLoadFun="onLoadFun" :isAuto="isAuto" :isShowAuth="isShowAuth" @authColse="authColse"></authorize> -->
		<!-- #endif -->
		<!-- #ifndef MP -->
		<home></home>
		<!-- #endif -->
	</view>
</template>

<script>
	import {
		spreadOrder
	} from '@/api/user.js';
	import {
		toLogin
	} from '@/libs/login.js';
	import {
		mapGetters
	} from "vuex";
	// #ifdef MP
	import authorize from '@/components/Authorize';
	// #endif
	import emptyPage from '@/components/emptyPage.vue'
	import home from '@/components/home';
	import colors from '@/mixins/color.js';
	export default {
		components: {
			// #ifdef MP
			authorize,
			// #endif
			emptyPage,
			home
		},
		mixins: [colors],
		data() {
			return {
				page: 1,
				limit: 5,
				status: false,
				recordList: [],
				times: [],
				recordCount: 0,
				count: 0,
				isAuto: false, //没有授权的不会自动授权
				isShowAuth: false //是否隐藏授权
			};
		},
		computed: mapGetters(['isLogin']),
		onLoad() {
			if (this.isLogin) {
				this.getRecordOrderList();
			} else {
				toLogin();
			}
		},
		methods: {
			open(item){
				item.open = !item.open
			},
			onLoadFun() {
				this.getRecordOrderList();
			},
			// 授权关闭
			authColse: function(e) {
				this.isShowAuth = e
			},
			getRecordOrderList: function() {
				let that = this;
				let page = that.page;
				let limit = that.limit;
				let status = that.status;
				if (status == true) return;
				spreadOrder({
					page: page,
					limit: limit
				}).then(res => {
					for (let i = 0; i < res.data.time.length; i++) {
						if (!this.times.includes(res.data.time[i].time)) {
							this.times.push(res.data.time[i].time)
							this.recordList.push({
								time: res.data.time[i].time,
								count: res.data.time[i].count,
								child: []
							})
						}
					}
					for (let x = 0; x < this.times.length; x++) {
						for (let j = 0; j < res.data.list.length; j++) {
							if (this.times[x] === res.data.list[j].time_key) {
								res.data.list[j].open = false
								this.recordList[x].child.push(res.data.list[j])
							}
						}
					}
					that.count = res.data.count || 0;
					that.status = res.data.list.length < 5;
					that.page += 1;
				});
			}
		},
		onReachBottom: function() {
			this.getRecordOrderList();
		}
	}
</script>

<style scoped lang="scss">
	.promoter-order .list .item .title {
		height: 133rpx;
		padding: 0 30rpx;
		font-size: 26rpx;
		color: #999;
	}

	.promoter-order .list .item .title .data {
		font-size: 28rpx;
		color: #282828;
		margin-bottom: 5rpx;
	}

	.promoter-order .list .item .listn .itenm {
		background-color: #fff;
		// margin: 0 $uni-index-margin-row;
		border-radius: 8rpx;
		.more-record{
			color: #999;
			font-size: 24rpx;
			.more-record-list{
				padding: 20rpx 30rpx;
				border-top: 1px solid #f2f2f2;
				.more-record-box{
					display: flex;
					justify-content: space-between;
				}
			}
		}
	}

	.promoter-order .list .item .listn .itenm~.itenm {
		margin-top: 12rpx;
	}

	.promoter-order .list .item .listn .itenm .top {
		margin-left: 30rpx;
		padding-right: 30rpx;
		border-bottom: 1rpx solid #eee;
		height: 100rpx;
	}

	.promoter-order .list .item .listn .itenm .top .pictxt {
		width: 320rpx;
	}

	.promoter-order .list .item .listn .itenm .top .pictxt .text {
		width: 230rpx;
		font-size: 30rpx;
		color: #282828;
	}

	.promoter-order .list .item .listn .itenm .top .pictxt .pictrue {
		width: 66rpx;
		height: 66rpx;
	}

	.promoter-order .list .item .listn .itenm .top .pictxt .pictrue image {
		width: 100%;
		height: 100%;
		border-radius: 50%;
		border: 3rpx solid #fff;
		box-sizing: border-box;
		box-shadow: 0 0 15rpx #aaa;
	}

	.promoter-order .list .item .listn .itenm .top .money {
		font-size: 28rpx;
	}

	.promoter-order .list .item .listn .itenm .bottom {
		padding: 20rpx 30rpx;
		font-size: 28rpx;
		color: #666;
		line-height: 1.6;
		position: relative;

		.more {
			font-size: 24rpx;
			position: absolute;
			right: 12rpx;
			bottom: 24rpx;
			.iconfont {
				font-size: 22rpx;
				margin-left: 5rpx;
			}
		}
	}

	.promoter-order .list .item .listn .itenm .bottom .name {
		color: #999;
	}
</style>
