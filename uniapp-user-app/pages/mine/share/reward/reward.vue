<template>
    <view>
        <view class="balance-views theme-bg">
            <view class="balance-text">
                当前奖励金额（元）
                <text class="iconfont iconwenhao" @tap="selectCurrent"></text>
            </view>
            <view class="balance-view">
                {{ userInfo.inviteRewardAmount }}
            </view>
            <view v-if="userInfo.unreceivedInviteRewardAmount > 0" @tap="bindUnreceivedInviteReward">未到账余额：{{ userInfo.unreceivedInviteRewardAmount }} 元</view>
            <view class="balance-detail">
                <view class="balance-detail-info">
                    <view class="balance-quota">￥{{ userInfo.yesterdayGainInviteRewardAmount }}</view>
                    <view class="balance-title">昨日收益</view>
                </view>
                <view class="balance-detail-info">
                    <button size="mini" type="primary" class="withdrawal-button" @tap="bindImmediatelyWithdrawal">立即提现</button>
                </view>
                <view class="balance-detail-info">
                    <view class="balance-quota">￥{{ userInfo.totalWithdrawInviteRewardAmount }}</view>
                    <view class="balance-title">累计已提</view>
                </view>
            </view>
        </view>
        <swiper :current="currentTab" class="swiper-box" duration="300" :style="'height:' + (winHeight - 10) + 'px'" @change="bindSlideChange">
            <swiper-item class="swiper-items">
                <scroll-view @scrolltoupper="onPullDownRefresh" upper-threshold="-50" @scrolltolower="onReachBottom" lower-threshold="10" scroll-y class="scroll-views">
                    <view class="integral-items" @tap="openSpecifications" :data-id="item.id" v-for="(item, index) in list" :key="index">
                        <view class="integral-item">
                            <view class="integral-item-name">
                                <text class="out_of_range one_row">{{ item.message }}</text>
                            </view>
                            <view class="integral-item-time">
                                <text>{{ item.createTime }}</text>
                            </view>
                        </view>

                        <view :class="'integral-item-number ' + (item.operateType == 1 ? 'theme-color' : 'color-integral')">
                            {{ item.operateType == 1 ? '+' : '-' }}{{ item.number }}
                        </view>
                    </view>
                    <is-show-tip :isShow="list.length <= 0" :top="25" :bottom="0" :text="list.length <= 0" tipText="暂无奖励" :button="button"></is-show-tip>
                </scroll-view>
            </swiper-item>
        </swiper>
        <mp-halfScreenDialog :show="specificationsDialog" extClass="extClassSpecifications">
            <view slot="title">账单详情</view>
            <view slot="desc">
                <view class="order-info-view">
                    <view class="order-title">账单说明</view>
                    <view class="order-info">{{ recordInfo.message }}</view>
                    <view class="order-title">账单金额</view>
                    <view class="order-info">{{ recordInfo.operateType == 1 ? '+' : '-' }}{{ recordInfo.number }}元</view>
                    <view class="order-title">支付时间</view>
                    <view class="order-info">{{ recordInfo.createTime }}</view>
                    <view class="order-title" v-if="recordInfo.serviceFee">服务费</view>
                    <view class="order-info" v-if="recordInfo.serviceFee">{{ recordInfo.serviceFee }}元</view>
                </view>
            </view>
        </mp-halfScreenDialog>
        <mp-halfScreenDialog :show="selectCurrentDialog" extClass="extClassSelectCurrent">
            <view slot="title">奖励规则</view>
            <view slot="desc">
                <view class="order-info-view">
                    <scroll-view style="height: 55vh" scroll-y>
                        <text class="order-title">{{ current.commissionRule }}</text>
                    </scroll-view>
                </view>
            </view>
        </mp-halfScreenDialog>
    </view>
</template>

<script>
import mpHalfScreenDialog from '../../../../components/half-screen-dialog/half-screen-dialog';
import isShowTip from '../../../../components/is-show-tip/is-show-tip';
import https from '../../../../utils/http';
import authService from '../../../../utils/auth';
import toastService from '../../../../utils/toast.service';
import dateHelper from '../../../../utils/date-helper';
import systemStatus from '../../../../utils/system-status';
var pageNo = 1;
var pageSize = 10;
export default {
    components: {
        mpHalfScreenDialog,
        isShowTip
    },
    data() {
        return {
            specificationsDialog: false,
            selectCurrentDialog: false,
            list: [],

            current: {
                commissionRule: ''
            },

            winHeight: '',

            recordInfo: {
                message: '',
                operateType: 0,
                number: '',
                createTime: '',
                serviceFee: ''
            },

            isLastPage: '',

            userInfo: {
                inviteRewardAmount: '',
                unreceivedInviteRewardAmount: 0,
                yesterdayGainInviteRewardAmount: '',
                totalWithdrawInviteRewardAmount: ''
            },

            currentTab: '',
            button: ''
        };
    },
    /**
     * 生命周期函数--监听页面加载
     */
    onLoad: function (options) {
        this.getHeight();
    },
    /**
     * 生命周期函数--监听页面初次渲染完成
     */
    onReady: function () {},
    /**
     * 生命周期函数--监听页面显示
     */
    onShow: function () {
        this.getUserInfo();
        pageNo = 1;
        this.setData({
            list: []
        });
        this.getMemberBillingRecord();
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
    onReachBottom: function () {
        if (this.isLastPage) {
            toastService.showToast('没有更多啦~');
            return;
        }
        pageNo = pageNo + 1;
        this.getVipRechargeRecordList();
    },
    /**
     * 用户点击右上角分享
     */
    onShareAppMessage: function () {},
    methods: {
        selectCurrent() {
            if (this.current) {
                this.setData({
                    selectCurrentDialog: true
                });
                return;
            }
            https.request('/api-util/rest/setting/selectCurrent', {}).then((result) => {
                if (result.success) {
                    result.data.commissionRule = result.data.commissionRule.replace('↵', '\n');
                    this.setData({
                        current: result.data,
                        selectCurrentDialog: true
                    });
                }
            });
        },

        getHeight() {
            //获取用户手机系统信息
            var that = this;
            uni.getSystemInfo({
                success: function (res) {
                    let winHeight = res.windowHeight; //获取高度
                    //获取class为settlement-view并减去这个高度，自适应scrollview的高度
                    const query = uni.createSelectorQuery();
                    query.select('.balance-views').boundingClientRect();
                    query.selectViewport().scrollOffset();
                    query.exec(function (res) {
                        that.setData({
                            winHeight: winHeight - res[0].height
                        });
                    });
                }
            });
        },

        openSpecifications(e) {
            console.log(e);
            this.setData({
                specificationsDialog: true
            });
            this.recordSelectById(e.currentTarget.dataset.id);
        },

        recordSelectById(id) {
            https
                .request('/api-user/rest/member/billingRecord/selectById', {
                    id: id
                })
                .then((result) => {
                    toastService.hideLoading();
                    if (result.success) {
                        result.data.createTime = dateHelper.fmtDate(result.data.createTime);
                        this.setData({
                            recordInfo: result.data
                        });
                    }
                });
        },

        getMemberBillingRecord() {
            toastService.showLoading('正在加载...');
            https
                .request('/api-user/rest/member/billingRecord/list', {
                    pageNo: pageNo,
                    pageSize: pageSize,
                    coinType: 3
                })
                .then((result) => {
                    toastService.hideLoading();
                    if (result.success) {
                        result.data.records.forEach((item, index) => {
                            item.createTime = dateHelper.fmtDate(item.createTime);
                            item.coinTypeText = systemStatus.withDrawalCoinTypeText(item.coinType);
                            this.list.push(item);
                        });
                        this.setData({
                            isLastPage: result.data.isLastPage,
                            list: this.list
                        });
                    }
                });
        },

        getUserInfo: function (e) {
            https.request('/api-user/rest/member/getLoginMemberInfo', {}).then((result) => {
                if (result.success) {
                    this.setData({
                        userInfo: result.data
                    });
                }
            });
        },

        bindImmediatelyWithdrawal(e) {
            uni.navigateTo({
                url: '../withdrawal/index/index'
            });
        },

        bindUnreceivedInviteReward(e) {
            uni.navigateTo({
                url: '../../unreceived/reward/list/list'
            });
        },

        bindSlideChange() {
            console.log('占位：函数 bindSlideChange 未声明');
        },

        onPullDownRefresh() {
            console.log('占位：函数 onPullDownRefresh 未声明');
        },

        onReachBottom() {
            console.log('占位：函数 onReachBottom 未声明');
        }
    }
};
</script>
<style>
@import './reward.css';
</style>
