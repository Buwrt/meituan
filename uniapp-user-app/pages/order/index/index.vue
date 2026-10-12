<template>
    <view>
        <view class="">
            <view class="swiper-content position-sticky">
                <view class="swiper-tab self-adaption">
                    <view class="swiper-tab-item" :data-current="index" @tap="clickTab" hover-class="hover-click-class" v-for="(item, index) in tabList" :key="index">
                        <view :class="'swiper_table_item_view ' + (currentTab == item.modeId ? 'active' : '')" :data-current="index" @tap="clickTab">
                            {{ item.modeName }}
                        </view>
                    </view>
                </view>
            </view>
            <view>
                <mp-searchbar
                    @selectresult="selectResult"
                    :search="search"
                    @clear="searchInputClear"
                    @hide="searchInputClear"
                    placeholder="请输入联系人手机号/订单号"
                ></mp-searchbar>
            </view>
            <scroll-view class="swiper-content" v-if="currentTab == 0" :scroll-x="true">
                <view class="swiper-tab self-adaption">
                    <view
                        class="swiper-tab-item two-tab"
                        :data-current="index"
                        @tap="clickOrderTab"
                        hover-class="hover-click-class"
                        v-for="(item, index) in shopOrderTabList"
                        :key="index"
                    >
                        <view
                            :class="'swiper_table_item_view ' + (currentOrderTab == item.modeId ? 'active' : '')"
                            :data-current="index"
                            @tap="clickOrderTab"
                            style="position: relative; margin-right: 10px"
                        >
                            <text class="out_of_range one_row">{{ item.modeName }}</text>
                            <!-- <text class="tab-number" wx:if="{{item.number>0}}">({{item.number}})</text> -->
                            <mp-badge v-if="item.number > 0" :content="item.number" style="position: absolute; top: -0.9em; right: -0.8em" />
                        </view>
                    </view>
                </view>
            </scroll-view>
            <scroll-view class="swiper-content" v-else :scroll-x="true">
                <view class="swiper-tab self-adaption">
                    <view
                        class="swiper-tab-item two-tab"
                        :data-current="index"
                        @tap="clickOrderTab"
                        hover-class="hover-click-class"
                        v-for="(item, index) in pointsOrderTabList"
                        :key="index"
                    >
                        <view
                            :class="'swiper_table_item_view ' + (currentOrderTab == item.modeId ? 'active' : '')"
                            :data-current="index"
                            @tap="clickOrderTab"
                            style="position: relative; margin-right: 10px"
                        >
                            <text class="out_of_range one_row" :decode="true">{{ item.modeName }}</text>
                            <!-- <text class="tab-number" wx:if="{{item.number>0}}">({{item.number}})</text> -->
                            <mp-badge v-if="item.number > 0" :content="item.number" style="position: absolute; top: -0.9em; right: -0.8em" />
                        </view>
                    </view>
                </view>
            </scroll-view>
        </view>

        <swiper :current="currentOrderTab" class="swiper-box" duration="300" @change="bindSlideChange" :style="'height:' + winHeight + 'px;'" v-if="currentTab == 0">
            <swiper-item class="swiper-items" v-for="(item, index) in shopOrderTabList" :key="index">
                <scroll-view
                    @scrolltoupper="onPullDownRefresh"
                    upper-threshold="-50"
                    :style="'height:' + winHeight + 'px;'"
                    @scrolltolower="onReachBottom"
                    lower-threshold="0"
                    scroll-y
                    class="scroll-views"
                    v-if="list.length > 0"
                >
                    <view class="order-list-view" v-for="(item, index1) in list" :key="index1">
                        <view class="order-item flex_between" hover-class="hover-class-public" @tap="orderDetailsTap" :data-id="item.id" :data-index="index">
                            <image :src="item.shopLogoImg" class="commodity-image" mode="aspectFill"></image>
                            <view class="right-item">
                                <view class="address-status-info">
                                    <view class="address-view">
                                        <!-- <view class="order-mode theme-color-border">{{item.shoppingWay==1?"自取":"外卖"}}</view> -->
                                        <view class="address-info out_of_range one_row">
                                            {{ item.shopName }}
                                        </view>
                                    </view>
                                    <view class="status-view">{{ item.paymentModeText }}</view>
                                </view>
                                <view class="name-num-money">
                                    <view class="name-num flex_between">
                                        <text class="name-view out_of_range one_row" :decode="true">{{ item.description }}</text>
                                        <text class="num-view">x{{ item.goodsTotalQuantity }}</text>
                                    </view>
                                    <view class="money-view">￥{{ item.actualPrice }}</view>
                                </view>
                                <view class="tiem-view">
                                    {{ item.createTime }}
                                    <text class="status-text theme-color">{{ item.statusText }}</text>
                                </view>
                            </view>
                        </view>

                        <view class="tiem-view" v-if="item.isAllowAppraise">
                            <text></text>
                            <button
                                class="evaluate-btn theme-color theme-border"
                                size="mini"
                                :data-id="item.id"
                                :data-shopId="item.shopId"
                                @tap.stop.prevent="evaluateTip"
                                style="font-size: 22rpx; margin-top: 0"
                            >
                                评价
                            </button>
                        </view>
                    </view>
                </scroll-view>

                <is-show-tip
                    :isShow="list.length <= 0"
                    @bindTap="goToDrink"
                    buttonText="去喝一杯"
                    :top="45"
                    :bottom="10"
                    :text="list.length <= 0"
                    :tipText="'暂无' + shopOrderTabList[currentOrderTab].modeName + '订单'"
                ></is-show-tip>
            </swiper-item>
        </swiper>
        <swiper :current="currentOrderTab" class="swiper-box" duration="300" @change="bindSlideChange" :style="'height:' + winHeight + 'px;'" v-else>
            <swiper-item class="swiper-items" v-for="(item, index) in pointsOrderTabList" :key="index">
                <scroll-view
                    @scrolltoupper="onPullDownRefresh"
                    upper-threshold="-50"
                    :style="'height:' + winHeight + 'px;'"
                    @scrolltolower="onReachBottom"
                    lower-threshold="0"
                    scroll-y
                    class="scroll-views"
                    v-if="mallList.length > 0"
                >
                    <view class="order-list-view" v-for="(item, index1) in mallList" :key="index1">
                        <view class="order-item flex_between" hover-class="hover-class-public" @tap="orderMallDetailsTap" :data-id="item.id" :data-index="index">
                            <image :src="item.firstGoodsMainImage" class="commodity-image" mode="aspectFill"></image>
                            <view class="right-item">
                                <view class="address-status-info">
                                    <view class="address-view">
                                        <!-- <view class="order-mode theme-color-border">{{item.shoppingWay==1?"自取":"外卖"}}</view> -->
                                        <view class="address-info out_of_range one_row">{{ item.contactCity }}{{ item.contactArea }}{{ item.contactStreet }}</view>
                                        <view class="address-info out_of_range one_row">
                                            {{ item.shopName }}
                                        </view>
                                    </view>
                                    <view class="status-view">{{ item.paymentModeText }}</view>
                                </view>
                                <view class="name-num-money">
                                    <view class="name-num flex_between">
                                        <text class="name-view out_of_range one_row" :decode="true">{{ item.description }}</text>
                                        <text class="num-view">x{{ item.goodsTotalQuantity }}</text>
                                    </view>
                                    <view class="money-view">￥{{ item.actualPrice }}</view>
                                </view>
                                <view class="tiem-view">
                                    {{ item.createTime }}
                                    <text class="status-text theme-color">{{ item.statusText }}</text>
                                </view>
                            </view>
                        </view>

                        <view class="tiem-view" v-if="item.isAllowAppraise">
                            <text></text>
                            <button
                                class="evaluate-btn theme-color theme-border"
                                size="mini"
                                :data-id="item.id"
                                :data-shopId="item.shopId"
                                @tap.stop.prevent="evaluateTip"
                                style="font-size: 22rpx; margin-top: 0"
                            >
                                评价
                            </button>
                        </view>
                    </view>
                </scroll-view>

                <is-show-tip
                    :isShow="mallList.length <= 0"
                    @bindTap="goToDrink"
                    buttonText="去喝一杯"
                    :top="45"
                    :bottom="10"
                    :text="mallList.length <= 0"
                    :tipText="'暂无' + pointsOrderTabList[currentOrderTab].modeName + '订单'"
                ></is-show-tip>
            </swiper-item>
        </swiper>
    </view>
</template>

<script>
import mpSearchbar from '../../../components/searchbar/searchbar';
import mpBadge from '../../../components/badge/badge';
import isShowTip from '../../../components/is-show-tip/is-show-tip';
import https from '../../../utils/http';
import toastService from '../../../utils/toast.service';
import dateHelper from '../../../utils/date-helper';
import systemStatus from '../../../utils/system-status';
import authService from '../../../utils/auth';
import GlabalConfig from '../../../utils/global-config';
var pageNo = 1;
var pageSize = 10;
var pageMallNo = 1;
var pageMallSize = 10;
var id;
var index;
var isEndList = false;
var isEndMallList = false; //获取应用实例
const app = getApp();
export default {
    components: {
        mpSearchbar,
        mpBadge,
        isShowTip
    },
    data() {
        return {
            currentTab: 0,
            orderTab: 1,
            winWidth: 0,
            winHeight: 0,

            shopOrderTabList: [
                {
                    modeId: 0,
                    modeType: 'all',
                    modeName: '全部'
                },
                {
                    modeId: 1,
                    modeType: 'waitPayment',
                    modeName: '待付款'
                },
                {
                    modeId: 2,
                    modeType: 'waitReceived',
                    modeName: '待收货'
                },
                {
                    modeId: 3,
                    modeType: 'waitPickedUp',
                    modeName: '待自提'
                },
                {
                    modeId: 4,
                    modeType: 'afterSales',
                    modeName: '退款/售后'
                }
            ],

            pointsOrderTabList: [
                {
                    modeId: 0,
                    modeType: 'all',
                    modeName: '全部'
                },
                {
                    modeId: 1,
                    modeType: 'waitPayment',
                    modeName: '待付款'
                },
                {
                    modeId: 2,
                    modeType: 'waitDelivered',
                    modeName: '待发货'
                },
                {
                    modeId: 3,
                    modeType: 'waitReceived',
                    modeName: '待收货'
                },
                {
                    modeId: 4,
                    modeType: 'afterSales',
                    modeName: '退款/售后'
                }
            ],

            list: [],
            mallList: [],

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

            currentOrderTab: 0,
            keyWords: '',
            inputShowed: false,
            inputVal: '',
            i: 0,
            tabType: '',
            search: '',
            modeType: '',
            isOperation: false,
            modeName: ''
        };
    },
    /**
     * 生命周期函数--监听页面加载
     */
    onLoad: function (options) {
        console.log(options);
        pageNo = 1;
        this.getHeight();
        // this.getMallOrderList("", "");
        this.setData({
            search: this.search.bind(this),
            currentTab: options.currentTab,
            currentOrderTab: options.currentOrderTab,
            modeType: options.modeType
        });
        this.selectTabNumber();
        this.selectMallTabNumber();
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
                console.log(this.modeType);
                if (this.currentTab == 0) {
                    console.log('如果pageNo乘以pageSize不等于pageSize====' + pageNo * pageSize);
                    console.log('如果pageNo乘以pageSize不等于pageSize====' + pageSize);
                    let no = pageNo;
                    if (pageNo * pageSize != pageSize) {
                        pageSize = no * pageSize;
                        pageNo = 1;
                        console.log('获取pageSize数' + pageSize);
                    }
                    this.getOrderList(this.modeType, '');
                    if (this.isOperation) {
                        this.getOrderDetail(this.modeType, '');
                    }
                }
                if (this.currentTab == 1) {
                    console.log('如果pageNo乘以pageSize不等于pageSize====' + pageMallNo * pageMallSize);
                    console.log('如果pageNo乘以pageSize不等于pageSize====' + pageMallSize);
                    let no = pageMallNo;
                    if (pageMallNo * pageMallSize != pageMallSize) {
                        pageMallSize = no * pageMallSize;
                        pageMallNo = 1;
                        console.log('获取pageSize数' + pageMallSize);
                    }
                    this.getMallOrderList(this.modeType, '');
                    if (this.isOperation) {
                        // this.getOrderDetail();
                    }
                }
                return;
            }
            app.globalData.checkIsAuth('scope.userInfo');
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
    onPullDownRefresh: function () {
        // 显示顶部刷新图标
        uni.showNavigationBarLoading();
        if (this.currentTab == 0) {
            pageNo = 1;
            pageSize = 10;
            isEndList = false;
            this.getOrderList(this.shopOrderTabList[this.currentOrderTab].modeType, this.keyWords);
        }
        if (this.currentTab == 1) {
            pageMallNo = 1;
            pageMallSize = 10;
            isEndMallList = false;
            this.getMallOrderList(this.pointsOrderTabList[this.currentOrderTab].modeType, this.keyWords);
        }
        // 隐藏导航栏加载框
        uni.hideNavigationBarLoading();
        // 停止下拉动作
        uni.stopPullDownRefresh();
    },
    /**
     * 页面上拉触底事件的处理函数
     */
    onReachBottom: function () {
        toastService.showLoading('正在加载...');
        if (this.currentTab == 0) {
            if (isEndList) {
                toastService.showToast('没有更多啦~');
                return;
            }
            pageNo = pageNo + 1;
            pageSize = 10;
            https
                .request('/api-order/rest/member/order/list', {
                    pageNo: pageNo,
                    pageSize: pageSize,
                    tabType: this.pointsOrderTabList[this.currentOrderTab].modeType,
                    keyWords: this.keyWords
                })
                .then((result) => {
                    toastService.hideLoading();
                    if (result.success) {
                        if (result.data.records.length > 0) {
                            result.data.records.forEach((res) => {
                                res.createTime = dateHelper.formatDate(res.createTime);
                                res.statusText = systemStatus.statusText(res.status);
                                res.shopLogoImg = GlabalConfig.ossUrl + res.shopLogoImg;
                                this.list.push(res);
                            });
                            this.setData({
                                list: this.list
                            });
                            return;
                        }
                        isEndList = true;
                        toastService.showToast('没有更多啦~');
                        pageNo = pageNo - 1;
                    }
                });
        }
        if (this.currentTab == 1) {
            if (isEndMallList) {
                toastService.showToast('没有更多啦~');
                return;
            }
            pageMallNo = pageMallNo + 1;
            pageMallSize = 10;
            https
                .request('/api-mall/rest/member/pointsMall/order/list', {
                    pageNo: pageMallNo,
                    pageSize: pageMallSize,
                    tabType: this.pointsOrderTabList[this.currentOrderTab].modeType,
                    keyWords: this.keyWords
                })
                .then((result) => {
                    toastService.hideLoading();
                    if (result.success) {
                        if (result.data.records.length > 0) {
                            result.data.records.forEach((res) => {
                                res.createTime = dateHelper.formatDate(res.createTime);
                                res.statusText = systemStatus.statusText(res.status);
                                res.firstGoodsMainImage = GlabalConfig.ossUrl + res.firstGoodsMainImage;
                                this.mallList.push(res);
                            });
                            this.setData({
                                mallList: this.mallList
                            });
                            return;
                        }
                        isEndMallList = true;
                        toastService.showToast('没有更多啦~');
                        pageMallNo = pageMallNo - 1;
                    }
                });
        }
    },
    /**
     * 用户点击右上角分享
     */
    onShareAppMessage: function () {},
    methods: {
        getHeight() {
            //获取用户手机系统信息
            var that = this;
            uni.getSystemInfo({
                success: function (res) {
                    let winHeight = res.windowHeight; //获取高度
                    //获取class为settlement-view并减去这个高度，自适应scrollview的高度
                    const query = uni.createSelectorQuery();
                    query.select('.swiper-content').boundingClientRect();
                    query.selectViewport().scrollOffset();
                    query.exec(function (res) {
                        console.log(winHeight - res[0].height);
                        that.setData({
                            winHeight: winHeight - res[0].height
                        });
                    });
                }
            });
        },

        // 滑动切换tab
        bindSlideChange: function (e) {
            console.log(e);
            // if (e.detail.current == 1) {
            //   if (this.data.mallList.length <= 0) {
            //     this.getMallOrderList();
            //   }
            // }
            pageNo = 1;
            this.setData({
                currentOrderTab: e.detail.current
            });
            if (this.currentTab == 0) {
                this.setData({
                    modeType: this.shopOrderTabList[e.detail.current].modeType
                });
                this.getOrderList(this.shopOrderTabList[e.detail.current].modeType, this.keyWords);
            }
            if (this.currentTab == 1) {
                this.setData({
                    modeType: this.pointsOrderTabList[e.detail.current].modeType
                });
                this.getMallOrderList(this.pointsOrderTabList[e.detail.current].modeType, this.keyWords);
            }
        },

        //点击切换
        clickTab: function (e) {
            console.log(e.target.dataset.current);
            if (this.currentTab === e.target.dataset.current) {
                return false;
            } else {
                // 显示加载图标
                this.setData({
                    currentTab: e.target.dataset.current,
                    currentOrderTab: 0
                });
                if (e.target.dataset.current == 0) {
                    this.setData({
                        modeType: this.shopOrderTabList[0].modeType
                    });
                    this.getOrderList(this.pointsOrderTabList[0].modeType, this.keyWords);
                }
                if (e.target.dataset.current == 1) {
                    console.log(this.pointsOrderTabList[0].modeType);
                    this.setData({
                        modeType: this.pointsOrderTabList[0].modeType
                    });
                    this.getMallOrderList(this.pointsOrderTabList[0].modeType, this.keyWords);
                }
            }
        },

        //点击切换
        clickOrderTab: function (e) {
            console.log(e);
            console.log(e.currentTarget.dataset.current);
            if (this.currentOrderTab === e.currentTarget.dataset.current) {
                return false;
            } else {
                pageNo = 1;
                // 显示加载图标
                this.setData({
                    currentOrderTab: e.currentTarget.dataset.current
                });
                //console.log(this.data.pointsOrderTabList[e.target.dataset.current].modeType);
                // if (this.data.currentTab == 0) {
                //   this.setData({
                //     modeType:this.data.shopOrderTabList[e.target.dataset.current].modeType
                //   })
                //   this.getOrderList(this.data.shopOrderTabList[e.target.dataset.current].modeType, this.data.keyWords);
                // }
                // if (this.data.currentTab == 1) {
                //   this.setData({
                //     modeType:this.data.pointsOrderTabList[e.target.dataset.current].modeType
                //   })
                //   this.getMallOrderList(this.data.pointsOrderTabList[e.target.dataset.current].modeType, this.data.keyWords);
                // }
            }
        },

        orderDetailsTap(e) {
            id = e.currentTarget.dataset.id;
            index = e.currentTarget.dataset.index;
            uni.navigateTo({
                url: '../detail/detail?id=' + e.currentTarget.dataset.id
            });
        },

        orderMallDetailsTap(e) {
            id = e.currentTarget.dataset.id;
            index = e.currentTarget.dataset.index;
            console.log(this.modeType);
            uni.navigateTo({
                url: '../../internal/mall/order/detail/detail?id=' + e.currentTarget.dataset.id
            });
        },

        evaluateTip(e) {
            id = e.currentTarget.dataset.id;
            let shopId = e.currentTarget.dataset.shopid;
            uni.navigateTo({
                url: '../evaluate/evaluate?orderId=' + id + '&shopId=' + shopId
            });
        },

        goToDrink() {
            uni.switchTab({
                url: '../../menu/index/index'
            });
        },

        getOrderList(tabType, keyWords) {
            toastService.showLoading('正在加载...');
            https
                .request('/api-order/rest/member/order/list', {
                    pageNo: pageNo,
                    pageSize: pageSize,
                    tabType: tabType,
                    keyWords: keyWords
                })
                .then((result) => {
                    toastService.hideLoading();
                    if (result.success) {
                        result.data.records.forEach((res) => {
                            res.createTime = dateHelper.fmtDate(res.createTime);
                            res.statusText = systemStatus.statusText(res.status);
                            res.paymentModeText = systemStatus.paymentModeText(res.paymentMode);
                            res.shopLogoImg = GlabalConfig.ossUrl + res.shopLogoImg;
                        });
                        this.setData({
                            list: result.data.records
                        });
                    }
                });
        },

        getMallOrderList(tabType, keyWords) {
            toastService.showLoading('正在加载...');
            https
                .request('/api-mall/rest/member/pointsMall/order/list', {
                    pageNo: pageMallNo,
                    pageSize: pageMallSize,
                    tabType: tabType,
                    keyWords: keyWords
                })
                .then((result) => {
                    toastService.hideLoading();
                    if (result.success) {
                        result.data.records.forEach((res) => {
                            res.createTime = dateHelper.fmtDate(res.createTime);
                            res.statusText = systemStatus.statusMallText(res.status);
                            res.paymentModeText = systemStatus.paymentModeText(res.paymentMode);
                            res.firstGoodsMainImage = GlabalConfig.ossUrl + res.firstGoodsMainImage;
                        });
                        this.setData({
                            mallList: result.data.records
                        });
                    }
                });
        },

        searchFun: function (keyWords) {
            console.log(keyWords);
            return new Promise((resolve, reject) => {
                //判断大标签页
                if (this.currentTab == 0) {
                    this.getOrderList(this.shopOrderTabList[this.currentOrderTab].modeType, keyWords);
                }
                if (this.currentTab == 1) {
                    this.getMallOrderList(this.pointsOrderTabList[this.currentOrderTab].modeType, keyWords);
                }
                resolve([]);
                this.setData({
                    keyWords: keyWords
                });
            });
        },

        selectResult: function (e) {
            console.log('select result', e.detail);
        },

        searchInputClear(e) {
            console.log(e);
            if (this.currentTab == 0) {
                this.getOrderList(this.shopOrderTabList[this.currentOrderTab].modeType, '');
            }
            if (this.currentTab == 1) {
                this.getMallOrderList(this.pointsOrderTabList[this.currentOrderTab].modeType, '');
            }
            this.setData({
                keyWords: ''
            });
        },

        selectTabNumber() {
            https.request('/api-order/rest/member/order/selectTabNumber', {}).then((result) => {
                if (result.success) {
                    this.shopOrderTabList.forEach((item, index) => {
                        if (item.modeType != 'all') {
                            for (let keyof in result.data) {
                                if (keyof.indexOf(item.modeType) != -1) {
                                    console.log(keyof);
                                    console.log(result.data[keyof]);
                                    item.number = result.data[keyof];
                                    continue;
                                }
                            }
                        }
                    });
                    console.log(this.shopOrderTabList);
                    this.setData({
                        shopOrderTabList: this.shopOrderTabList
                    });
                }
            });
        },

        selectMallTabNumber() {
            https.request('/api-mall/rest/member/pointsMall/order/selectTabNumber', {}).then((result) => {
                if (result.success) {
                    this.pointsOrderTabList.forEach((item, index) => {
                        if (item.modeType != 'all') {
                            for (let keyof in result.data) {
                                if (keyof.indexOf(item.modeType) != -1) {
                                    console.log(keyof);
                                    console.log(result.data[keyof]);
                                    item.number = result.data[keyof];
                                    continue;
                                }
                            }
                        }
                    });
                    console.log(this.shopOrderTabList);
                    this.setData({
                        pointsOrderTabList: this.pointsOrderTabList
                    });
                }
            });
        },

        getOrderDetail() {
            https
                .request('/api-order/rest/member/order/selectById', {
                    id: id
                })
                .then((result) => {
                    if (result.success) {
                        const status = result.data.order.status;
                        const createTime = result.data.order.createTime;
                        result.data.order.statusText = systemStatus.statusText(status);
                        result.data.order.createTime = dateHelper.formatDate(createTime);
                        this.setData({
                            ['list[' + index + ']']: result.data.order,
                            isOperation: false
                        });
                    }
                });
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
@import './index.css';
</style>
