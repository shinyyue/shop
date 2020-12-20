<template>
    <view>
        <view class='distribution-posters'>
            <swiper :indicator-dots="indicatorDots"
                    :autoplay="autoplay"
                    :circular="circular"
                    :interval="interval"
                    :duration="duration"
                    @change="bindchange"
                    previous-margin="40px"
                    next-margin="40px">
                <block v-for="(item,index) in spreadList"
                       :key="index">
                    <swiper-item>
                        <image :src="item.pic"
                               class="slide-image"
                               :class="swiperIndex == index ? 'active' : 'quiet'"
                               mode="aspectFill" />
                    </swiper-item>
                </block>
            </swiper>
            <view class='keep bg-color'
                  @click='savePosterPath'>保存海报</view>
        </view>
        <authorize @onLoadFun="onLoadFun"
                   :isAuto="isAuto"
                   :isShowAuth="isShowAuth"
                   @authColse="authColse"></authorize>
        <view class="canvas"
              v-show="showCanvas">
            <canvas style="width:750px;height:1190px;"
                    canvas-id="canvasOne"></canvas>
        </view>
    </view>
</template>

<script>
    import uQRCode from '@/js_sdk/uqrCode/uqrCode.js'
	import {
		getUserInfo,
		spreadBanner
	} from '@/api/user.js';
	import {
		toLogin
	} from '@/libs/login.js';
	import {
		mapGetters
	} from "vuex";
	// #ifdef MP
	import authorize from '@/components/Authorize';
	import {
		getQrcode
	} from '@/api/api.js';
	// #endif
	import {
		imageBase64
	} from "@/api/public";
	import base64src from '@/utils/base64ToPath';
	export default {
		components: {
			authorize,
		},
		data() {
			return {
				imgUrls: [],
				indicatorDots: false,
				circular: false,
				autoplay: false,
				interval: 3000,
				duration: 500,
				swiperIndex: 0,
				spreadList: [],
				userInfo: {},
				poster: '',
				isAuto: false, //没有授权的不会自动授权
				isShowAuth: false, //是否隐藏授权
				imagePath: '',
				qrcodeSize: 1000,
				PromotionCode: '',
                base64List: [],
				uQRCode: '',
                picture: '',
                showCanvas: true
			};
		},
		computed: mapGetters(['isLogin']),
		onLoad() {
			if (this.isLogin) {
                this.userSpreadBannerList();
                this.userInfos();
			} else {
				toLogin();
				this.isAuto = true;
				this.$set(this, 'isShowAuth', true)
			}
		},
		/**
		 * 用户点击右上角分享
		 */
		onShareAppMessage: function() {
			return {
				title: this.userInfo.nickname + '-分销海报',
				imageUrl: this.spreadList[0].pic,
				path: '/pages/index/index?spid=' + this.userInfo.uid,
			};
		},
		onReady() {
		},
		methods: {
			// 小程序二维码
			getQrcode(){
				let data = {
					pid: this.userInfo.uid,
					path: '/pages/index/index'
				}
				getQrcode(data).then(res=>{
                    this.$set(this, 'PromotionCode', res.data.code)
					this.PosterCanvas(this.spreadList[0].pic, res.data.code, this.userInfo.nickname,0);
				})
            },
			PosterCanvas(arrImages, code, nickname,index) {
                let that = this;
				let context = uni.createCanvasContext('canvasOne')
				context.clearRect(0, 0, 0, 0);
				uni.getImageInfo({
					src: arrImages,
					success: (res) => {
						base64src(code).then(codePath => {
							context.drawImage(res.path, 0, 0, 750, 1190);
							context.save();
							context.drawImage(codePath, 110, 925, 200, 200);

							context.setFontSize(28);
							context.fillText(nickname, 340, 980);
							context.fillText('邀请您加入', 340, 1020);

							setTimeout(() => {
								context.draw(true,() => {
									uni.canvasToTempFilePath({
										destWidth: 750,
										destHeight: 1190,
										canvasId: 'canvasOne',
										fileType: 'jpg',
										success: (res) => {
                                            uni.hideLoading();
                                            that.$set(that, 'showCanvas', false)
											that.spreadList[index].pic = res.tempFilePath;
										} 
									})
								})
							}, 500)
						})
					},
					fail: (err) => {
                        uni.hideLoading();
						this.$util.Tips({
							title: JSON.stringify(err) || '无法获取图片信息'
                        });
					}
				});
			},
			onLoadFun: function(e) {
				this.$set(this, 'userInfo', e);
				this.userSpreadBannerList();
			},
			// 授权关闭
			authColse: function(e) {
				this.isShowAuth = e
			},
			bindchange(e) {
				let index = e.detail.current;
				this.swiperIndex = index;
                this.PosterCanvas(this.spreadList[index].pic, this.PromotionCode, this.userInfo.nickname,index);
                this.$set(this, 'showCanvas', true)
                let context = uni.createCanvasContext('canvasOne')
                context.clearRect(0, 0, 0, 0);
                context.draw();
            },
			// 点击保存海报
			savePosterPath() {
                let that = this;
                const path = this.spreadList[this.swiperIndex].pic;
                uni.getSetting({
                    success(res) {
                        if (!res.authSetting['scope.writePhotosAlbum']) {
                            uni.authorize({
                                scope: 'scope.writePhotosAlbum',
                                success() {
                                    uni.saveImageToPhotosAlbum({
                                        filePath: path,
                                        success: function(res) {
                                            return that.$util.Tips({
                                                title: '保存成功'
                                            });
                                        },
                                        fail: function(res) {
                                            return that.$util.Tips({
                                                title: res.errMsg
                                            });
                                        },
                                        complete: function(res) {
                                        },
                                    })
                                },
                                fail() {
                                    uni.showModal({
                                        title: '您已拒绝获取相册权限',
                                        content: '是否进入权限管理，调整授权？',
                                        success(res) {
                                            if (res.confirm) {
                                                uni.openSetting({
                                                    success: function(res) {
                                                        console.log(res.authSetting)
                                                    }
                                                });
                                            } else if (res.cancel) {
                                                return that.$util.Tips({
                                                    title: '已取消！'
                                                });
                                            }
                                        }
                                    })
                                }
                            })
                        } else {
                            uni.saveImageToPhotosAlbum({
                                filePath: path,
                                success: function(res) {
                                    return that.$util.Tips({
                                        title: '保存成功'
                                    });
                                },
                                fail: function(res) {
                                    return that.$util.Tips({
                                        title: res.errMsg
                                    });
                                }
                            })
                        }
                    },
                    fail(err) {
                        console.log('getSetting---', err)
                    }
                })
			},
			userInfos() {
				getUserInfo().then(res => {
					this.userInfo = res.data;
					this.getQrcode();
				})
			},
			// setShareInfoStatus: function() {
			// 	if (this.$wechat.isWeixin()) {
			// 		let configAppMessage = {
			// 			desc: '分销海报',
			// 			title: this.userInfo.nickname + '-分销海报',
			// 			link: '/pages/index/index?spread=' + this.userInfo.uid,
			// 			imgUrl: this.spreadList[0].pic
			// 		};
			// 		this.$wechat.wechatEvevt(["updateAppMessageShareData", "updateTimelineShareData"], configAppMessage)
			// 	}
			// },
			userSpreadBannerList: function() {
				let that = this;
				uni.showLoading({
					title: '获取中',
					mask: true,
				})
				spreadBanner({
					page: 1,
					limit: 5
				}).then(res => {
					uni.hideLoading();
					that.$set(that, 'spreadList', res.data);
				}).catch(err => {
					uni.hideLoading();
				});
			}
		}
	}
</script>

<style lang="scss">
	page {
		background-color: #a3a3a3 !important;
	}
	.canvas canvas{
		position: fixed;
        top: 0;
        left: 0;
        opacity: 0;
		z-index: -5;
	}
	.distribution-posters swiper {
		width: 100%;
		height: 1000rpx;
		position: relative;
		margin-top: 40rpx;
	}

	.distribution-posters .slide-image {
		width: 100%;
		height: 100%;
		margin: 0 auto;
		border-radius: 15rpx;
	}

	.distribution-posters .slide-image.active {
		transform: none;
		transition: all 0.2s ease-in 0s;
	}

	.distribution-posters .slide-image.quiet {
		transform: scale(0.8333333);
		transition: all 0.2s ease-in 0s;
	}

	.distribution-posters .keep {
		font-size: 30rpx;
		color: #fff;
		width: 600rpx;
		height: 80rpx;
		border-radius: 50rpx;
		text-align: center;
		line-height: 80rpx;
		margin: 38rpx auto;
	}

	.distribution-posters .preserve {
		color: #fff;
		text-align: center;
		margin-top: 38rpx;
	}

	.distribution-posters .preserve .line {
		width: 100rpx;
		height: 1px;
		background-color: #fff;
	}

	.distribution-posters .preserve .tip {
		margin: 0 30rpx;
	}
</style>
