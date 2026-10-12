<template>
    <view class="page-content">
        <view class="member-center">
            <view class="member-vip-info">
                <view class="vip-bg-view">
                    <view class="flex_between">
                        <view class="member-info">
                            <image :src="data.headImg" mode="widthFix" class="headImg"></image>
                            <view class="nickname-vipEndTime">
                                <view class="nickname flex_start">
                                    <view class="out_of_range one_row">{{ data.username }}</view>
                                    <view class="vip-image-view">
                                        <image :src="item + '?v=' + appVersion" mode="widthFix" class="is-vip-image" v-for="(item, index) in isVipImages" :key="index"></image>
                                    </view>
                                </view>
                                <view class="vip-recharge flex_between">
                                    <!-- <view class="vipEndTime {{data.type==1?'ptyh-class':'cjhy-class'}}">
                  {{data.typeVipText}}
                </view> -->

                                    <view class="renew-botton" v-if="data.type == 1" @tap="bindVipRecharge">加入会员</view>
                                    <view class="renew-botton" v-else @tap="bindVipRecharge">去续费</view>
                                </view>
                                <view class="no-vipno flex_start">ID：{{ data.vipNo }}</view>
                            </view>
                        </view>
                        <view @tap="bindReward" class="wodejl-view">佣金明细></view>
                    </view>
                    <view>
                        <!-- <view class="vip-recharge-view flex_between">
            <view class="renew-botton" wx:if="{{data.vipStatus!=3&&data.vipStatus!=2}}" bindtap="bindVipRecharge">会员充值
            </view>
            <view class="renew-botton" wx:if="{{data.vipStatus==3}}">立即续费</view>
            <text class="iconfont iconhuiyuan1"></text>
          </view>-->
                        <view class="mine-blocking-view">
                            <view class="mine-blocking">
                                <navigator class="mine-navigator" url="../../balance/index/index">
                                    <view class="number-tip theme-color">{{ data.balance }}</view>
                                    <view class="text-tip">余额</view>
                                </navigator>
                                <navigator class="mine-navigator" url="../../integral/integral">
                                    <view class="number-tip theme-color">{{ data.points }}</view>
                                    <view class="text-tip">我的积分</view>
                                </navigator>
                                <navigator class="mine-navigator" url="../../share/reward/reward">
                                    <view class="number-tip theme-color">{{ data.inviteRewardAmount }}</view>
                                    <view class="text-tip">佣金</view>
                                </navigator>
                                <navigator class="mine-navigator" url="../../coupons/coupons">
                                    <view class="number-tip theme-color">{{ data.couponsNumber ? data.couponsNumber : 0 }}</view>
                                    <view class="text-tip">卡券</view>
                                </navigator>
                            </view>
                        </view>
                    </view>
                </view>
            </view>
        </view>
        <view class="add_buttom">
            <view class="add-vip" @tap="bindVipRecharge">加入会员尊享4大权益</view>
            <!-- <image src="https://siam-hangzhou.oss-cn-hangzhou.aliyuncs.com/data/images/business/vip_page/benefits_icon.png"
								mode="aspectFill" class="vip-button-image"></image> -->
        </view>

        <view class="vip-details-view">
            <view class="vip-detail">
                <!-- <text class="iconfont icondingdan-"></text> -->
                <image
                    :src="'https://siam-hangzhou.oss-cn-hangzhou.aliyuncs.com/data/images/business/vip_page/guarantee.png?v=' + appVersion"
                    mode="aspectFill"
                    class="invite-image"
                ></image>
                <view class="iconfont-title">账户保障</view>
            </view>
            <view class="vip-detail">
                <!-- <text class="iconfont iconyue"></text> -->
                <image
                    :src="'https://siam-hangzhou.oss-cn-hangzhou.aliyuncs.com/data/images/business/vip_page/wallet.png?v=' + appVersion"
                    mode="aspectFill"
                    class="invite-image"
                ></image>
                <view class="iconfont-title">独家礼券</view>
            </view>
            <view class="vip-detail">
                <!-- <text class="iconfont iconyouhuiquan"></text> -->
                <image
                    :src="'https://siam-hangzhou.oss-cn-hangzhou.aliyuncs.com/data/images/business/vip_page/trophy.png?v=' + appVersion"
                    mode="aspectFill"
                    class="invite-image"
                ></image>
                <view class="iconfont-title">下单奖励</view>
            </view>
            <view class="vip-detail">
                <!-- <text class="iconfont iconyouhuiquan"></text> -->
                <image
                    :src="'https://siam-hangzhou.oss-cn-hangzhou.aliyuncs.com/data/images/business/vip_page/share.png?v=' + appVersion"
                    mode="aspectFill"
                    class="invite-image"
                ></image>
                <view class="iconfont-title">邀请奖励</view>
            </view>
        </view>

        <view class="gift-view">
            <view class="view-line"></view>
            <view class="gift-view-info">
                <image
                    :src="'https://siam-hangzhou.oss-cn-hangzhou.aliyuncs.com/data/images/business/vip_page/recharge.png?v=' + appVersion"
                    mode="widthFix"
                    class="gift-image"
                    @tap="bindVipRecharge"
                ></image>
            </view>
            <view class="gift-content">
                <view>
                    <view class="mrwcyqrw">每日完成邀请任务</view>
                    <view class="stqdsh">送TA券的时候你还有现金奖励</view>
                </view>
                <view class="goto-finish" @tap="bindInvite">去完成</view>
            </view>
            <view class="view-line"></view>
            <view class="vip-rules">
                <text class="vip-text">{{ current.vipRule }}</text>
            </view>
        </view>
    </view>
</template>

<script>
import https from '../../../../utils/http';
import authService from '../../../../utils/auth';
import toastService from '../../../../utils/toast.service';
import dateHelper from '../../../../utils/date-helper';
import systemStatus from '../../../../utils/system-status';
const app = getApp();
export default {
    data() {
        return {
            isVipImages: [],
            appVersion: app.globalData.appVersion,

            data: {
                headImg: '',
                username: '',
                type: 0,
                vipNo: '',
                balance: '',
                points: '',
                inviteRewardAmount: '',
                couponsNumber: false
            },

            userInfo: '',
            timestamp: '',

            current: {
                vipRule: ''
            }
        };
    }
    /**
     * 生命周期函数--监听页面加载
     */,
    onLoad: function (options) {
        this.selectCurrent();
    },
    /**
     * 生命周期函数--监听页面初次渲染完成
     */
    onReady: function () {},
    /**
     * 生命周期函数--监听页面显示
     */
    onShow: function () {
        authService.checkIsLogin().then((result) => {
            if (result) {
                this.getUserInfo();
                return;
            }
            this.setData({
                data: null,
                userInfo: null
            });
        });
    },
    /**
     * 生命周期函数--监听页面隐藏
     */
    onHide: function () {},
    /**
     * 生命周期函数--监听页面卸载
     */
    onUnload: function () {},
    /**
     * 页面相关事件处理函数--监听用户下拉动作
     */
    onPullDownRefresh: function () {},
    /**
     * 页面上拉触底事件的处理函数
     */
    onReachBottom: function () {},
    /**
     * 用户点击右上角分享
     */
    onShareAppMessage: function () {},
    methods: {
        getTimestamp() {
            var timestamp = dateHelper.getTimestamp();
            console.log(timestamp);
            this.setData({
                timestamp: timestamp
            });
        },

        selectCurrent() {
            https.request('/api-util/rest/setting/selectCurrent', {}).then((result) => {
                if (result.success) {
                    result.data.vipRule = result.data.vipRule.replace('↵', '\n');
                    this.setData({
                        current: result.data
                    });
                }
            });
        },

        bindInvite() {
            uni.navigateTo({
                url: `../../share/index/index?inviterId=${this.data.id}`
            });
        },

        getUserInfo: function (e) {
            https.request('/api-user/rest/member/getLoginMemberInfo', {}).then((result) => {
                if (result.success) {
                    result.data.typeVipText = systemStatus.typeVipText(result.data.type);
                    if (this.isVipImages.length <= 0) {
                        if (result.data.type == 1) {
                            this.isVipImages.push('https://siam-hangzhou.oss-cn-hangzhou.aliyuncs.com/data/images/business/mine_page/user_1.png');
                            this.isVipImages.push('https://siam-hangzhou.oss-cn-hangzhou.aliyuncs.com/data/images/business/mine_page/user_2.png');
                            this.isVipImages.push('https://siam-hangzhou.oss-cn-hangzhou.aliyuncs.com/data/images/business/mine_page/user_3.png');
                        } else {
                            this.isVipImages.push('https://siam-hangzhou.oss-cn-hangzhou.aliyuncs.com/data/images/business/mine_page/vip_1.png');
                            this.isVipImages.push('https://siam-hangzhou.oss-cn-hangzhou.aliyuncs.com/data/images/business/mine_page/vip_2.png');
                            this.isVipImages.push('https://siam-hangzhou.oss-cn-hangzhou.aliyuncs.com/data/images/business/mine_page/vip_3.png');
                        }
                    }
                    result.data.isVip = result.data.type == 1 ? false : true;
                    this.setData({
                        data: result.data,
                        isVipImages: this.isVipImages
                    });
                }
            });
        },

        bindReward() {
            uni.navigateTo({
                url: '../../../mine/share/reward/reward'
            });
        },

        bindOrders() {
            uni.navigateTo({
                url: '../record/record'
            });
        },

        bindBalance() {
            uni.navigateTo({
                url: '../../balance/index/index'
            });
        },

        bindCoupons() {
            uni.navigateTo({
                url: '../../../mine/coupons/coupons'
            });
        },

        bindVipRecharge() {
            uni.navigateTo({
                url: '../recharge/recharge'
            });
        }
    }
};
</script>
<style>
@import './index.css';
</style>
