<template>
    <view class="log">
        <view class="log_top">
            <view class="log_top_title">
                快递公司：
                <text class="title_value">{{ logisticsJson.logisticsWay }}</text>
            </view>
            <view class="log_top_title">
                运单编号：
                <text class="title_value">{{ logisticsJson.logisticsNo }}</text>
            </view>
            <view class="log_top_title">
                物流状态：
                <text class="title_value">{{ logisticsJson.logisticsStatusText }}</text>
            </view>
        </view>
        <view class="log_top flex_between">
            <view class="log_top_title">快递员：{{ logisticsJson.courierName }}</view>
            <button size="mini" class="pinglun-btn theme-color-border" @tap="contactCourier" style="font-size: 22rpx">联系快递员</button>
        </view>
        <view class="log_content" v-if="list.length > 0">
            <view class="log_content_box">
                <view :class="'log_content_box_add ' + (index + 1 != list.length ? 'is_index_end' : 'not_index_end')" v-for="(item, index) in list" :key="index">
                    <view :class="'spot ' + (index === 0 ? 'default' : '') + ' flex_center'">{{ list.length - index }}</view>

                    <view :class="'address ' + (index === 0 ? 'default' : '')">{{ item.description }}</view>

                    <view :class="'time ' + (index === 0 ? 'default' : '')">{{ item.descriptionTime }}</view>
                </view>
            </view>
        </view>
        <view v-else class="kuaidi_wu">暂无物流信息</view>
    </view>
</template>

<script>
import https from '../../../../../utils/http';
import authService from '../../../../../utils/auth';
import toastService from '../../../../../utils/toast.service';
import dateHelper from '../../../../../utils/date-helper';
import systemStatus from '../../../../../utils/system-status';
import GlobalConfig from '../../../../../utils/global-config';
import util from '../../../../../utils/util';
const app = getApp();
let logisticsJson = {};
export default {
    data() {
        return {
            list: [],

            logisticsJson: {
                logisticsWay: '',
                logisticsNo: '',
                logisticsStatusText: '',
                courierName: ''
            }
        };
    }
    /**
     * 生命周期函数--监听页面加载
     */,
    onLoad: function (options) {
        console.log(options.logisticsJson);
        logisticsJson = JSON.parse(options.logisticsJson);
        logisticsJson.logisticsStatusText = systemStatus.logisticsStatusText(logisticsJson.deliveryStatus);
        this.getOrderDetail(logisticsJson.orderId);
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
    onReachBottom: function () {},
    /**
     * 用户点击右上角分享
     */
    onShareAppMessage: function () {},
    methods: {
        getOrderDetail(id) {
            toastService.showLoading();
            https
                .request('/api-mall/rest/member/pointsMall/orderLogistics/list', {
                    orderId: id,
                    pageNo: -1,
                    pageSize: 20
                })
                .then((result) => {
                    toastService.hideLoading();
                    if (result.success) {
                        result.data.records.forEach((item, index) => {
                            item.descriptionTime = dateHelper.formatDate(item.descriptionTime);
                        });
                        this.setData({
                            list: result.data.records,
                            logisticsJson: logisticsJson
                        });
                    }
                });
        },

        contactCourier() {
            uni.makePhoneCall({
                phoneNumber: this.logisticsJson.courierPhone
            });
        }
    }
};
</script>
<style>
@import './logistics.css';
</style>
