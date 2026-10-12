<template>
    <view>
        <view class="page">
            <view class="banner">
                <view class="location-address" @tap="shoppingAddressTap">
                    <image src="/static/assets/common/locate.png" mode="widthFix" class="place-image"></image>
                    <text class="out_of_range one_row">{{ regeoInfo ? regeoInfo.name + regeoInfo.address : '请授权位置定位' }}</text>
                    <text class="iconfont iconweibiaoti35"></text>
                </view>
            </view>
            <view class="search-input-views position-sticky" @tap="searchBusinessTap">
                <view class="search-input-view">
                    <!-- <image src="../../assets/common/search.png" mode="widthFix" class="search-image-class"></image> -->
                    <text class="iconfont iconsousuo-copy"></text>
                    <text class="search-input">搜索商家、商品名称</text>
                </view>
            </view>

            <swiper
                :indicator-dots="indicatorDots"
                class="carousel-swiper"
                :autoplay="autoplay"
                :interval="interval"
                :duration="duration"
                :indicator-active-color="afterColor"
                style="margin-top: 10rpx"
            >
                <block v-for="(item, index) in carouselUrls" :key="index">
                    <swiper-item class="carousel-swiper-item">
                        <image :src="item.imagePath" class="carousel-image" mode="aspectFill" :data-imageLinkUrl="item.imageLinkUrl" @tap="carouseCommodityDetailTap" />
                    </swiper-item>
                </block>
            </swiper>
            <view v-if="recommendGoodsList.length > 0">
                <view class="recommend-business-title margin-common theme-color">商品推荐</view>
                <view class="like-items">
                    <view
                        class="like-item"
                        hover-class="hover-class-public"
                        @tap="commodityDetailTap"
                        :data-shopid="item.shopId"
                        :data-id="item.id"
                        v-for="(item, index) in recommendGoodsList"
                        :key="index"
                    >
                        <view class="like-detail-view">
                            <image :src="item.mainImage" mode="aspectFill" class="icon-like-class"></image>
                            <view class="fullname-class">
                                <text class="name-text out_of_range one_row">{{ item.name }}</text>
                                <text class="name-text latelyMonthlySales out_of_range one_row">月售：{{ item.latelyMonthlySales }}</text>
                                <!-- <text class="go-to-shop theme-bg" bindtap="businessTap" data-id="{{item.shopId}}">进店</text> -->
                            </view>
                            <view class="engname-class out_of_range one_row">{{ item.detailmainImage }}</view>
                        </view>

                        <view class="like-money-view">
                            <view class="like-money">
                                <text>￥</text>
                                <text>{{ item.price }}</text>
                            </view>
                            <view class="theme-color new-commodity-button">立即购买>></view>
                        </view>
                    </view>
                </view>
            </view>
            <view>
                <view class="recommend-business-title margin-common theme-color">推荐商家</view>
                <is-show-tip
                    :isShow="shopList.length <= 0"
                    :top="25"
                    :button="shopList.length <= 0"
                    :bottom="20"
                    :text="shopList.length <= 0"
                    @bindTap="getRegeo"
                    tipText="定位地址暂无商家"
                    buttonText="重新定位"
                ></is-show-tip>
                <view class="business-items margin-common" v-if="shopList.length > 0">
                    <view class="business-item" hover-class="hover-class-public" v-for="(item, shopIndex) in shopList" :key="shopIndex">
                        <view class="main-image-num" style="width: 25%">
                            <image :src="item.shopLogoImg" @tap="businessTap" :data-id="item.id" mode="widthFix" class="business-image"></image>
                            <!-- <view class="num" wx:if="{{shopCartNums[index].shopId==item.id&&shopCartNums[index].num>0}}">{{shopCartNums[index].num}}</view> -->
                            <view class="num" v-if="item.shopCartNums > 0">{{ item.shopCartNums }}</view>
                        </view>

                        <view class="business-info">
                            <view @tap="businessTap" :data-id="item.id">
                                <view class="business-info-flex business-name">
                                    <text>{{ item.name }}</text>
                                </view>
                                <view class="business-info-flex">
                                    <view class="business-sale">
                                        <text class="business-evaluate">★ {{ item.serviceRating }}</text>
                                        <text class="business-right">月售：{{ item.shopAdditionalVo.latelyMonthlySales }}</text>
                                    </view>
                                </view>
                                <view class="business-info-flex">
                                    <view class="business-sale">
                                        <text>起送￥{{ item.startDeliveryPrice }}</text>
                                        <text :decode="true">&nbsp;&nbsp;配送&nbsp;</text>
                                        <text v-if="item.isfeeData">{{ item.feeData }}</text>
                                        <text :class="'business-right ' + (item.isfeeData ? 'strike_through' : '')">￥{{ item.shopAdditionalVo.deliveryFee }}</text>
                                    </view>
                                    <text :decode="true">{{ item.shopAdditionalVo.deliveryDurationText }}&nbsp;&nbsp;{{ item.shopAdditionalVo.deliveryDistanceText }}</text>
                                </view>
                            </view>
                            <view
                                class="business-discount-info business-info-flex"
                                v-if="item.shopAdditionalVo.promotionList.length > 0 || item.isfeeData"
                                @tap="isPromotionTap"
                                :data-shopIndex="shopIndex"
                            >
                                <view class="business-discount out_of_range one_row">
                                    <radio-group class="radio-group">
                                        <label
                                            class="group-label theme-color-border"
                                            v-if="index < 4 && item.shopAdditionalVo.promotionList.length > 0"
                                            v-for="(rule, index) in item.shopAdditionalVo.promotionList"
                                            :key="index"
                                        >
                                            <radio class="radio"></radio>

                                            {{ rule.name }}
                                        </label>
                                        <view v-if="item.feeData" class="theme-color">配送费立减{{ item.shopAdditionalVo.deliveryFee - item.feeData }}元</view>
                                    </radio-group>
                                </view>
                                <text class="iconfont iconweibiaoti35-copy"></text>
                            </view>
                        </view>
                    </view>
                </view>
            </view>
        </view>
        <mp-halfScreenDialog :show="isActivityDialog" extClass="extClassSpecifications">
            <view slot="title">优惠活动</view>
            <view slot="desc">
                <scroll-view style="height: 30vh" scroll-y>
                    <view>
                        <view v-if="shopList[shopIndex].shopAdditionalVo.promotionList.length > 0">
                            <text class="dialog-title">优惠：</text>
                            <view class="business-discount-info business-info-flex">
                                <view class="business-discount out_of_range one_row">
                                    <view
                                        class="theme-color-border business-discount-list"
                                        v-for="(rule, index) in shopList[shopIndex].shopAdditionalVo.promotionList"
                                        :key="index"
                                    >
                                        {{ rule.name }}
                                    </view>
                                </view>
                            </view>
                        </view>
                        <view class="reduced-delivery-price" v-if="shopList[shopIndex].reducedDeliveryPrice > 0">
                            <text class="dialog-title">配送费：</text>
                            配送费立减{{ shopList[shopIndex].reducedDeliveryPrice }}元
                        </view>
                    </view>
                </scroll-view>
            </view>
        </mp-halfScreenDialog>
    </view>
</template>

<script>
import mpDialog from '../../components/dialog/dialog';
import isShowTip from '../../components/is-show-tip/is-show-tip';
import mpHalfScreenDialog from '../../components/half-screen-dialog/half-screen-dialog';
import GlobalConfig from '../../utils/global-config';
const show = require('../../utils/toast.service.js');
import https from '../../utils/http';
var amapFile = require('../../utils/gaode-libs/amap-wx.js');
var config = require('../../utils/gaode-libs/config.js');
import authService from '../../utils/auth';
//获取应用实例
const app = getApp();
export default {
    components: {
        mpDialog,
        isShowTip,
        mpHalfScreenDialog
    },
    data() {
        return {
            indicatorDots: true,
            autoplay: true,
            interval: 5000,
            duration: 1000,

            //beforeColor: "white",//指示点颜色,
            afterColor: '#f1a142',

            //当前选中的指示点颜色
            previousmargin: '60px',

            //前边距
            nextmargin: '60px',

            //后边距
            opacity: 0.4,

            extClass: 'weui-dialog-ext-index',
            scrollTop: false,
            noDataTip: '../../assets/common/no-data.png',
            shopList: [],
            isActivityDialog: false,
            statusBarHeight: '',
            dialogShow: false,
            maskClosable: false,

            regeoInfo: {
                name: '',
                address: ''
            },

            dialogvisible: false,
            recommendGoodsList: '',
            carouselUrls: '',
            shopIndex: '',

            shopAdditionalVo: {
                promotionList: []
            },

            rule: {
                name: ''
            },

            reducedDeliveryPrice: 0
        };
    },
    onLoad: function () {
        this.setData({
            statusBarHeight: app.globalData.systemInfoSync.statusBarHeight * 2
        });
        this.getCarouselList();
        this.getRegeo();
    },
    onShow: function () {
        //判断是否显示新用户弹窗
        // app.getMemberInfo().then(result => {
        //   if (result.isNewPeople && result.isRemindNewPeople) {
        //     this.openConfirm();
        //   }
        // });
        //app.getShoppingCarNumber();
    },
    onPullDownRefresh() {
        this.getCarouselList();
        this.getRegeo();
        setTimeout((outtime) => {
            // 隐藏导航栏加载框
            uni.hideNavigationBarLoading();
            // 停止下拉动作
            uni.stopPullDownRefresh();
            clearTimeout(outtime);
        }, 1000);
    },
    onHide() {
        this.setData({
            dialogShow: false,
            maskClosable: true
        });
    },
    methods: {
        getRegeo() {
            var self = this;
            var key = config.Config.key;
            var myAmapFun = new amapFile.AMapWX({
                key: key
            });
            myAmapFun.getRegeo({
                success: function (getRegeo) {
                    console.log(getRegeo);
                    if (!app.globalData.deliveryAndSelfTaking.regeoInfo) {
                        self.setData({
                            regeoInfo: getRegeo[0].regeocodeData.pois[0]
                        });
                        app.globalData.deliveryAndSelfTaking.location = getRegeo[0].longitude + ',' + getRegeo[0].latitude;
                        app.globalData.deliveryAndSelfTaking.regeoInfo = getRegeo[0].regeocodeData.pois[0];
                    } else {
                        app.globalData.deliveryAndSelfTaking.location = app.globalData.deliveryAndSelfTaking.location;
                        self.setData({
                            regeoInfo: app.globalData.deliveryAndSelfTaking.regeoInfo
                        });
                    }
                    show.showLoading();
                    self.getShopList();
                    self.getRecommendGoodsList();
                },
                fail: function (info) {
                    //toastService.showLoading();
                    // wx.showModal({title:info.errMsg})
                }
            });
        },

        getRegeoInit() {
            var self = this;
            if (!app.globalData.deliveryAndSelfTaking.regeoInfo) {
                var addressInfo = {
                    name: '麓谷小镇',
                    address: '岳麓大道尖山路口北300米',
                    location: '112.885538,28.232363'
                };
                self.setData({
                    regeoInfo: addressInfo
                });
                app.globalData.deliveryAndSelfTaking.location = addressInfo.location;
                app.globalData.deliveryAndSelfTaking.regeoInfo = addressInfo;
            } else {
                app.globalData.deliveryAndSelfTaking.location = app.globalData.deliveryAndSelfTaking.location;
                self.setData({
                    regeoInfo: app.globalData.deliveryAndSelfTaking.regeoInfo
                });
            }
            show.showLoading();
            self.getShopList();
            self.getRecommendGoodsList();
        },

        shoppingAddressTap() {
            uni.navigateTo({
                url: `../address/replace/replace?jump_page=index`
            });
        },

        showDialog: function () {
            this.setData({
                dialogvisible: true
            });
        },

        gotoShop(e) {
            app.globalData.isRemindNewPeople().then((result) => {
                if (result) {
                    this.setData({
                        dialogShow: false,
                        maskClosable: true
                    });
                }
                uni.switchTab({
                    url: '../menu/index/index'
                });
            });
        },

        businessTap(e) {
            uni.navigateTo({
                url: '../menu/index/index?id=' + e.currentTarget.dataset.id
            });
        },

        searchBusinessTap(e) {
            console.log(this);
            uni.navigateTo({
                url: '../menu/search/search?location=' + app.globalData.deliveryAndSelfTaking.location
            });
        },

        commodityDetailTap(e) {
            uni.navigateTo({
                url: '../menu/detail/detail?id=' + e.currentTarget.dataset.id + '&shopId=' + e.currentTarget.dataset.shopid
            });
        },

        carouseCommodityDetailTap(e) {
            let _this = this;
            authService.checkIsLogin().then((result) => {
                console.log(result);
                if (result) {
                    _this.getUserInfo(e);
                    return;
                }
                app.globalData.checkIsAuth('scope.userInfo');
            });
        },

        getUserInfo: function (e) {
            https.request('/api-user/rest/member/getLoginMemberInfo', {}).then((result) => {
                if (result.success) {
                    uni.navigateTo({
                        url: e.currentTarget.dataset.imagelinkurl
                    });
                } else {
                    app.globalData.checkIsAuth('scope.userInfo');
                }
            });
        },

        bindInDevelopment() {
            app.globalData.bindInDevelopment();
        },

        getRecommendGoodsList() {
            https
                .request('/api-goods/rest/goods/homePage/recommendGoodsList', {
                    pageNo: -1,
                    pageSize: 5,
                    position: app.globalData.deliveryAndSelfTaking.location
                })
                .then((result) => {
                    if (result.success) {
                        result.data.forEach(function (item, index) {
                            item.mainImage = GlobalConfig.ossUrl + item.mainImage;
                        });
                        this.setData({
                            recommendGoodsList: result.data
                        });
                    }
                });
        },

        getShopList() {
            if (app.globalData.deliveryAndSelfTaking.location) {
                https
                    .request('/api-merchant/rest/shop/list', {
                        pageNo: -1,
                        pageSize: 5,
                        position: app.globalData.deliveryAndSelfTaking.location
                    })
                    .then((result) => {
                        show.hideLoading();
                        if (result.success) {
                            result.data.records.forEach((item, index) => {
                                item.shopLogoImg = GlobalConfig.ossUrl + item.shopLogoImg;
                                item.isfeeData = false;
                                if (item.reducedDeliveryPrice > 0) {
                                    if (item.reducedDeliveryPrice >= item.shopAdditionalVo.deliveryFee) {
                                        item.feeData = 0;
                                        item.isfeeData = true;
                                    } else {
                                        item.feeData = item.shopAdditionalVo.deliveryFee - item.reducedDeliveryPrice;
                                        item.isfeeData = true;
                                    }
                                }
                            });
                            this.setData({
                                shopList: result.data.records
                            });
                            authService.checkIsLogin().then((result) => {
                                if (!result) {
                                    return;
                                }
                                this.getShoppingCartList();
                            });
                        }
                    });
            }
        },

        getShoppingCartList() {
            this.shopList.forEach((item, index) => {
                https
                    .request('/api-goods/rest/member/shoppingCart/list', {
                        shopId: item.id,
                        pageNo: -1,
                        pageSize: 20
                    })
                    .then((result) => {
                        show.hideLoading();
                        if (result.success && result.data) {
                            var number = 0;
                            result.data.records.forEach((cart, index) => {
                                number = number + cart.number;
                            });
                            item.shopCartNums = number;
                            this.setData({
                                shopList: this.shopList
                            });
                        }
                    });
            });
        },

        //获取满减规则
        getFullReductionRule(shopList) {
            shopList.forEach((item, index) => {
                https
                    .request('/api-promotion/rest/fullReductionRule/list', {
                        pageNo: -1,
                        pageSize: 1,
                        shopId: item.id
                    })
                    .then((result) => {
                        if (result.success) {
                            shopList[index].reductionRules = result.data.records;
                            this.setData({
                                shopList: shopList
                            });
                        }
                    });
            });
        },

        getCarouselList() {
            https
                .request('/api-promotion/rest/advertisement/list', {
                    type: 1,
                    pageNo: -1,
                    pageSize: 20
                })
                .then((result) => {
                    if (result.success) {
                        result.data.records.forEach(function (item, index) {
                            item.imagePath = GlobalConfig.ossUrl + item.imagePath;
                        });
                        this.setData({
                            carouselUrls: result.data.records
                        });
                    }
                });
        },

        openConfirm() {
            this.setData({
                dialogShow: true,
                maskClosable: false
            });
        },

        close() {
            app.globalData.isRemindNewPeople().then((result) => {
                if (result) {
                    this.setData({
                        dialogShow: false,
                        maskClosable: true
                    });
                }
            });
        },

        isPromotionTap(e) {
            let shopIndex = e.currentTarget.dataset.shopindex;
            console.log(shopIndex);
            this.setData({
                isActivityDialog: true,
                shopIndex: shopIndex
            });
        }
    }
};
</script>
<style>
@import './index.css';
</style>
