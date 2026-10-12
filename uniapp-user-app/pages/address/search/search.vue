<template>
    <view>
        <view class="section" v-if="userLocation">
            <region-picker @change="bindRegionChange" :value="region" :custom-item="customItem">
                <view class="picker flex_center">
                    {{ region[1] ? region[1] : '定位中...' }}
                    <text class="iconfont iconweibiaoti35" v-if="region[1]"></text>
                </view>
            </region-picker>
            <input @input="bindInput" :placeholder="'请输入地址名称	' + scope.userLocation" :focus="true" />
        </view>
        <view>
            <scroll-view :scroll-y="true" :style="'height:' + winHeight + 'px'">
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
            </scroll-view>
        </view>
    </view>
</template>

<script>
import isShowTip from '../../../components/is-show-tip/is-show-tip';
var amapFile = require('../../../utils/gaode-libs/amap-wx.js');
var config = require('../../../utils/gaode-libs/config.js');
import toastService from '../../../utils/toast.service';
var lonlat;
var city;
var keywords;
export default {
    components: {
        isShowTip
    },
    data() {
        return {
            tips: [],
            isButton: false,
            customItem: '',
            region: '',
            street: '',
            location: '',
            longitude: '',
            latitude: '',
            userLocation: '',
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
    methods: {
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
                            tips: data.tips
                        });
                    }
                }
            });
        },

        bindSearch: function (e) {
            keywords = e.currentTarget.dataset.keywords;
            console.log(keywords);
            this.prevPage(keywords);
        },

        prevPage(data) {
            var pages = getCurrentPages();
            var prevPage = pages[pages.length - 2]; //上一个页面
            //直接调用上一个页面的 setData() 方法，把数据存到上一个页面中去
            let district;
            let city;
            let regins;
            if (data.district) {
                //   district = data.district.split("省");
                //   city = district[1].split("市");
                //   regins = [district[0] + "省", city[0] + "市", city[1]]
                district = this.region[0];
                city = this.region[1];
                regins = [district, city, data.district.split('市')[1]];
            }
            prevPage.setData({
                region: data.district ? regins : this.region,
                street: data.name,
                location: data.location,
                longitude: data.location.split(',')[0],
                latitude: data.location.split(',')[1]
            });
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
                    toastService.hideLoading();
                    that.setData({
                        region: [data[0].regeocodeData.addressComponent.province, data[0].regeocodeData.addressComponent.city, data[0].regeocodeData.addressComponent.district],
                        tips: data[0].regeocodeData.pois
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

        getSettingInfo() {
            let that = this;
            uni.getSetting({
                success(res) {
                    console.log(res);
                    that.setData({
                        userLocation: res.authSetting['scope.userLocation'],
                        tips: []
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
        }
    }
};
</script>
<style>
@import './search.css';
</style>
