<template>
    <view>
        <view class="lists-class" @tap="bindDetailInfo" :data-id="item.id" v-for="(item, index) in list" :key="index">
            <!-- <view class="orderNo">订单号：{{item.orderNo}}</view> -->

            <view class="list-top-class">
                <text>{{ item.denominationName }}</text>
                <view>
                    <text class="strike_through" v-if="item.denominationIsSale">￥{{ item.denominationSalePrice }}</text>
                    <text :decode="true" class="denominationSalePrice">&nbsp;&nbsp;￥{{ item.denominationPrice }}</text>
                </view>
            </view>

            <!-- <view class="list-top-class createTime">
      <text>会员充值</text>
      <text>{{item.statusText}}</text>
    </view> -->

            <view class="list-top-class">
                <text class="createTime">{{ item.createTime }}</text>
            </view>

            <!-- <view>
      <view wx:if="{{item.denominationIsGiveBalance}}">赠送余额：￥{{item.denominationGiveBalance}}</view>
      <view wx:if="{{item.denominationIsGiveCoupons}}">{{item.denominationGiveCouponsDescription}}</view>
    </view> -->
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
import isShowTip from '../../../../components/is-show-tip/is-show-tip';
import https from '../../../../utils/http';
import authService from '../../../../utils/auth';
import toastService from '../../../../utils/toast.service';
import dateHelper from '../../../../utils/date-helper';
import systemStatus from '../../../../utils/system-status';
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
        this.getVipRechargeRecordList();
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
        getVipRechargeRecordList() {
            toastService.showLoading('正在加载...');
            https
                .request('/api-user/rest/member/vipRechargeRecord/list', {
                    pageNo: pageNo,
                    pageSize: pageSize
                })
                .then((result) => {
                    toastService.hideLoading();
                    if (result.success) {
                        result.data.records.forEach((item, index) => {
                            item.createTime = dateHelper.fmtDate(item.createTime);
                            item.statusText = systemStatus.payStatusText(item.status);
                            this.list.push(item);
                        });
                        this.setData({
                            isLastPage: result.data.isLastPage,
                            list: this.list
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
@import './record.css';
</style>
