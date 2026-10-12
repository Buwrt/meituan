<template>
    <view>
        <view class="section-location">
            <view class="section" v-if="userLocation">
                <region-picker @change="bindRegionChange" :value="region" :custom-item="customItem">
                    <view class="picker flex_center">
                        {{ region[1] ? region[1] : '定位中...' }}
                        <text class="iconfont iconweibiaoti35" v-if="region[1]"></text>
                    </view>
                </region-picker>
                <input @input="bindInput" :placeholder="'请输入地址名称	' + scope.userLocation" :focus="true" />
            </view>
            <view class="location-address">
                <view class="address-title">当前地址</view>
                <view class="location-name theme-color">
                    <view @tap="bindSearch" :data-keywords="sectionLocation" v-if="sectionLocation" hover-class="hover-class-public">{{ sectionLocation.name }}</view>
                    <view class="please-location" v-else>请重新定位</view>
                    <view @tap="parseEventDynamicCode($event, isOpenSettingInfo ? 'openSettingInfo' : 'getRegeoAddress')">重新定位</view>
                </view>
            </view>
        </view>
        <view v-if="!searchInput">
            <view class="address-title">收货地址</view>
            <view class="delivery-address" v-if="deliveryAddressList.length > 0">
                <view
                    hover-class="hover-class-public"
                    class="delivery-address-item"
                    @tap="parseEventDynamicCode($event, addressType ? 'getShopAddressTap' : 'getDeliveryAddressTip')"
                    :data-address="item"
                    v-for="(item, index) in deliveryAddressList"
                    :key="index"
                >
                    <view class="delivery-address-detail">{{ item.province }}{{ item.city }}{{ item.area }}{{ item.street }}</view>

                    <view class="delivery-address-realname-phone">
                        <text>{{ item.realname }}</text>
                        <text :decode="true">{{ item.sex == 0 ? '先生' : '女士' }}&nbsp;&nbsp;&nbsp;</text>
                        <text>{{ item.phone }}</text>
                    </view>
                </view>
            </view>
            <is-show-tip
                :isShow="deliveryAddressList.length <= 0"
                @bindTap="goInsertAddress"
                buttonText="添加地址"
                :top="40"
                :bottom="10"
                :text="deliveryAddressList.length <= 0"
                tipText="暂无收货地址"
                :button="true"
            ></is-show-tip>
        </view>
        <view v-else>
            <view @tap="bindSearch" :data-keywords="i" class="text_box" v-for="(i, index) in tips" :key="index">
                <view class="address-name">
                    {{ i.name }}
                </view>

                <view class="address-address">
                    {{ i.address }}
                </view>
            </view>
            <is-show-tip
                :isShow="tips.length <= 0"
                @bindTap="openSettingInfo"
                buttonText="位置授权"
                :top="40"
                :bottom="10"
                :text="tips.length <= 0"
                tipText="没有搜索到地址"
                :button="!userLocation"
            ></is-show-tip>
        </view>
    </view>
</template>

<script>
import isShowTip from '../../../components/is-show-tip/is-show-tip';
var amapFile = require('../../../utils/gaode-libs/amap-wx.js');
var config = require('../../../utils/gaode-libs/config.js');
import toastService from '../../../utils/toast.service';
import https from '../../../utils/http';
import authService from '../../../utils/auth';
var lonlat;
var city;
var keywords;
const app = getApp();
export default {
    components: {
        isShowTip
    },
    data() {
        return {
            tips: [],
            isButton: false,
            searchInput: false,
            customItem: '',
            deliveryAddressList: [],
            jump_page: '',
            addressType: '',
            shopId: '',
            regeoInfo: '',
            region: [],

            sectionLocation: {
                name: ''
            },

            userLocation: '',
            isOpenSettingInfo: '',
            winHeight: '',

            scope: {
                userLocation: ''
            },

            i: {
                name: '',
                address: ''
            }
        };
    },
    onShow: function (e) {
        this.getSettingInfo();
        // this.getRegeoAddress();
        // this.getAutoHeight();
    },
    onLoad(options) {
        this.onLoadClone3389(options);
    },
    methods: {
        onLoadClone3389(options) {
            this.setData({
                jump_page: options.jump_page
            });
            if (options.addressType) {
                this.setData({
                    addressType: options.addressType,
                    shopId: options.shopId
                });
            }
            this.getDeliveryAddressList();
        },

        bindInput: function (e) {
            var that = this;
            keywords = e.detail.value;
            if (!keywords) {
                this.setData({
                    tips: []
                });
                return;
            }
            var key = config.Config.key;
            var myAmapFun = new amapFile.AMapWX({
                key: key
            });
            toastService.showLoading();
            myAmapFun.getInputtips({
                keywords: keywords,
                location: that.longitude + ',' + that.latitude,
                city: city,
                citylimit: true,
                output: 'JSON',
                success: function (data) {
                    toastService.hideLoading();
                    if (data && data.tips) {
                        that.setData({
                            tips: data.tips,
                            searchInput: true
                        });
                    }
                }
            });
        },

        getDeliveryAddressTip: function (e) {
            var that = this;
            keywords = e.currentTarget.dataset.address;
            console.log(keywords);
            var key = config.Config.key;
            var myAmapFun = new amapFile.AMapWX({
                key: key
            });
            toastService.showLoading();
            myAmapFun.getInputtips({
                keywords: keywords.street,
                city: keywords.city,
                citylimit: true,
                output: 'JSON',
                success: function (data) {
                    if (data && data.tips) {
                        console.log(data);
                        console.log(keywords);
                        // data.tips[0].name='';
                        // data.tips[0].address=keywords.street;
                        var location;
                        for (var i = 0; i < data.tips.length; i++) {
                            console.log(data.tips[i]);
                            if (data.tips[i].location.length > 0) {
                                location = data.tips[i].location;
                                break;
                            }
                        }
                        console.log(location);
                        app.globalData.deliveryAndSelfTaking.location = location ? location : keywords.longitude + ',' + latitude;
                        app.globalData.deliveryAndSelfTaking.deliveryAddress = e.currentTarget.dataset.address;
                        app.globalData.deliveryAndSelfTaking.regeoInfo.name = '';
                        app.globalData.deliveryAndSelfTaking.regeoInfo.address = keywords.street;
                        //self.prevPage(data.tips[0]);
                        if (that.jump_page == 'index') {
                            var pages = getCurrentPages();
                            var prevPage = pages[pages.length - 2];
                            prevPage.onLoad();
                        }
                        uni.navigateBack(1);
                    }
                }
            });
        },

        getShopAddressTap(e) {
            var _this = this;
            keywords = e.currentTarget.dataset.address;
            console.log(keywords);
            var key = config.Config.key;
            var myAmapFun = new amapFile.AMapWX({
                key: key
            });
            toastService.showLoading();
            myAmapFun.getInputtips({
                keywords: keywords.street,
                city: keywords.city,
                citylimit: true,
                output: 'JSON',
                success: function (data) {
                    console.log(data);
                    if (data && data.tips) {
                        let location = '';
                        for (let i = 0; i < data.tips.length; i++) {
                            if (data.tips[i].location.length > 0) {
                                location = data.tips[i].location;
                                app.globalData.deliveryAndSelfTaking.location = location;
                                console.log(location);
                                break;
                            }
                        }
                        app.globalData.deliveryAndSelfTaking.deliveryAddress = e.currentTarget.dataset.address;
                        app.globalData.deliveryAndSelfTaking.regeoInfo.name = '';
                        app.globalData.deliveryAndSelfTaking.regeoInfo.address = keywords.street;
                        if (_this.jump_page == 'index') {
                            var pages = getCurrentPages();
                            var prevPage = pages[pages.length - 2];
                            prevPage.getCarouselList();
                            prevPage.getRegeo();
                        }
                        uni.navigateBack(1);
                    }
                }
            });
        },

        bindSearch: function (e) {
            keywords = e.currentTarget.dataset.keywords;
            console.log(keywords);
            app.globalData.deliveryAndSelfTaking.regeoInfo.name = keywords.name;
            app.globalData.deliveryAndSelfTaking.regeoInfo.address = keywords.address;
            app.globalData.deliveryAndSelfTaking.location = keywords.location;
            //this.prevPage(keywords);
            if (this.jump_page == 'index') {
                var pages = getCurrentPages();
                var prevPage = pages[pages.length - 2];
                prevPage.getCarouselList();
                prevPage.getRegeo();
            }
            uni.navigateBack(1);
        },

        prevPage(data) {
            console.log(data);
            var pages = getCurrentPages();
            var prevPage = pages[pages.length - 2]; //上一个页面
            //直接调用上一个页面的 setData() 方法，把数据存到上一个页面中去
            prevPage.setData({
                regeoInfo: data
            });
            if (this.jump_page == 'index') {
                prevPage.getCarouselList();
                prevPage.getRegeoAddress();
            }
            uni.navigateBack(1);
        },

        getRegeoAddress() {
            var that = this;
            var key = config.Config.key;
            var myAmapFun = new amapFile.AMapWX({
                key: key
            });
            toastService.showLoading();
            myAmapFun.getRegeo({
                success: function (data) {
                    console.log(data);
                    toastService.hideLoading();
                    that.setData({
                        region: [data[0].regeocodeData.addressComponent.province, data[0].regeocodeData.addressComponent.city, data[0].regeocodeData.addressComponent.district],
                        //tips: data[0].regeocodeData.pois,
                        sectionLocation: data[0].regeocodeData.pois[0]
                    });
                    city = data[0].regeocodeData.addressComponent.city;
                    //console.log(city)
                    that.initData(data[0].regeocodeData.addressComponent.streetNumber.location);
                },
                fail: function (info) {
                    //toastService.showLoading();
                    //toastService.showToast(info.errMsg)
                    // wx.showModal({title:info.errMsg})
                }
            });
        },

        getRegeoInit() {
            var addressInfo = {
                name: '麓谷小镇',
                address: '岳麓大道尖山路口北300米',
                location: '112.885538,28.232363'
            };
            this.setData({
                region: ['湖南省', '长沙市', '岳麓区'],
                //tips: data[0].regeocodeData.pois,
                sectionLocation: addressInfo
            });
            city = '长沙市';
            //console.log(city)
            this.initData(addressInfo.location);
        },

        getDeliveryAddressList() {
            toastService.showLoading();
            https
                .request('/api-user/rest/member/deliveryAddress/list', {
                    pageNo: -1,
                    pageSize: 10
                })
                .then((result) => {
                    toastService.hideLoading();
                    if (result.success) {
                        console.log(result.data.records);
                        this.setData({
                            deliveryAddressList: result.data.records
                        });
                    }
                });
        },

        getSettingInfo() {
            let that = this;
            uni.getSetting({
                success(res) {
                    console.log(res);
                    that.setData({
                        userLocation: res.authSetting['scope.userLocation'],
                        tips: [],
                        isOpenSettingInfo: res.authSetting['scope.userLocation'] ? false : true
                    });
                    if (res.authSetting['scope.userLocation']) {
                        that.getRegeoAddress();
                        that.getAutoHeight();
                    }
                }
            });
        },

        openSettingInfo() {
            let that = this;
            uni.openSetting({
                success(res) {
                    console.log(res);
                    that.getSettingInfo();
                    // if (res.authSetting['scope.userLocation']) {
                    //   that.getRegeoAddress();
                    //   that.getAutoHeight();
                    // }
                }
            });
        },

        initData(location) {
            var key = config.Config.key;
            var myAmapFun = new amapFile.AMapWX({
                key: key
            });
            myAmapFun.getInputtips({
                location: location,
                city: city,
                citylimit: true,
                success: function (data) {
                    if (data && data.tips) {
                        that.setData({
                            tips: data.tips
                        });
                    }
                }
            });
        },

        bindRegionChange: function (e) {
            //console.log('picker发送选择改变，携带值为', e.detail.value)
            this.setData({
                region: e.detail.value
            });
            var value = {
                detail: {
                    value: keywords
                }
            };
            city = e.detail.value[1];
            this.bindInput(value);
        },

        getAutoHeight() {
            //获取用户手机系统信息
            var winHeight = 0;
            var that = this;
            uni.getSystemInfo({
                success: function (res) {
                    winHeight = res.windowHeight; //获取高度
                    //获取class为settlement-view并减去这个高度，自适应scrollview的高度
                    const query = uni.createSelectorQuery();
                    query.select('.section').boundingClientRect();
                    query.selectViewport().scrollOffset();
                    query.exec(function (res) {
                        that.setData({
                            winHeight: winHeight - res[0].height
                        });
                    });
                }
            });
        },

        goInsertAddress(e) {
            authService.checkIsLogin().then((result) => {
                if (result) {
                    uni.navigateTo({
                        url: '/pages/address/insert/insert'
                    });
                    return;
                }
                app.globalData.checkIsAuth('scope.userInfo');
            });
        }
    }
};
</script>
<style>
@import './replace.css';
</style>
