<template>
    <view>
        <view class="swiper-tab self-adaption" v-if="pageType != 'orderDetail'">
            <view class="swiper-tab-item" :data-current="index" @tap="clickTab" hover-class="hover-click-class" v-for="(item, index) in modeList" :key="index">
                <view :class="'swiper_table_item_view ' + (currentTab == item.modeId ? 'active' : '')" :data-current="index" @tap="clickTab">
                    {{ item.modeName }}
                </view>
            </view>
        </view>
        <swiper :current="currentTab" class="swiper-box" duration="300" :style="'height:' + (winHeight - 5) + 'px'" @change="bindSlideChange">
            <swiper-item class="swiper-items">
                <scroll-view
                    @scrolltoupper="onPullDownRefresh"
                    upper-threshold="-50"
                    v-if="addressList.length > 0"
                    @scrolltolower="onReachBottom"
                    lower-threshold="0"
                    :scroll-x="false"
                    :scroll-y="true"
                    class="scroll-views"
                >
                    <view class="delivery-address-list">
                        <view class="my-address">我的地址</view>
                        <view class="delivery-address-item" hover-class="hover-class-public" v-for="(item, index) in addressList" :key="index">
                            <text class="delivery-address-choice theme-color" v-if="chooseId == item.id">✓</text>

                            <view
                                class="address-items"
                                @tap="parseEventDynamicCode($event, pageType == 'orderDetail' ? 'confirmChoiceDelivery' : 'confirmChoice')"
                                :data-index="index"
                            >
                                <view class="address-name">
                                    <text class="address-name-text out_of_range one_row">{{ item.province }}{{ item.city }}{{ item.area }}{{ item.street }}</text>
                                </view>
                                <view class="personal-info">
                                    <view class="default-address" v-if="item.isDefault">默认地址</view>
                                    <!-- <view class="address-tag theme-bg">家</view> -->
                                    <view class="phone-number">{{ item.phone }}</view>
                                    <view class="username-class">{{ item.realname }}</view>
                                </view>
                            </view>

                            <text class="iconfont iconbianji-copy" @tap="editAddressTap" :data-data="item" :data-key="index"></text>
                        </view>
                    </view>
                </scroll-view>
                <is-show-tip
                    :isShow="addressList.length <= 0"
                    @bindTap="goToDrink"
                    buttonText="去喝一杯"
                    :top="45"
                    :bottom="10"
                    :text="addressList.length <= 0"
                    tipText="您还没有地址信息"
                    :button="addressList.length > 0"
                ></is-show-tip>
            </swiper-item>

            <swiper-item class="swiper-items" v-if="pageType != 'orderDetail'">
                <scroll-view @scrolltoupper="onPullDownRefresh" upper-threshold="-50" @scrolltolower="onReachBottom" lower-threshold="0" scroll-y class="scroll-views">
                    <view class="self-taking-list">
                        <view class="self-taking-item" @tap="confirmShopChoice" hover-class="hover-class-public" :data-index="index">
                            <view class="store-name out_of_range one_row">{{ shopAddress.name }}</view>
                            <view class="business-hours-distance">
                                <view class="business-hours">
                                    <text class="iconfont iconshijian"></text>
                                    <view>
                                        {{ shopAddress.startTime }}-{{ shopAddress.endTime }}
                                        <text class="isNotData" v-if="!item.isBusiness || !item.isOperating">{{ shopAddress.isBusinessText }}</text>
                                    </view>
                                </view>
                                <!-- <view class="distance-class">距您226m</view> -->
                            </view>
                            <view class="store-address-details">
                                <view class="store-address">
                                    <text class="iconfont icondizhi"></text>
                                    <view class="out_of_range one_row">{{ shopAddress.province }}{{ shopAddress.city }}{{ shopAddress.area }}{{ shopAddress.street }}</view>
                                </view>
                            </view>
                        </view>
                    </view>
                </scroll-view>
            </swiper-item>
        </swiper>
        <navigator url="../insert/insert" class="insert-address-view theme-color self-adaption" v-if="currentTab == 0 || pageType == 'orderDetail'">+新增地址</navigator>
    </view>
</template>

<script>
import isShowTip from '../../../components/is-show-tip/is-show-tip';
import https from '../../../utils/http';
import dateHelper from '../../../utils/date-helper';
import toastService from '../../../utils/toast.service';
import utilsHelper from '../../../utils/util';
import authService from '../../../utils/auth';
//获取应用实例
const app = getApp();
var heightList = [];
export default {
    components: {
        isShowTip
    },
    data() {
        return {
            addressList: [],
            shopAddressList: [],

            modeList: [
                {
                    modeName: '外卖配送',
                    modeId: 0
                }
            ],

            isInsert: false,
            reducedDeliveryPrice: '',
            currentTab: '',
            shopId: '',
            pageType: '',
            chooseId: '',
            orderNo: '',
            orderId: '',
            isOperation: false,
            deliveryAndSelfTaking: '',

            shopAddress: {
                name: '',
                isBusinessText: '',
                province: '',
                city: '',
                area: '',
                street: ''
            },

            data: '',
            oldAddressId: '',
            time: '',
            isPreviousPage: false,
            isJs: false,
            refreshKey: '',
            winHeight: ''
        };
    },
    /**
     * 生命周期函数--监听页面加载
     */
    onLoad: function (options) {
        console.log(options);
        let deliveryAndSelfTaking = JSON.parse(options.deliveryData);
        console.log(deliveryAndSelfTaking);
        this.setData({
            reducedDeliveryPrice: options.reducedDeliveryPrice,
            currentTab: options.radioIndex,
            shopId: options.shopId,
            pageType: options.pageType,
            chooseId: deliveryAndSelfTaking.deliveryAddress ? deliveryAndSelfTaking.deliveryAddress.id : '',
            orderNo: options.pageType == 'orderDetail' ? options.orderNo : '',
            orderId: options.pageType == 'orderDetail' ? options.orderId : ''
        });
        this.getDeliveryAddressList();
        this.getShopAddressList();
    },
    /**
     * 生命周期函数--监听页面初次渲染完成
     */
    onReady: function () {
        //自适应手机高度
        this.selfAdaption();
    },
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
        // // 显示加载图标
        // toastService.showLoading('正在加载...');
        // if (this.data.currentTab == 1) {
        //   // 页数+1
        //   pageNo = pageNo + 1;
        //   this.getDeliveryAddressList();
        //   return
        // }
        // shopPageNo = shopPageNo + 1;
        // this.getShopAddressList();
    },
    /**
     * 用户点击右上角分享
     */
    onShareAppMessage: function () {},
    methods: {
        // 滑动切换tab
        bindSlideChange: function (e) {
            var that = this;
            that.setData({
                currentTab: e.detail.current
            });
            //自适应手机高度
            that.selfAdaption();
        },

        //点击切换
        clickTab: function (e) {
            var that = this;
            if (that.currentTab === e.target.dataset.current) {
                return false;
            } else {
                that.setData({
                    currentTab: e.target.dataset.current
                });
                //自适应手机高度
                that.selfAdaption();
            }
        },

        getHasShopDelivery(deliveryAddressId) {
            return new Promise((fulfil, reject) => {
                https
                    .request('/common/selectHasShopDelivery', {
                        deliveryAddressId: deliveryAddressId
                    })
                    .then((result) => {
                        if (result.success) {
                            fulfil(result.data);
                        }
                    });
            });
        },

        // 订单详情我要配送方法
        confirmChoiceDelivery(e) {
            var key = e.currentTarget.dataset.index;
            //直接调用上一个页面的 setData() 方法，把数据存到上一个页面中去
            var shopAddress = this.addressList[key];

            // this.getHasShopDelivery(shopAddress.id).then(result => {
            //   if (!result) {
            //     toastService.showToast("当前暂时没有门店可以进行配送");
            //     return
            //   }
            https
                .request('/api-order/rest/common/selectDeliveryFeee', {
                    deliveryAddressId: shopAddress.id,
                    shopId: this.shopId
                })
                .then((result) => {
                    if (result.success) {
                        var that = this;
                        var pages = getCurrentPages();
                        var prevPage = pages[pages.length - 2]; //上一个页面
                        var prevToPage = pages[pages.length - 3]; //上一个页面
                        var payPrice = result.data - Number(this.reducedDeliveryPrice);
                        if (payPrice > 0) {
                            toastService.showModal(
                                '',
                                '确认支付' + payPrice + '元配送费吗？',
                                function () {
                                    authService.getOpenId().then((openId) => {
                                        //console.log(that.data.orderId)
                                        if (openId) {
                                            toastService.showLoading();
                                            https
                                                .request('/api-order/rest/member/wxPay/toPay4Applet', {
                                                    openid: openId,
                                                    type: 3,
                                                    out_trade_no: that.orderNo,
                                                    total_fee: payPrice,
                                                    deliveryAddressId: shopAddress.id,
                                                    shopId: that.shopId
                                                })
                                                .then((result) => {
                                                    toastService.hideLoading();
                                                    if (result.success) {
                                                        uni.requestPayment({
                                                            appId: result.data.appid,
                                                            timeStamp: result.data.timeStamp,
                                                            nonceStr: result.data.nonceStr,
                                                            package: result.data.package,
                                                            signType: 'MD5',
                                                            paySign: result.data.paySign,
                                                            success(res) {
                                                                toastService.showSuccess('支付成功', true);
                                                                let timeout = setTimeout(() => {
                                                                    prevPage.getOrderDetail(that.orderId);
                                                                    prevToPage.setData({
                                                                        isOperation: true
                                                                    });
                                                                    clearTimeout(timeout);
                                                                    uni.navigateBack(1);
                                                                }, 1000);
                                                            },
                                                            fail(res) {
                                                                toastService.showError('支付失败', true);
                                                            }
                                                        });
                                                    }
                                                });
                                            return;
                                        }
                                        toastService.showToast('登录用户错误，请重新登录');
                                    });
                                },
                                null
                            );
                        }
                    }
                });
            // })
        },

        //选择用户地址
        confirmChoice(e) {
            //console.log(e)
            var key = e.currentTarget.dataset.index;
            //直接调用上一个页面的 setData() 方法，把数据存到上一个页面中去
            var shopAddress = this.addressList[key];
            // this.getHasShopDelivery(shopAddress.id).then(result => {
            //   if (!result) {
            //     toastService.showToast("当前暂时没有门店可以进行配送");
            //     return
            //   }
            //直接调用上一个页面的 setData() 方法，把数据存到上一个页面中去
            for (let index in app.globalData.deliveryAndSelfTaking.modeList) {
                app.globalData.deliveryAndSelfTaking.modeList[index].checked = false;
                if (index == this.currentTab) {
                    app.globalData.deliveryAndSelfTaking.modeList[index].checked = true;
                }
            }
            // console.log(app.deliveryAndSelfTaking)
            if (this.pageType == 'pay') {
                let feeData = app.globalData.deliveryAndSelfTaking.reducedDeliveryTotalPrice ? app.globalData.deliveryAndSelfTaking.reducedDeliveryTotalPrice : 0;
                let pages = getCurrentPages();
                let prevPage = pages[pages.length - 2]; //上一个页面
                console.log(prevPage);
                //如果重复选择地址则不进行任何操作
                if (this.chooseId == shopAddress.id) {
                    return;
                }
                //选择地址和更换地址进行配送费的加减操作
                https
                    .request('/api-order/rest/common/selectDeliveryFee', {
                        deliveryAddressId: shopAddress.id,
                        shopId: this.shopId
                    })
                    .then((result) => {
                        if (result.success) {
                            //如果选择了不同的地址就进行减去上一次选择的地址的配送费，加上本次选择的地址的配送费（总价一起操作）
                            //prevPage.data.data.actualPrice = prevPage.data.data.actualPrice - (feeData) + result.data;
                            let priceFee = 0;
                            console.log('获取配送金额====>' + feeData);
                            console.log('获取选择的配送金额====>' + result.data);
                            if (feeData > result.data) {
                                priceFee = feeData - result.data;
                                prevPage.data.data.actualPrice = prevPage.data.data.actualPrice - priceFee;
                            } else {
                                priceFee = result.data - feeData;
                                prevPage.data.data.actualPrice = prevPage.data.data.actualPrice + priceFee;
                            }
                            prevPage.data.data.fullPriceReduction = utilsHelper.toFixed(prevPage.data.data.fullPriceReduction - feeData + result.data, 2);
                            prevPage.data.data.feeData = result.data;
                            app.globalData.deliveryAndSelfTaking.radioIndex = this.currentTab;
                            app.globalData.deliveryAndSelfTaking.deliveryAddress = shopAddress;
                            app.globalData.deliveryAndSelfTaking.feeData = result.data;
                            app.globalData.deliveryAndSelfTaking.reducedDeliveryTotalPrice = result.data;
                            app.globalData.deliveryAndSelfTaking.isThereADiscount = false;
                            if (app.globalData.deliveryAndSelfTaking.feeData != result.data) {
                                app.globalData.deliveryAndSelfTaking.isThereADiscount = true;
                            }
                            let time = dateHelper.formatTime('hms');
                            console.log('用户地址配送费==========' + app.globalData.deliveryAndSelfTaking.feeData);
                            console.log('最终用户需支付的配送费==========' + feeData);
                            //prevPage.getShopAddressList();
                            // //刷新获取满减
                            prevPage.getFullReductionRuleMode(prevPage.data.data);
                            console.log(prevPage.data);
                            // //刷新获取新人优惠券
                            //prevPage.getCouponsMemberRelation();
                            prevPage.setData({
                                deliveryAndSelfTaking: app.globalData.deliveryAndSelfTaking,
                                shopAddress: this.shopAddress,
                                data: prevPage.data.data,
                                oldAddressId: this.addressList[key].id,
                                time: time,
                                isPreviousPage: true,
                                isJs: true
                            });
                            this.setData({
                                itemId: this.addressList[key].id
                            });
                            uni.navigateBack(1);
                            return;
                        }
                        app.globalData.deliveryAndSelfTaking.feeData = 0;
                        prevPage.setData({
                            deliveryAndSelfTaking: app.globalData.deliveryAndSelfTaking
                        });
                    });
                return;
            }

            //选择地址和更换地址进行配送费的加减操作
            https
                .request('/api-order/rest/common/selectDeliveryFee', {
                    deliveryAddressId: shopAddress.id,
                    shopId: this.shopId
                })
                .then((result) => {
                    if (result.success) {
                        app.globalData.deliveryAndSelfTaking.radioIndex = this.currentTab;
                        app.globalData.deliveryAndSelfTaking.deliveryAddress = shopAddress;
                        app.globalData.deliveryAndSelfTaking.feeData = result.data;
                        prevPage.setData({
                            deliveryAndSelfTaking: app.globalData.deliveryAndSelfTaking
                        });
                        uni.navigateBack(1);
                    }
                });
            // })
        },

        editAddressTap(e) {
            uni.navigateTo({
                url: '../edit/edit?detail=' + JSON.stringify(e.currentTarget.dataset.data)
            });
            this.setData({
                refreshKey: e.currentTarget.dataset.key
            });
        },

        selfAdaption() {
            var that = this;
            let winHeight = 0;
            let height = 0;
            uni.getSystemInfo({
                success: function (res) {
                    //获取到用户的手机信息
                    winHeight = res.windowHeight;
                    // 获取需要减去的dom结构的高度信息
                    uni.createSelectorQuery()
                        .in(uni)
                        .selectAll('.self-adaption')
                        .boundingClientRect(function (rects) {
                            rects.forEach(function (rect, index) {
                                height = height + rect.height;
                            });
                            that.setData({
                                winHeight: winHeight - height
                            });
                        })
                        .exec();
                }
            });
        },

        getShopAddressList() {
            https
                .request('/api-merchant/rest/shop/detail', {
                    id: this.shopId,
                    position: app.globalData.deliveryAndSelfTaking.location
                })
                .then((result) => {
                    if (result.success) {
                        this.setData({
                            shopAddress: result.data
                        });
                    }
                });
        },

        getDeliveryAddressList() {
            https
                .request('/api-user/rest/member/deliveryAddress/list', {
                    pageNo: -1,
                    pageSize: 10
                })
                .then((result) => {
                    toastService.hideLoading();
                    if (result.success) {
                        var list = this.addressList;
                        result.data.records.forEach((key) => {
                            // if(key.isDefault){
                            //   this.deliveryAndSelfTaking.deliveryAddress = key;
                            // }
                            list.push(key);
                        });
                        this.setData({
                            addressList: list
                        });
                    }
                });
        },

        onPullDownRefresh() {
            console.log('占位：函数 onPullDownRefresh 未声明');
        },

        onReachBottom() {
            console.log('占位：函数 onReachBottom 未声明');
        },

        goToDrink() {
            console.log('占位：函数 goToDrink 未声明');
        },

        confirmShopChoice() {
            console.log('占位：函数 confirmShopChoice 未声明');
        }
    }
};
</script>
<style>
@import './choose.css';
</style>
