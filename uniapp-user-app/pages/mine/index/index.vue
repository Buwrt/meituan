<template>
    <view class="container">
        <image :src="'https://siam-hangzhou.oss-cn-hangzhou.aliyuncs.com/data/images/business/mypage_top.jpg?v=' + appVersion" class="index-bg-class" mode="widthFix"></image>
        <view class="userinfo" :style="'margin-top:' + (statusBarHeight + 30) + 'rpx;'">
            <block v-if="!data">
                <button @tap="getUserProfile" :plain="true" class="userinfo-button" style="margin: 0; width: 100%; padding: inherit">
                    <view class="notlogin">
                        <image class="userinfo-avatar" src="/static/assets/images/user-head.png" mode="cover"></image>
                        <!-- <text class="userinfo-nickname">{{userInfo.nickName}}</text> -->
                        请登录/注册
                    </view>
                </button>
            </block>
            <block v-else>
                <navigator url="../userinfo/userinfo" class="navigator-userinfo">
                    <view class="userinfo-view">
                        <image class="userinfo-avatar" :src="data.headImg ? data.headImg : '../../../assets/images/user-head.png'" mode="cover"></image>
                        <view class="userinfo-nickname">
                            <view class="nickname-class">{{ data.username }}</view>
                            <view class="vip-class">
                                <!-- <text class="{{data.type!=1?'is-vip-class':'not-vip-class'}}">{{data.typeVipText}}</text> -->
                                <!-- <text decode="true">&nbsp;&nbsp;</text> -->
                                <view class="vip-image-view">
                                    <image :src="item + '?v=' + appVersion" mode="widthFix" class="is-vip-image" v-for="(item, index) in isVipImages" :key="index"></image>
                                </view>
                            </view>
                            <view>
                                <text :decode="true">NO:&nbsp;&nbsp;{{ data.vipNo }}</text>
                            </view>
                        </view>
                    </view>
                    <text class="iconfont iconhtbArrowright02"></text>
                </navigator>
            </block>
            <view v-if="data">
                <view class="mine-blocking-view">
                    <view class="mine-blocking">
                        <navigator class="mine-navigator" url="../share/reward/reward">
                            <view class="number-tip theme-color">{{ data.inviteRewardAmount }}</view>
                            <view class="text-tip">佣金</view>
                        </navigator>
                        <text class="line-tip">|</text>
                        <navigator class="mine-navigator" url="../integral/integral">
                            <view class="number-tip theme-color">{{ data.points }}</view>
                            <view class="text-tip">我的积分</view>
                        </navigator>
                        <text class="line-tip">|</text>
                        <navigator class="mine-navigator" url="../coupons/coupons">
                            <view class="number-tip theme-color">{{ data.couponsNumber ? data.couponsNumber : 0 }}</view>
                            <view class="text-tip">优惠券</view>
                        </navigator>
                        <text class="line-tip">|</text>
                        <navigator class="mine-navigator" url="../member/index/index">
                            <view class="number-tip theme-color">{{ data.balance }}</view>
                            <view class="text-tip">会员中心</view>
                        </navigator>
                    </view>
                </view>
                <view class="my-order-box">
                    <view class="my-order-title my-order-all" @tap="bindOrderInfo">
                        <view class="order-title">我的订单</view>
                        <view class="order-all">
                            <text>查看全部</text>
                            <text class="iconfont iconhtbArrowright02"></text>
                        </view>
                    </view>
                    <view class="swiper-content">
                        <view class="swiper-tab self-adaption">
                            <view class="swiper-tab-item" :data-current="index" @tap="clickTab" hover-class="hover-click-class" v-for="(item, index) in tabList" :key="index">
                                <view :class="'swiper_table_item_view ' + (currentTab == item.modeId ? 'active_' : '')" :data-current="index" @tap="clickTab">
                                    {{ item.modeName }}
                                </view>
                            </view>
                        </view>
                    </view>
                    <swiper :current="currentTab" class="swiper-box" duration="300" @change="bindSlideChange" style="height: 78px">
                        <swiper-item class="swiper-items">
                            <view class="mine-blocking">
                                <navigator
                                    class="order-navigator"
                                    :url="'../../order/index/index?currentTab=' + currentTab + '&modeType=' + item.modeType + '&currentOrderTab=' + item.modeId"
                                    v-for="(item, index) in shopOrderTabList"
                                    :key="index"
                                >
                                    <text :class="'iconfont ' + item.icon"></text>

                                    <view class="text-tip">{{ item.modeName }}</view>
                                </navigator>
                            </view>
                        </swiper-item>
                        <swiper-item class="swiper-items">
                            <view class="mine-blocking">
                                <navigator
                                    class="order-navigator"
                                    :url="'../../order/index/index?currentTab=' + currentTab + '&modeType=' + item.modeType + '&currentOrderTab=' + item.modeId"
                                    v-for="(item, index) in pointsOrderTabList"
                                    :key="index"
                                >
                                    <text :class="'iconfont ' + item.icon"></text>

                                    <view class="text-tip">{{ item.modeName }}</view>
                                </navigator>
                            </view>
                        </swiper-item>
                    </swiper>
                    <navigator class="navigator-class" url="../collect/index/index">
                        <view class="navigator-view">
                            <view>我的收藏</view>
                            <text class="iconfont iconhtbArrowright02"></text>
                        </view>
                    </navigator>
                </view>
                <view class="navigator-box">
                    <view class="navigator-radius">
                        <navigator class="navigator-class" url="../../address/index/index">
                            <view class="navigator-view">
                                <view>收货地址</view>
                                <text class="iconfont iconhtbArrowright02"></text>
                            </view>
                        </navigator>
                        <view class="view-line"></view>
                        <navigator class="navigator-class" url="../security/index/index">
                            <view class="navigator-view">
                                <view>账号安全</view>
                                <text class="iconfont iconhtbArrowright02"></text>
                            </view>
                        </navigator>
                        <!-- 分享 -->
                        <view class="view-line"></view>
                        <navigator class="navigator-class" :url="'../../mine/share/index/index?inviterId=' + data.id">
                            <view class="invite-wrapper">
                                <image
                                    :src="'https://siam-hangzhou.oss-cn-hangzhou.aliyuncs.com/data/images/business/share-invite/share_mine.png?v=' + timestamp"
                                    mode="widthFix"
                                    class="invite-image"
                                ></image>
                            </view>
                        </navigator>
                    </view>
                </view>
            </view>
        </view>
    </view>
</template>

<script>
import https from '../../../utils/http';
import authService from '../../../utils/auth';
import toastService from '../../../utils/toast.service';
import dateHelper from '../../../utils/date-helper';
import systemStatus from '../../../utils/system-status';
//获取应用实例
const app = getApp();
export default {
    data() {
        return {
            userInfo: {},
            hasUserInfo: false,
            canIUse: uni.canIUse('button.open-type.getUserInfo'),
            couponsNum: 0,

            tabList: [
                {
                    modeId: 0,
                    modeName: '外卖订单'
                },
                {
                    modeId: 1,
                    modeName: '商城订单'
                }
            ],

            shopOrderTabList: [
                {
                    modeId: 1,
                    modeType: 'waitPayment',
                    modeName: '待付款',
                    icon: 'icondaifukuan'
                },
                {
                    modeId: 2,
                    modeType: 'waitReceived',
                    modeName: '待收货',
                    icon: 'icondaifahuo1'
                },
                {
                    modeId: 3,
                    modeType: 'waitPickedUp',
                    modeName: '待自提',
                    icon: 'icondaifahuo'
                },
                {
                    modeId: 4,
                    modeType: 'afterSales',
                    modeName: '退款/售后',
                    icon: 'icontuikuan'
                }
            ],

            pointsOrderTabList: [
                {
                    modeId: 1,
                    modeType: 'waitPayment',
                    modeName: '待付款',
                    icon: 'icondaifukuan'
                },
                {
                    modeId: 2,
                    modeType: 'waitDelivered',
                    modeName: '待发货',
                    icon: 'icondaifahuo1'
                },
                {
                    modeId: 3,
                    modeType: 'waitReceived',
                    modeName: '待收货',
                    icon: 'icondaifahuo'
                },
                {
                    modeId: 4,
                    modeType: 'afterSales',
                    modeName: '退款/售后',
                    icon: 'icontuikuan'
                }
            ],

            currentTab: 0,
            isVipImages: [],
            appVersion: app.globalData.appVersion,
            statusBarHeight: '',

            data: {
                headImg: false,
                username: '',
                vipNo: '',
                inviteRewardAmount: '',
                points: '',
                couponsNumber: false,
                balance: '',
                id: ''
            },

            timestamp: ''
        };
    }, // getCouponsSelectCounts() {
    //   https.request('/api-promotion/rest/member/couponsMemberRelation/selectCounts').then(result => {
    //     if (result.success) {
    //       this.getPointsCouponsSelectCounts(result.data)
    //     }
    //   })
    // },
    // getPointsCouponsSelectCounts(couponsNum) {
    //   https.request('/api-mall/rest/member/pointsMall/couponsMemberRelation/selectCounts').then(result => {
    //     if (result.success) {
    //       this.setData({
    //         couponsNum: result.data+couponsNum
    //       })
    //     }
    //   })
    // },
    onLoad: function () {
        this.setData({
            statusBarHeight: app.globalData.systemInfoSync.statusBarHeight * 2
        });
        if (app.globalData.userInfo) {
            console.log(1);
            this.setData({
                userInfo: app.globalData.userInfo,
                hasUserInfo: true
            });
        } else if (this.canIUse) {
            console.log(2);
            // 由于 getUserInfo 是网络请求，可能会在 Page.onLoad 之后才返回
            // 所以此处加入 callback 以防止这种情况
            app.globalData.userInfoReadyCallback = (res) => {
                this.setData({
                    userInfo: res.userInfo,
                    hasUserInfo: true
                });
            };
        } else {
            console.log(3);
            // 在没有 open-type=getUserInfo 版本的兼容处理
        }
    },
    onShow() {
        authService.checkIsLogin().then((result) => {
            console.log(result);
            if (result) {
                this.getUserInfo();
                // this.getCouponsSelectCounts();
                return;
            }
            this.setData({
                data: null,
                userInfo: null
            });
        });
        this.getTimestamp();
    },
    methods: {
        // 滑动切换tab
        bindSlideChange: function (e) {
            this.setData({
                currentTab: e.detail.current
            });
        },

        getTimestamp() {
            var timestamp = dateHelper.getTimestamp();
            console.log(timestamp);
            this.setData({
                timestamp: timestamp
            });
        },

        //点击切换
        clickTab: function (e) {
            console.log(e.target.dataset.current);
            if (this.currentTab === e.target.dataset.current) {
                return false;
            } else {
                // 显示加载图标
                this.setData({
                    currentTab: e.target.dataset.current
                });
            }
        },

        getUserProfile(e) {
            // 推荐使用wx.getUserProfile获取用户信息，开发者每次通过该接口获取用户个人信息均需用户确认
            // 开发者妥善保管用户快速填写的头像昵称，避免重复弹窗
            console.log(app.globalData.userInfo);
            if (app.globalData.userInfo) {
                uni.navigateTo({
                    url: '../../login/choose/choose'
                });
            } else {
                uni.getUserProfile({
                    desc: '用于完善会员资料',
                    // 声明获取用户个人信息后的用途，后续会展示在弹窗中，请谨慎填写
                    success: (res) => {
                        console.log(res);
                        if (res.userInfo) {
                            app.globalData.userInfo = res.userInfo;
                            this.setData({
                                userInfo: res.userInfo,
                                hasUserInfo: true
                            });
                            uni.navigateTo({
                                url: '../../login/choose/choose'
                            });
                        } else {
                        }
                    },
                    fail(res) {
                        console.log(res);
                        app.globalData.userInfo = null;
                        uni.navigateTo({
                            url: '../../login/choose/choose'
                        });
                    }
                });
            }
        },

        bindUserinfoTap() {
            uni.navigateTo({
                url: '../userinfo/userinfo'
            });
        },

        getUserInfo: function (e) {
            https.request('/api-user/rest/member/getLoginMemberInfo', {}).then((result) => {
                if (result.success) {
                    result.data.typeVipText = systemStatus.typeVipText(result.data.type);
                    result.data.statusVipText = systemStatus.statusVipText(result.data.vipStatus);
                    result.data.vipStartTime = dateHelper.formatDate(result.data.vipStartTime);
                    result.data.vipEndTime = dateHelper.formatDate(result.data.vipEndTime);
                    this.isVipImages = [];
                    if (result.data.type == 1) {
                        this.isVipImages.push('https://siam-hangzhou.oss-cn-hangzhou.aliyuncs.com/data/images/business/mine_page/user_1.png');
                        this.isVipImages.push('https://siam-hangzhou.oss-cn-hangzhou.aliyuncs.com/data/images/business/mine_page/user_2.png');
                        this.isVipImages.push('https://siam-hangzhou.oss-cn-hangzhou.aliyuncs.com/data/images/business/mine_page/user_3.png');
                    } else {
                        this.isVipImages.push('https://siam-hangzhou.oss-cn-hangzhou.aliyuncs.com/data/images/business/mine_page/vip_1.png');
                        this.isVipImages.push('https://siam-hangzhou.oss-cn-hangzhou.aliyuncs.com/data/images/business/mine_page/vip_2.png');
                        this.isVipImages.push('https://siam-hangzhou.oss-cn-hangzhou.aliyuncs.com/data/images/business/mine_page/vip_3.png');
                    }
                    this.setData({
                        data: result.data,
                        userInfo: this.userInfo,
                        isVipImages: this.isVipImages
                    });
                }
            });
        },

        getRequestSubscribeMessage() {
            console.log(111);
            let use = uni.canIUse('requestSubscribeMessage');
            console.log(use);
            if (typeof uni.requestSubscribeMessage === 'function') {
                console.log(true);
            }
            uni.requestSubscribeMessage({
                tmplIds: ['N63hQksq6Rp3XlrBySkligk-pSvvpJ5fCovwUkwHVH4', 'eF29ugG7ZKKKHCDH2Nk48Q2JCH1qKDLBMX2LnAzCz-w', 'CE3V7tzt4-PSsf8s-xZg731zHtDEkQmBOedcEEv3cx8'],
                success(res) {
                    console.log(res);
                },
                fail(error) {
                    console.log(error);
                }
            });
        },

        bindAccountBalance() {
            uni.navigateTo({
                url: '../balance/index/index'
            });
        },

        bindMemberCenter() {
            uni.navigateTo({
                url: '../member/index/index'
            });
        },

        bindInDevelopment() {
            app.globalData.bindInDevelopment();
        },

        bindOrderInfo(e) {
            uni.navigateTo({
                url: '../../order/index/index?currentTab=0&modeType=all&currentOrderTab=0'
            });
        }
    }
};
</script>
<style>
@import './index.css';
</style>
