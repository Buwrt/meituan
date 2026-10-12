<template>
    <view>
        <view class="lists-class" :data-id="item.id" v-for="(item, index) in list" :key="index">
            <view class="list-top-class">
                <text>{{ item.message }}</text>
                <view :class="'integral-item-number ' + (item.operateType == 1 ? 'pay-success' : 'pay-fail')">
                    <text :decode="true" class="denominationSalePrice">&nbsp;&nbsp;&nbsp;{{ item.operateType == 1 ? '+' : '-' }}{{ item.number }}</text>
                </view>
            </view>

            <view class="list-top-class">
                <text class="createTime">{{ item.createTime }}</text>
            </view>
        </view>
        <is-show-tip
            :isShow="list.length <= 0"
            @bindTap="goToDrink"
            buttonText="去兑换"
            :top="25"
            :bottom="0"
            :text="list.length <= 0"
            tipText="暂无充值记录"
            :button="button"
        ></is-show-tip>
    </view>
</template>

<script>
import isShowTip from '../../../../../components/is-show-tip/is-show-tip';
import https from '../../../../../utils/http';
import authService from '../../../../../utils/auth';
import toastService from '../../../../../utils/toast.service';
import dateHelper from '../../../../../utils/date-helper';
import systemStatus from '../../../../../utils/system-status';
var pageNo = 1;
var pageSize = 20;
export default {
    components: {
        isShowTip
    },
    data() {
        return {
            list: [],
            isLastPage: '',
            button: ''
        };
    },
    /**
     * 生命周期函数--监听页面加载
     */
    onLoad: function (options) {
        this.getMemberBillingRecordList();
    },
    /**
     * 生命周期函数--监听页面初次渲染完成
     */
    onReady: function () {},
    /**
     * 生命周期函数--监听页面显示
     */
    onShow: function () {},
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
        getMemberBillingRecordList() {
            toastService.showLoading('正在加载...');
            https
                .request('/api-user/rest/member/billingRecord/list', {
                    pageNo: pageNo,
                    pageSize: pageSize,
                    coinType: 4,
                    isSettled: 0
                })
                .then((result) => {
                    toastService.hideLoading();
                    if (result.success) {
                        result.data.records.forEach((item, index) => {
                            item.createTime = dateHelper.fmtDate(item.createTime);
                        });
                        this.setData({
                            isLastPage: result.data.isLastPage,
                            list: result.data.records
                        });
                    }
                });
        },

        bindDetailInfo(e) {
            console.log(e);
            let id = e.currentTarget.dataset.id;
            uni.navigateTo({
                url: '../detail/detail?id=' + id
            });
        },

        goToDrink() {
            console.log('占位：函数 goToDrink 未声明');
        }
    }
};
</script>
<style>
@import './list.css';
</style>
