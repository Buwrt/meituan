<template>
    <view>
        <!-- <view class="member-info">
  <image src="{{userInfo.headImg}}" mode="widthFix" class="headImg"></image>
  <view class="nickname-vipEndTime">
    <view class="nickname">{{userInfo.username}}</view>
    <view class="vipEndTime {{userInfo.type==1?'ptyh-class':'cjhy-class'}}">

      {{userInfo.typeVipText}}
    </view>
  </view>
</view> -->
        <view class="swiper-content">
            <!-- <view class="swiper-tab self-adaption">
    <view class="swiper-tab-item" wx:for="{{modeList}}" wx:key="index" data-current="{{index}}" bindtap="clickTab"
      hover-class='hover-click-class'>
      <view class="swiper_table_item_view {{currentTab==item.modeId?'active':''}}" data-current="{{index}}"
        bindtap="clickTab">
        {{item.modeName}}
      </view>
    </view>
  </view> -->
            <view class="swiper-current">
                <swiper :current="currentTab" class="swiper-box" duration="300" @change="bindSlideChange" style="height: 80vh">
                    <swiper-item class="swiper-items">
                        <scroll-view
                            @scrolltoupper="onPullDownRefresh"
                            upper-threshold="-50"
                            style="height: 100%"
                            @scrolltolower="onReachBottom"
                            lower-threshold="0"
                            scroll-y
                            class="scroll-views"
                            v-if="list.length > 0"
                        >
                            <view class="content-recharge">
                                <radio-group class="radio-group" @change="radioChange">
                                    <label :class="'group-label ' + (!item.checked ? 'disabled-group-label' : 'active theme-bg')" v-for="(item, index) in list" :key="index">
                                        <view :class="item.isSale && item.briefDescription ? '' : 'label-view'">
                                            <radio :value="index" :checked="item.checked" class="radio" />
                                            <view class="recharge-name">{{ item.name ? item.name : '' }}</view>
                                            <view :class="'theme-color recharge-price ' + (item.checked ? 'radio-active' : '')">
                                                {{ item.isSale ? item.salePrice : item.price }}
                                                <text class="recharge-text" :decode="true">元</text>
                                            </view>
                                            <view class="sale-name" v-if="item.isSale">{{ item.isSale ? item.price : item.price }}元</view>
                                            <view class="brief-description" v-if="item.isSale && item.briefDescription">
                                                {{ item.briefDescription }}
                                            </view>
                                        </view>
                                    </label>
                                </radio-group>
                                <view class="description">
                                    <text>{{ list[checkValue].description }}</text>
                                </view>
                            </view>
                        </scroll-view>
                        <is-show-tip
                            :isShow="list.length <= 0"
                            @bindTap="goToDrink"
                            buttonText="去兑换"
                            :top="5"
                            :bottom="0"
                            :text="list.length <= 0"
                            tipText="暂无数据"
                            :button="button"
                        ></is-show-tip>
                    </swiper-item>
                    <!-- <swiper-item class="swiper-items">
        <view class="content-recharge balance-swiper">
          <view class="balance-tip">
            <text class="iconfont iconyue" decode="true">&nbsp;</text>余额查询</view>
          <view class="select-balance">
            <view class="balance-button" bindtap="bindMoneyBalance">
              <view class="balance-value">￥{{userInfo.balance}}</view>
              <view class="balance-title">钱包余额</view>
            </view>
            |
            <view class="balance-button" bindtap="bindIntegralBalance">
              <view class="balance-value">{{userInfo.points}}</view>
              <view class="balance-title">积分余额</view>
            </view>
          </view>
          <view class="lijichaxun-view">
            <view class="lijichaxun theme-bg" bindtap="bindMoneyBalance">
              立即查询
            </view>
          </view>
        </view>
      </swiper-item>
      <swiper-item class="swiper-items">
        <view class="content-recharge my-record-swiper" bindtap="bindRechargeRecord">
          <view>
            <view class="ckwdjl-view">查看你的充值记录</view>
            <view class="zdjl-czjl-view">
              <text>账单记录</text>
              <text decode="true">&nbsp;充值记录</text>
            </view>
            <view class="ljck-button theme-border theme-color">立即查看</view>
          </view>
          <image src="{{userInfo.headImg}}" mode="widthFix" class="headImg"></image>
        </view>
      </swiper-item> -->
                </swiper>
            </view>
        </view>
        <view class="bottom-view" v-if="list.length > 0">
            <view class="total-view">
                <view>
                    合计：
                    <text class="symbol-text">￥</text>
                    <text class="total-text">{{ list[checkValue].isSale ? list[checkValue].salePrice : list[checkValue].price }}</text>
                    <text class="strike_through sale-price" v-if="list[checkValue].isSale">￥{{ list[checkValue].price }}</text>
                </view>
                <view class="pay-button" @tap="bindPay">立即支付</view>
            </view>
        </view>
    </view>
</template>

<script>
import isShowTip from '../../../../components/is-show-tip/is-show-tip';
import https from '../../../../utils/http';
import authService from '../../../../utils/auth';
import toastService from '../../../../utils/toast.service';
import dateHelper from '../../../../utils/date-helper';
import systemStatus from '../../../../utils/system-status';
export default {
    components: {
        isShowTip
    },
    data() {
        return {
            currentTab: 0,

            modeList: [
                {
                    modeId: 0,
                    modeName: '充值'
                },
                {
                    modeId: 1,
                    modeName: '查余额'
                },
                {
                    modeId: 2,
                    modeName: '我的记录'
                }
            ],

            checkValue: 0,
            list: [],
            winHeight: '',
            userInfo: '',
            description: '',
            button: '',
            isSale: false,
            salePrice: '',
            price: ''
        };
    },
    /**
     * 生命周期函数--监听页面加载
     */
    onLoad: function (options) {
        this.getVipRechargeDenominationList();
        this.getUserInfo();
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
        getHeight() {
            //获取用户手机系统信息
            var that = this;
            uni.getSystemInfo({
                success: function (res) {
                    let winHeight = res.windowHeight; //获取高度
                    //获取class为settlement-view并减去这个高度，自适应scrollview的高度
                    const query = uni.createSelectorQuery();
                    query.select('.bottom-view').boundingClientRect();
                    query.selectViewport().scrollOffset();
                    query.exec(function (res) {
                        console.log(winHeight - res[0].height);
                        that.setData({
                            winHeight: winHeight - res[0].height - 160
                        });
                    });
                }
            });
        },

        getUserInfo: function (e) {
            https.request('/api-user/rest/member/getLoginMemberInfo', {}).then((result) => {
                if (result.success) {
                    result.data.typeVipText = systemStatus.typeVipText(result.data.type);
                    result.data.isVip = result.data.type == 1 ? false : true;
                    this.setData({
                        userInfo: result.data
                    });
                }
            });
        },

        getVipRechargeDenominationList: function (e) {
            toastService.showLoading('正在加载...');
            https
                .request('/api-user/rest/vipRechargeDenomination/list', {
                    pageNo: -1,
                    pageSize: 20
                })
                .then((result) => {
                    toastService.hideLoading();
                    if (result.success) {
                        console.log(result);
                        result.data.records.forEach((item, index) => {
                            item.checked = false;
                            if (index == 0) {
                                item.checked = true;
                            }
                            item.description = item.description.replace('↵', '\n');
                        });
                        this.setData({
                            list: result.data.records
                        });
                        this.getHeight();
                    }
                });
        },

        radioChange(e) {
            console.log(e);
            let checkValue = e.detail.value;
            for (let value in this.list) {
                this.list[value].checked = false;
            }
            this.list[checkValue].checked = true;
            this.setData({
                list: this.list,
                checkValue: checkValue
            });
        },

        // 滑动切换tab
        bindSlideChange: function (e) {
            this.setData({
                currentTab: e.detail.current
            });
            // this.getHeight();
        },

        //点击切换
        clickTab: function (e) {
            if (this.currentTab === e.target.dataset.current) {
                return false;
            } else {
                this.setData({
                    currentTab: e.target.dataset.current
                });
                // this.getHeight();
            }
        },

        bindPay(e) {
            toastService.showLoading('正在加载...', true);
            var vipRechargeDenominationId = this.list[this.checkValue].id;
            var total_fee = this.list[this.checkValue].price;
            if (this.list[this.checkValue].isSale) {
                total_fee = this.list[this.checkValue].salePrice;
            }
            var that = this;
            authService.getOpenId().then((openId) => {
                if (openId) {
                    https
                        .request('/api-order/rest/member/wxPay/toPay4Applet', {
                            openid: openId,
                            type: 2,
                            vipRechargeDenominationId: vipRechargeDenominationId,
                            total_fee: total_fee
                        })
                        .then((result) => {
                            toastService.hideLoading();
                            if (result.success) {
                                //console.log(result)
                                uni.requestPayment({
                                    appId: result.data.appid,
                                    timeStamp: result.data.timeStamp,
                                    nonceStr: result.data.nonceStr,
                                    package: result.data.package,
                                    signType: 'MD5',
                                    paySign: result.data.paySign,
                                    success(res) {
                                        // that.getUserInfo();
                                        uni.navigateBack(1);
                                        toastService.showSuccess('支付成功', true);
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

        bindRechargeRecord(e) {
            uni.navigateTo({
                url: '../record/record'
            });
        },

        bindMoneyBalance() {
            uni.navigateTo({
                url: '../../balance/index/index'
            });
        },

        bindIntegralBalance() {
            uni.navigateTo({
                url: '../../integral/integral'
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
        }
    }
};
</script>
<style>
@import './recharge.css';
</style>
