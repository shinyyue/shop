<template>
    <view class="memberVip">
        <view class="bg">
            <view class="experience">
                <view class="picTxt acea-row row-middle">
                    <view class="pictrue">
                        <image class="avator-img"
                               :src="userInfo.avatar"></image>
                    </view>
                    <view class="user-info">
                        <view class="name line1">{{userInfo.nickname}}</view>
                        <view class="vip">
                            <image class="vip-icon"
                                :src="userInfo.vipIcon"></image>
                            <view>{{userInfo.vipName}}</view>
                        </view>
                    </view>
                </view>
                <view class="vip-experence">
                    <view class="title">当前经验值</view>
				    <view class="num">{{levelInfo}}</view>
                </view>
                <view class="axis">
                    <view class="bar">
                        <view class="barCon">
                            <view class="solidBar"
                                  :style="'width: ' + widthLen +'%;'"></view>
                        </view>
                        <!-- <view class="acea-row row-around row-middle">
                            <view class="spotw acea-row row-center"
                                  v-for="(item,index) in levelList"
                                  :key='index'>
                                <view class="spot"
                                      :class="current >item.experience?'past':'' + ' ' + current==item.experience?'on':''"></view>
                            </view>
                        </view> -->
                    </view>
                    <view class="numList acea-row row-around row-middle">
                        <view class="item"
                              :class="current >=item.experience?'past':''"
                              v-for="(item,index) in levelList"
                              :key="index">{{item.experience}}</view>
                    </view>
                </view>
            </view>
            <view class="module">
                <view class="public_title acea-row row-middle">
                    <view class="icons"></view>获取经验
                </view>
                <view class="gainList">
                    <view class="item acea-row row-between-wrapper">
                        <view class="picTxt acea-row row-middle">
                            <view class="pictrue"><text class="iconfont icon-qiandao2"></text></view>
                            <view class="text">
                                <view class="name line1">签到</view>
                                <view class="info line1">每日签到可获得经验值</view>
                            </view>
                        </view>
                        <navigator url='/pages/users/user_sgin/index'
                                   class="button"
                                   hover-class="none">去获取</navigator>
                    </view>
                    <view class="item acea-row row-between-wrapper">
                        <view class="picTxt acea-row row-middle">
                            <view class="pictrue on"><text class="iconfont icon-shangpin"></text></view>
                            <view class="text">
                                <view class="name line1">购买商品</view>
                                <view class="info line1">购买商品可获得对应是经验值</view>
                            </view>
                        </view>
                        <navigator url="/pages/goods_cate/goods_cate"
                                   class="button"
                                   hover-class="none"
                                   open-type='switchTab'>去获取</navigator>
                    </view>
                    <view class="item acea-row row-between-wrapper">
                        <view class="picTxt acea-row row-middle">
                            <view class="pictrue on2"><text class="iconfont icon-yaoqing"></text></view>
                            <view class="text">
                                <view class="name line1">邀请好友</view>
                                <view class="info line1">邀请好友注册商城可获得经验值</view>
                            </view>
                        </view>
                        <navigator url="/pages/users/user_spread_code/index"
                                   class="button"
                                   hover-class="none">去获取</navigator>
                    </view>
                </view>
            </view>
        </view>
        <view class="detailed"
              v-if="expList.length">
            <view class="public_title acea-row row-middle">
                <view class="icons"></view>经验值明细
            </view>
            <view class="list">
                <view class="item acea-row row-between-wrapper"
                      v-for="(item,index) in expList"
                      :key="index">
                    <view class="text">
                        <view class="name">{{item.title}}</view>
                        <view class="data">{{item.add_time}}</view>
                    </view>
                    <view class="num"
                          v-if="item.pm">+{{item.number}}</view>
                    <view class="num on"
                          v-else>-{{item.number}}</view>
                </view>
            </view>
        </view>
        <view class='loadingicon acea-row row-center-wrapper'
              v-if="expList.length">
            <text class='loading iconfont icon-jiazai'
                  :hidden='loading==false'></text>{{loadTitle}}
        </view>
    </view>
</template>

<script>
	import {
		getUserInfo,
		getlevelInfo,
		getlevelExpList
	} from '@/api/user.js';
	export default {
		data() {
			return {
				userInfo: '',
				levelInfo: '',
				levelList: [],
				current: 0,
				widthLen: 0,
				loading: false,
				loadend: false,
				loadTitle: '加载更多', //提示语
				page: 1,
				limit: 20,
				expList: []
			};
		},
		onShow() {
			this.getUserInfo();
			this.getInfo();
			this.getlevelList();
		},
		methods: {
			/**
			 * 获取个人用户信息
			 */
			getUserInfo: function() {
				let that = this;
				getUserInfo().then(res => {
					that.userInfo = res.data;
                    // that.levelInfo = res.data.experience;
                    this.levelInfo = res.data.integral;
				}).catch(function(res) {
					return that.$util.Tips({
						title: res.msg
					});
				})
			},
			getInfo: function() {
				let that = this;
				getlevelInfo().then(res => {
					let levelList = res.data.list;
					let list = []
					that.levelList = levelList;
					levelList.forEach((item, index) => {
						if (item.experience <= that.levelInfo) {
							list.push(item.experience)
						}
					})
					let maxn = Math.max.apply(null, list);
					that.current = maxn;
					// 解决len取的值没有时；
					let levelListLen = levelList[list.length] ? levelList[list.length] : levelList[list.length - 1];
					// 解决除数不能为0
					let divisor = levelListLen.experience - maxn ? levelListLen.experience - maxn : 1;
					// 每小格所占的百分比
					let per = (that.levelInfo - maxn) / divisor / levelList.length;

					that.widthLen = ((list.length - 0.5) / (levelList.length)) * 100 + per * 100

				}).catch(function(res) {
					return that.$util.Tips({
						title: res.msg
					});
				})
			},
			getlevelList: function() {
				let that = this
				if (this.loadend) return false;
				if (this.loading) return false;
				getlevelExpList({
					page: that.page,
					limit: that.limit
				}).then(res => {
					let list = res.data.list,
						loadend = list.length < that.limit;
					let expList = that.$util.SplitArray(list, that.expList);
					that.$set(that, 'expList', expList);
					that.loadend = loadend;
					that.loadTitle = loadend ? '我也是有底线的' : '加载更多';
					that.page = that.page + 1;
					that.loading = false;
				}).catch(err => {
					that.loading = false;
					that.loadTitle = '加载更多';
				});
			}
		},
		onReachBottom: function() {
			this.getlevelList();
		}
	}
</script>

<style lang="scss">
	.memberVip {
		.bg {
			background-color: #fff;

			.experience {
				background-image: url('data:image/jpg;base64,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');
				background-repeat: no-repeat;
				background-size: 100% 100%;
				width: 690rpx;
				height: 360rpx;
				margin: 20rpx auto;
				border-radius: 23rpx;
				padding: 22rpx 27rpx;
				box-sizing: border-box;
                .avator-img {
                    width: 120rpx;
                    height: 120rpx;
                    border-radius: 120rpx;
                    margin-right: 20rpx;
                }

                .vip-experence {
                    display: flex;
                    flex-direction: row;
                    justify-content: flex-start;
                    align-items: center;
                    margin-top: 30rpx;
                    margin-bottom: 20rpx;
                }

				.title {
					font-size: 24rpx;
					color: #AE8B4A;
				}

				.num {
					font-size: 40rpx;
					color: #775C29;
					margin-left: 10rpx;
				}

				.axis {
					margin: 10rpx 0 15rpx 0;

					.bar {
						width: 630rpx;
						.spotw{
							width: 96rpx;
						}
						.barCon {
							width: 100%;
							height: 3rpx;
							background: #D7BD89;
							border-radius: 2rpx;

							.solidBar {
								width: 0;
								height: 100%;
								background: #775C29;
								border-radius: 2rpx;
								transition: width 0.6s ease;
							}
						}

						.spot {
							width: 8rpx;
							height: 8rpx;
							background: #D7BD89;
							border-radius: 50%;
							margin-top: -5rpx;

							&.past {
								background: #775C29;
							}

							&.on {
								background: #775C29;
								box-shadow: 0rpx 0rpx 8rpx #000;
							}
						}
					}

					.numList {
						font-size: 26rpx;
						color: #D7BD89;
						margin-top: 14rpx;

						.item {
							width: 96rpx;
							text-align: center;

							&.past {
								color: #775C29;
							}
						}
					}
				}

				.vipList {
					.item {
						width: 20%;
						text-align: center;

						.pictrue {
							width: 70rpx;
							height: 70rpx;
							margin: 0 auto;

							image {
								width: 100%;
								height: 100%;
							}
						}

						.name {
							font-size: 24rpx;
							color: #403D4E;
							margin-top: 18rpx;
						}
					}
				}

                .vip-icon {
                    width: 30rpx;
                    height: 30rpx;
                    margin-right: 10rpx;
                }
			}

			.module {
				padding: 40rpx 30rpx 0 30rpx;

				.gainList {
					margin-top: 10rpx;

					.item {
						height: 130rpx;
						position: relative;

						.picTxt {
							.pictrue {
								width: 70rpx;
								height: 70rpx;
								border-radius: 50%;
								background: linear-gradient(-45deg, rgba(249, 119, 107, 1) 0%, rgba(255, 147, 137, 1) 100%);
								text-align: center;
								line-height: 70rpx;
								color: #fff;

								&.on {
									background: linear-gradient(-45deg, rgba(254, 160, 96, 1) 0%, rgba(255, 204, 170, 1) 100%);
								}

								&.on2 {
									background: linear-gradient(-45deg, rgba(157, 208, 116, 1) 0%, rgba(161, 214, 124, 1) 100%);
								}
							}

							.text {
								margin-left: 30rpx;
								width: 400rpx;

								.name {
									font-size: 30rpx;
									color: #282828;
								}

								.info {
									font-size: 24rpx;
									color: #999999;
									margin-top: 6rpx;
								}
							}
						}

						.button {
							width: 140rpx;
							height: 50rpx;
							background: linear-gradient(-90deg, rgba(231, 182, 103, 1) 0%, rgba(255, 234, 181, 1) 100%);
							border-radius: 25rpx;
							text-align: center;
							line-height: 50rpx;
							font-size: 26rpx;
							color: #8D5306;
						}

						&~.item {
							&::after {
								position: absolute;
								content: ' ';
								width: 720rpx;
								height: 1rpx;
								background: rgba(245, 245, 245, 1);
								top: 0;
								left: 0;
							}
						}
					}
				}
			}
		}

		.public_title {
			color: #282828;
			font-size: 30rpx;

			.icons {
				width: 6rpx;
				height: 28rpx;
				background: rgba(230, 192, 131, 1);
				margin-right: 10rpx;
			}
		}

		.detailed {
			padding: 30rpx 30rpx 0 30rpx;
			margin-top: 15rpx;
			background-color: #fff;

			.list {
				margin-top: 15rpx;

				.item {
					height: 122rpx;
					border-bottom: 1px solid #EEEEEE;
					
					.text {
						.name {
							font-size: 28rpx;
							color: #282828;
						}

						.data {
							color: #999;
							font-size: 24rpx;
						}
					}

					.num {
						font-size: 32rpx;
						color: #16AC57;
					}
					.on{
						color: #E93323;
					}
				}
			}
		}

        .vip {
            display: flex;
            flex-direction: 'row';
            justify-content: flex-start;
            align-items: center;
        }

        .user-info {
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            align-items: flex-start;
        }
	}
</style>
