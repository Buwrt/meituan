<template>
    <view>
        <view class="top-detail" v-if="tipReward">
            <view class="current-tips" @tap.stop.prevent="customerServiceWechat">
                {{ tipReward }}
            </view>
        </view>
        <view class="ask-for-delivery-detail">
            <view class="swiper-tab self-adaption">
                <view class="swiper-tab-item" :data-current="index" @tap="clickTab" hover-class="hover-click-class" v-for="(item, index) in modeTabList" :key="index">
                    <view :class="'swiper_table_item_view ' + (deliveryAndSelfTaking.currentTab == item.modeId ? 'active' : '')" :data-current="index" @tap="clickTab">
                        {{ item.modeName }}
                    </view>
                </view>
            </view>
            <view @change="bindSlideChange" class="bindSlideChange">
                <view v-if="deliveryAndSelfTaking.currentTab == 0" class="bindSlideChange-class">
                    <view class="space-between-class">
                        <view class="deliveryAddress-info">
                            <view class="ask-for-delivery-house-view" v-if="deliveryAndSelfTaking.deliveryAddress" data-radioIndex="0" @tap="radioChange">
                                <view class="ask-for-delivery-view">
                                    <view class="ask-for-delivery-house">
                                        <text class="iconfont icondizhi"></text>
                                        <view class="right-class out_of_range two_row">
                                            {{ deliveryAndSelfTaking.deliveryAddress.province ? deliveryAndSelfTaking.deliveryAddress.province : ''
                                            }}{{ deliveryAndSelfTaking.deliveryAddress.city ? deliveryAndSelfTaking.deliveryAddress.city : '' }}
                                            {{ deliveryAndSelfTaking.deliveryAddress.area ? deliveryAndSelfTaking.deliveryAddress.area : ''
                                            }}{{ deliveryAndSelfTaking.deliveryAddress.street }}
                                        </view>
                                    </view>
                                    <view class="ask-for-delivery-address-detail">
                                        <text class="iconfont icon002dianhua"></text>
                                        <view class="right-class">
                                            <text>{{ deliveryAndSelfTaking.deliveryAddress.phone }}</text>
                                            <text :decode="true">
                                                &nbsp;&nbsp;&nbsp;{{ deliveryAndSelfTaking.deliveryAddress.realname
                                                }}{{ deliveryAndSelfTaking.deliveryAddress.sex == 0 ? '先生' : '女士' }}
                                            </text>
                                        </view>
                                    </view>
                                </view>
                                <text class="iconfont iconhtbArrowright02"></text>
                            </view>
                            <view class="view-line" v-if="deliveryAndSelfTaking.deliveryAddress"></view>
                            <view class="dayue-time" v-if="deliveryAndSelfTaking.deliveryAddress">
                                <text>立即送出</text>
                                <view class="dayue-time-songda-view">
                                    <text class="dayue-time-songda theme-color">大约{{ time }}送达</text>
                                </view>
                            </view>
                            <view class="ask-for-delivery-address-detail" v-else data-radioIndex="0" @tap="radioChange">
                                <text>去选择地址</text>
                                <text class="iconfont iconhtbArrowright02"></text>
                            </view>
                        </view>
                    </view>
                </view>
                <view v-if="deliveryAndSelfTaking.currentTab == 1">
                    <view class="currentTab1-view">
                        <view class="currentTab1-title">商家地址</view>
                        <view>{{ shopAddress.shop.name }}</view>
                        <view>
                            {{ shopAddress.shop.province ? shopAddress.shop.province : '' }}{{ shopAddress.shop.city ? shopAddress.shop.city : ''
                            }}{{ shopAddress.shop.area ? shopAddress.shop.area : '' }}{{ shopAddress.shop.street }}
                        </view>
                    </view>
                    <view class="time-phone-view">
                        <view class="time-phone-item">
                            <view class="time-phone-title">自取时间</view>
                            <view class="time-phone-content">
                                <view>{{ time }}</view>
                            </view>
                        </view>
                        <view class="time-phone-item">
                            <view class="time-phone-title">预留电话</view>
                            <view class="time-phone-content">
                                <input :value="phoneNumber" />
                                <text class="iconfont iconhtbArrowright02"></text>
                            </view>
                        </view>
                    </view>
                </view>
            </view>
        </view>
        <view class="shopping-commodity-details">
            <view>
                {{ shopAddress.shop.name }}
            </view>
            <view class="commodity-name-price-detail" v-for="(item, index) in data.orderDetailList" :key="index">
                <view class="commodity-name-types">
                    <view class="commodity-name">{{ item.goodsName }}</view>
                    <view class="commodity-types">{{ item.restructure }}</view>
                </view>

                <view class="commodity-totalnum-price">
                    <view class="commodity-totalnum">x{{ item.number }}</view>
                    <view class="commodity-price">￥{{ item.totalPrice }}</view>
                </view>
            </view>
            <view class="commodity-name-price-detail">
                <view class="commodity-name-types">
                    <view class="commodity-name">包装费</view>
                    <view class="commodity-types commodity-delivery-tip theme-color">商品包装费</view>
                </view>
                <view class="commodity-totalnum-price">
                    <view></view>
                    <view class="commodity-price">￥{{ data.packingCharges }}</view>
                </view>
            </view>
            <view class="commodity-name-price-detail" v-if="deliveryAndSelfTaking.currentTab == 0">
                <view class="commodity-name-types">
                    <view class="commodity-name">配送费</view>
                </view>
                <view class="commodity-totalnum-price">
                    <view></view>
                    <view class="commodity-price">
                        <text v-if="deliveryAndSelfTaking.isReducedDeliveryPrice" class="fullPriceReductionClass">￥{{ deliveryAndSelfTaking.reducedDeliveryTotalPrice }}</text>
                        <text :decode="true">&nbsp;&nbsp;&nbsp;￥{{ deliveryAndSelfTaking.feeData }}</text>
                    </view>
                </view>
            </view>
            <view class="commodity-name-price-detail" v-if="data.fullPriceReductionIsHidden">
                <view class="commodity-name-types">
                    <view class="commodity-name">满减</view>
                </view>
                <view class="commodity-totalnum-price">
                    <view></view>
                    <view class="commodity-price full-reduction-view theme-color-border">{{ data.fullReductionRuleName }}</view>
                </view>
            </view>
            <view class="view-line"></view>
            <view class="total-money-view">
                <text class="money-icon commodity-price">总计：</text>
                <view :class="data.fullPriceReductionIsHidden || data.couponsIsHidden || deliveryAndSelfTaking.isThereADiscount ? 'fullPriceReductionClass' : ''">
                    <text class="commodity-price">￥{{ data.actualPrice }}</text>
                </view>
                <text class="commodity-price" v-if="data.fullPriceReductionIsHidden || data.couponsIsHidden || deliveryAndSelfTaking.isThereADiscount" :decode="true">
                    &nbsp;&nbsp;&nbsp;￥{{ data.fullPriceReduction }}
                </text>
            </view>
        </view>
        <view class="pay-mode-view">
            <view class="pay-mode-title">优惠券</view>
            <view class="choose-pay-mode coupon-mode" @tap="parseEventDynamicCode($event, afterDiscount ? 'onCoupon' : '')">
                <view class="theme-color after-discount flex_end" v-if="afterDiscount">
                    <view class="out_of_range tow_row">
                        <text>{{ afterDiscount.couponsName ? afterDiscount.couponsName : '未使用优惠券 ' }}</text>
                        <text :decode="true" class="">&nbsp;&nbsp;已优惠{{ afterDiscount.price }}元</text>
                    </view>
                </view>
                <view class="theme-color after-discount" v-else>
                    <text>无可用优惠券</text>
                </view>
                <text class="iconfont iconhtbArrowright02"></text>
            </view>
        </view>
        <!-- <view class="pay-mode-view">
	<view>支付方式</view>
	<view class="choose-pay-mode" bindtap="choosePayModeTap">
		<text class="iconfont {{paymentModes[paymentModeIndex].icon}}"></text>
		<text decode="true">&nbsp;{{paymentModes[paymentModeIndex].text}}</text>
		<text class="iconfont iconhtbArrowright02"></text>
	</view>
</view> -->
        <view class="pay-mode-view margin-border-radius">
            <view>支付方式</view>
            <view class="choose-pay-mode">
                <radio-group class="radio-group-address" @change="radioChangeAddress" :data-firstIndex="key">
                    <label
                        v-if="item.show"
                        :class="'radio-label-payment flex_center ' + (item.checked ? 'payment-checked' : 'payment-not-checked')"
                        v-for="(item, index) in paymentModes"
                        :key="index"
                    >
                        <radio :value="index" :checked="item.checked" class="pay_radio" />

                        <text :class="'iconfont ' + item.icon"></text>

                        <text :decode="true">&nbsp;{{ item.text }}</text>

                        <text :decode="true" class="actionItem__desc" v-if="item.desc">&nbsp;{{ item.desc }}</text>
                    </label>
                </radio-group>
            </view>
        </view>
        <view class="ask-for-remarks-view">
            <!-- <view class="choose-ask-for" wx:if="{{radioIndex==0}}">
    <view>自取方式</view>
    <radio-group class="ask-for-radio-group">
      <label wx:for="{{modeList[radioIndex].mode}}" wx:key="index" class="ask-for-label">
        <radio value="{{index}}" checked="{{item.checked}}" class="ask-for-radio"/>{{item.name}}
      </label>
    </radio-group>
  </view> -->
            <view class="view-line" v-if="radioIndex == 0"></view>
            <view class="remarks-view">
                <view class="remarks-title-input-num">
                    <view class="remarks-title">特殊备注要求</view>
                    <view class="remarks-input-num">{{ inputLength }}/30</view>
                </view>
                <textarea
                    class="textarea-remarks"
                    @input="remarksInput"
                    maxlength="30"
                    placeholder="输入其他特殊备注要求（可不填）"
                    placeholder-class="textarea-placeholder"
                ></textarea>
            </view>
        </view>
        <view class="go-pay-view">
            <view class="go-pay-money">
                <view class="more-pay">
                    <view>还需支付</view>
                </view>
                <view class="total-money" v-if="!data.fullPriceReductionIsHidden && !data.couponsIsHidden && !deliveryAndSelfTaking.isThereADiscount">
                    ￥{{ data.actualPrice }}
                </view>
                <view class="total-money" v-else>￥{{ data.fullPriceReduction }}</view>
            </view>
            <view
                class="go-pay theme-bg"
                hover-class="hover-class-public"
                @tap="parseEventDynamicCode($event, isForgetThePassword ? 'showPwdLayer' : 'getRequestSubscribeMessage')"
            >
                去支付
            </view>
        </view>
        <mp-actionSheet @actiontap="goToPay" :show="dialogShow" :actions="paymentModes" :title="title"></mp-actionSheet>
        <mp-halfScreenDialog :show="showPayPwdInput" extClass="extClassPayPwd" @bindtouchend="bindtouchend" @close="balancePayFail" :maskClosable="maskClosable">
            <view slot="title">输入支付密码</view>
            <view slot="desc">
                <view class="password_dialog_tip"><text>使用会员卡余额支付需要验证身份，验证通过后才可进行支付。</text></view>
                <view class="password_dialog_row" @tap.stop.prevent="getFocus">
                    <view class="password_dialog_item_input" v-for="(item, i) in 6" :key="i">
                        <text v-if="pwdVal.length > i"></text>
                    </view>
                </view>
                <view class="theme-color password_dialog_forget_pwd" @tap.stop.prevent="forgetThePassword">忘记密码</view>
                <input
                    class="password_dialog_input_control"
                    password
                    type="number"
                    :focus="payFocus"
                    :hold-keyboard="holdKeyboard"
                    :value="pwdVal"
                    @input="inputPwd"
                    maxlength="6"
                    :adjust-position="adjustPosition"
                    cursor-spacing="100"
                />
            </view>
            <view slot="footer"></view>
        </mp-halfScreenDialog>
        <mp-dialog
            :show="isVipDialogShow"
            :buttons="buttons"
            :mask-closable="maskClosable"
            @touchmove.native.stop.prevent="trueFun"
            extClass="extClassIsVip"
            :ext-class="extClass"
            @tap.native="close"
        >
            <image
                :src="'https://siam-hangzhou.oss-cn-hangzhou.aliyuncs.com/data/images/business/vip_recharge_guide.png?v=' + timestamp"
                mode="widthFix"
                class="now-order-image"
                @tap="goToRecharge"
            ></image>
        </mp-dialog>
    </view>
</template>

<script>
import mpActionSheet from '../../../components/actionsheet/actionsheet';
import mpHalfScreenDialog from '../../../components/half-screen-dialog/half-screen-dialog';
import mpDialog from '../../../components/dialog/dialog';
import https from '../../../utils/http';
import authService from '../../../utils/auth';
import toastService from '../../../utils/toast.service';
import systemStatus from '../../../utils/system-status';
import dateHelper from '../../../utils/date-helper';
import utilHelper from '../../../utils/util';
import base64 from '../../../utils/base64';
const app = getApp();
var wxNotifyTemplates = [];
export default {
    components: {
        mpActionSheet,
        mpHalfScreenDialog,
        mpDialog
    },
    data() {
        return {
            time: '10:00',
            isChoose: false,

            //是否选择派送方式
            isFirstShop: false,

            //是否选择的是门店
            isFirstAddress: false,

            inputLength: 0,
            deliveryData: {},
            remarks: '',

            modeTabList: [
                {
                    modeName: '外卖配送',
                    modeId: 0
                },
                {
                    modeName: '门店自取',
                    modeId: 1
                }
            ],

            currentTab: 0,
            reducedDeliveryPrice: 0,
            title: '请选择支付方式',

            // paymentModes: [{
            //   checked: true,
            //   value: 1,
            //   text: '微信支付',
            //   icon: 'iconwechat_pay',
            //   show:true,
            // }, {
            //   checked: false,
            //   value: 2,
            //   text: '平台余额',
            //   icon: 'iconyue',
            //   show:true,
            // }],
            paymentModes: [
                {
                    checked: false,
                    value: 1,
                    text: '微信支付',
                    icon: 'iconwechat_pay',
                    show: true
                },
                {
                    checked: true,
                    value: 2,
                    text: '平台余额',
                    icon: 'iconyue',
                    show: true
                }
            ],

            paymentModeIndex: 1,
            showPayPwdInput: false,

            //是否展示密码输入层
            pwdVal: '',

            //输入的密码
            payFocus: false,

            //文本框焦点
            adjustPosition: false,

            holdKeyboard: true,
            isVipDialogShow: false,

            deliveryAndSelfTaking: {
                isReducedDeliveryPrice: false,
                currentTab: '',
                reducedDeliveryTotalPrice: '',
                feeData: '',
                isThereADiscount: '',
                reducedDeliveryTotalPric: '',

                deliveryAddress: {
                    province: false,
                    city: false,
                    area: false,
                    street: '',
                    phone: '',
                    realname: '',
                    sex: 0
                }
            },

            data: {
                actualPrice: '',
                fullPriceReduction: '',
                couponsIsHidden: '',
                orderDetailList: [],
                packingCharges: '',
                fullPriceReductionIsHidden: '',
                fullReductionRuleName: ''
            },

            payType: '',
            initData: '',
            isJs: false,
            timestamp: '',
            userInfo: '',
            dialogShow: false,

            shopAddress: {
                shop: {
                    name: '',
                    province: false,
                    city: false,
                    area: false,
                    street: ''
                }
            },

            afterDiscount: {
                couponsName: false,
                price: ''
            },

            tipReward: '',
            isPayJson: '',
            balanceId: '',
            balanceOrderNo: '',
            balanceActualPrice: '',
            balanceOpenId: '',
            phoneNumber: 0,
            radioIndex: 0,
            maskClosable: '',
            i: '',
            buttons: '',
            extClass: ''
        };
    },
    /**
     * 生命周期函数--监听页面加载
     */
    onLoad: function (options) {
        this.setData({
            deliveryAndSelfTaking: app.globalData.deliveryAndSelfTaking
        });
        let data = JSON.parse(options.orderDetail);
        console.log(app.globalData.deliveryAndSelfTaking);
        console.log(data);
        if (app.globalData.deliveryAndSelfTaking.deliveryAddress && app.globalData.deliveryAndSelfTaking.currentTab == 0) {
            //选择地址和更换地址进行配送费的加减操作
            https
                .request('/api-order/rest/common/selectDeliveryFee', {
                    deliveryAddressId: app.globalData.deliveryAndSelfTaking.deliveryAddress.id,
                    shopId: data.shopId
                })
                .then((result) => {
                    console.log(result);
                    app.globalData.deliveryAndSelfTaking.reducedDeliveryTotalPrice = result.data;
                    app.globalData.deliveryAndSelfTaking.feeData = result.data;
                    app.globalData.deliveryAndSelfTaking.actualPrice = data.actualPrice + result.data;
                    data.actualPrice = data.actualPrice + result.data;
                    this.setData({
                        deliveryAndSelfTaking: app.globalData.deliveryAndSelfTaking,
                        'deliveryAndSelfTaking.isReducedDeliveryPrice': true
                    });
                    this.getCouponsMemberRelation(data);
                });
        } else {
            // this.setData({
            //   "deliveryAndSelfTaking.deliveryAddress": null
            // });
            //data.actualPrice = data.actualPrice + app.deliveryAndSelfTaking.reducedDeliveryTotalPrice;
            this.getCouponsMemberRelation(data);
        }
        let time = dateHelper.formatTime('hms');
        this.setData({
            data: data,
            payType: options.payType,
            time: time,
            initData: data,
            isJs: true
        });
    },
    /**
     * 生命周期函数--监听页面初次渲染完成
     */
    onReady: function () {
        console.log('onReady');
        this.getShopAddressList();
    },
    /**
     * 生命周期函数--监听页面显示
     */
    onShow: function () {
        if (this.showPayPwdInput) {
            this.showPwdLayer();
        }
        this.getLoginMemberInfo();
        //console.log(this.data)
        if (app.globalData.loginUserInfo.isRequestWxNotify) {
            this.getWxNotifyTemplate();
        }
        setTimeout((time) => {
            this.selectCommissionReward();
            clearTimeout(time);
        }, 2000);
        this.getTimestamp();
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
    onReachBottom: function () {},
    /**
     * 用户点击右上角分享
     */
    onShareAppMessage: function () {},
    methods: {
        getTimestamp() {
            var timestamp = dateHelper.getTimestamp();
            console.log(timestamp);
            this.setData({
                timestamp: timestamp
            });
        },

        // 滑动切换tab
        bindSlideChange: function (e) {
            var that = this;
            that.setData({
                currentTab: e.detail.current
            });
        },

        //点击切换
        clickTab: function (e) {
            var that = this;
            app.globalData.deliveryAndSelfTaking.radioIndex = e.target.dataset.current;
            let time = dateHelper.formatTime('hms');
            if (that.deliveryAndSelfTaking.currentTab === e.target.dataset.current) {
                return false;
            } else {
                let feeData = app.globalData.deliveryAndSelfTaking.feeData;
                let actualPrice = this.data.actualPrice;
                let fullPriceReduction = this.data.fullPriceReduction;
                if (e.target.dataset.current == 1) {
                    actualPrice = actualPrice - feeData;
                    fullPriceReduction = fullPriceReduction - feeData;
                } else {
                    actualPrice = actualPrice + feeData;
                    fullPriceReduction = fullPriceReduction + feeData;
                }
                that.setData({
                    'deliveryAndSelfTaking.currentTab': e.target.dataset.current,
                    time: time,
                    'data.actualPrice': utilHelper.toFixed(actualPrice, 2),
                    'data.fullPriceReduction': utilHelper.toFixed(fullPriceReduction, 2)
                });
            }
        },

        radioChange(e) {
            let checkValue = e.currentTarget.dataset.radioindex;
            let deliveryData = JSON.stringify(this.deliveryAndSelfTaking);
            console.log(this);
            uni.navigateTo({
                url:
                    '../../address/choose/choose?radioIndex=' +
                    checkValue +
                    '&deliveryData=' +
                    deliveryData +
                    '&shopId=' +
                    this.initData.shopId +
                    '&reducedDeliveryPrice=' +
                    this.reducedDeliveryPrice +
                    '&pageType=pay'
            });
        },

        getLoginMemberInfo: function (e) {
            toastService.showLoading();
            https.request('/api-user/rest/member/getLoginMemberInfo', {}).then((result) => {
                if (result.success) {
                    result.data.typeVipText = systemStatus.typeVipText(result.data.vipType);
                    result.data.statusVipText = systemStatus.statusVipText(result.data.vipStatus);
                    result.data.vipStartTime = dateHelper.formatDate(result.data.vipStartTime);
                    result.data.vipEndTime = dateHelper.formatDate(result.data.vipEndTime);
                    this.setData({
                        userInfo: result.data
                    });
                    setTimeout((time) => {
                        this.choosePayModeTap();
                        clearTimeout(time);
                    }, 1000);
                }
            });
        },

        closeDialog: function () {
            this.setData({
                dialogShow: false
            });
        },

        close() {
            this.setData({
                isVipDialogShow: false
            });
            toastService.showLoading();
            var _this = this;
            https.request('/api-order/rest/member/order/insert', _this.isPayJson).then((result) => {
                toastService.hideLoading();
                if (result.success) {
                    // toastService.showLoading("正在支付...", true);
                    //console.log(result.data)
                    _this.toPay4Applet(result.data.id, result.data.orderNo, result.data.actualPrice);
                }
            });
        },

        btnClick(e) {
            console.log(e);
            this.closeDialog();
        },

        radioChangeAddress(e) {
            console.log(e);
            var that = this;
            var paymentModeIndex = e.detail.value;
            authService.getOpenId().then((openId) => {
                console.log(openId);
                that.closeDialog();
                // if (!openId) {
                //   toastService.showToast("登录用户错误，请重新登录");
                //   return
                // }
                let paymentModes = that.paymentModes;
                // if (e.detail.value == 0) {
                //   toastService.showToast("暂不支持微信支付，请选择余额支付/积分支付");
                //   return
                // }
                if (e.detail.value == 1) {
                    console.log(that.userInfo.paymentPassword);
                    if (!that.userInfo.paymentPassword) {
                        uni.navigateTo({
                            url: '../../mine/security/index/index'
                        });
                        return;
                    }
                    if (!paymentModes[e.detail.value].isBindTap) {
                        toastService.showToast(paymentModes[e.detail.value].desc);
                        return;
                    }
                }
                for (let key in paymentModes) {
                    paymentModes[key].checked = false;
                }
                paymentModes[e.detail.value].checked = true;
                that.setData({
                    paymentModes: paymentModes,
                    paymentModeIndex: e.detail.value
                });
            });
        },

        choosePayModeTap() {
            let paymentModes = this.paymentModes;
            let paymentModeIndex = this.paymentModeIndex;
            let actualPrice =
                !this.data.fullPriceReductionIsHidden && !this.data.couponsIsHidden && !this.deliveryAndSelfTaking.isThereADiscount
                    ? this.data.actualPrice
                    : this.data.fullPriceReduction;
            console.log('余额=', this.userInfo.balance);
            console.log('需支付金额=', actualPrice);
            if (paymentModes[paymentModeIndex].value == 2) {
                paymentModes[paymentModeIndex].desc = '';
                paymentModes[paymentModeIndex].isBindTap = true;
                if (actualPrice > this.userInfo.balance) {
                    paymentModes[paymentModeIndex].desc = '余额不足';
                    paymentModes[paymentModeIndex].isBindTap = false;
                }
                if (!this.userInfo.paymentPassword) {
                    paymentModes[paymentModeIndex].desc = '未设置支付密码,去设置';
                    paymentModes[paymentModeIndex].isBindTap = false;
                }
            }

            // this.setData({
            //   dialogShow: true,
            //   maskClosable: false,
            //   title: "请选择支付方式",
            //   paymentModes: paymentModes
            // })
            toastService.hideLoading();
            this.setData({
                paymentModes: paymentModes
            });
        },

        remarksInput(e) {
            this.setData({
                inputLength: e.detail.value.length,
                remarks: e.detail.value
            });
        },

        getShopAddressList() {
            https
                .request('/api-merchant/rest/shop/detail', {
                    id: this.initData.shopId,
                    position: app.globalData.deliveryAndSelfTaking.location
                })
                .then((result) => {
                    if (result.success) {
                        this.setData({
                            shopAddress: result.data,
                            reducedDeliveryPrice: result.data.shop.reducedDeliveryPrice
                        });
                        this.getPsf();
                    }
                });
        },

        getCouponsMemberRelation(pageData) {
            toastService.showLoading();
            https
                .request('/api-promotion/rest/member/couponsMemberRelation/list', {
                    pageNo: -1,
                    pageSize: 20,
                    isUsed: 0,
                    isExpired: 0,
                    isValid: 0
                })
                .then((result) => {
                    toastService.hideLoading();
                    if (result.success) {
                        let afterDiscounts = [];
                        result.data.records.forEach((res) => {
                            //判断优惠券是否过期和是否已经使用
                            //if (!res.couponsMemberRelationMap.isExpired && !res.couponsMemberRelationMap.isUsed) {
                            //console.log(this.data)
                            //判断是否已经满减，如果已经满减就取满减之后的字段值，反之取总额
                            //if (this.data.data.actualPrice && !this.data.data.fullPriceReduction) {

                            //判断当前优惠券是折扣还是满减券,1等于折扣,2等于满减
                            if (res.couponsMemberRelationMap.preferentialType == 1) {
                                if (res.shopList.length <= 0) {
                                    return;
                                }
                                res.shopList.forEach((shop, shopIndex) => {
                                    if (shop.id == this.data.shopId) {
                                        //遍历当前订单的商品
                                        this.data.orderDetailList.forEach((order, orderIndex) => {
                                            //source  优惠券发放来源 1=商家中心 2=调度中心
                                            //如果是商家中心发放的优惠券，则需要判断关联商品
                                            //调度中心发放的优惠券，则无需判断关联商品，所有商品皆可使用
                                            console.log(order.price);
                                            let orderPrice = order.price;
                                            let unitPrice = utilHelper.toFixed(orderPrice - orderPrice * res.couponsMemberRelationMap.discountAmount, 2);
                                            if (res.couponsMemberRelationMap.source == 1) {
                                                res.goodsList.forEach((goods) => {
                                                    //判断当前订单的商品是否等于优惠券绑定的优惠商品
                                                    //等于则进行优惠
                                                    if (order.goodsId == goods.id) {
                                                        afterDiscounts.push({
                                                            id: res.couponsMemberRelationMap.id,
                                                            price: res.couponsMemberRelationMap.discountAmount != 0 ? unitPrice : unitPrice,
                                                            goodsId: order.goodsId,
                                                            couponsId: res.couponsMemberRelationMap.couponsId,
                                                            couponsName: res.couponsMemberRelationMap.couponsName,
                                                            preferentialType: systemStatus.preferentialTypeText(res.couponsMemberRelationMap.preferentialType),
                                                            isExpired: res.couponsMemberRelationMap.isExpired,
                                                            isUsed: res.couponsMemberRelationMap.isUsed,
                                                            isValid: res.couponsMemberRelationMap.isValid
                                                        });
                                                    }
                                                });
                                                return;
                                            }
                                            afterDiscounts.push({
                                                id: res.couponsMemberRelationMap.id,
                                                price: res.couponsMemberRelationMap.discountAmount != 0 ? unitPrice : unitPrice,
                                                goodsId: order.goodsId,
                                                couponsId: res.couponsMemberRelationMap.couponsId,
                                                couponsName: res.couponsMemberRelationMap.couponsName,
                                                preferentialType: systemStatus.preferentialTypeText(res.couponsMemberRelationMap.preferentialType),
                                                isExpired: res.couponsMemberRelationMap.isExpired,
                                                isUsed: res.couponsMemberRelationMap.isUsed,
                                                isValid: res.couponsMemberRelationMap.isValid
                                            });
                                        });
                                    }
                                });
                            }
                            //判断如果是优惠券满减的话这就进行优惠券的总价满减
                            //console.log(res)
                            if (res.couponsMemberRelationMap.preferentialType == 2) {
                                if (this.data.fullPriceReduction) {
                                    if (this.data.fullPriceReduction >= res.couponsMemberRelationMap.limitedPrice) {
                                        //console.log(res.couponsMemberRelationMap.limitedPrice)
                                        afterDiscounts.push({
                                            id: res.couponsMemberRelationMap.id,
                                            couponsId: res.couponsMemberRelationMap.couponsId,
                                            price: utilHelper.toFixed(this.data.fullPriceReduction, 2) - res.couponsMemberRelationMap.reducedPrice,
                                            couponsName: res.couponsMemberRelationMap.couponsName,
                                            preferentialType: systemStatus.preferentialTypeText(res.couponsMemberRelationMap.preferentialType),
                                            isExpired: res.couponsMemberRelationMap.isExpired,
                                            isUsed: res.couponsMemberRelationMap.isUsed,
                                            isValid: res.couponsMemberRelationMap.isValid
                                        });
                                    }
                                    return;
                                }
                                if (this.data.actualPrice >= res.couponsMemberRelationMap.limitedPrice) {
                                    afterDiscounts.push({
                                        id: res.couponsMemberRelationMap.id,
                                        couponsId: res.couponsMemberRelationMap.couponsId,
                                        price: this.data.actualPrice - res.couponsMemberRelationMap.reducedPrice,
                                        couponsName: res.couponsMemberRelationMap.couponsName,
                                        preferentialType: systemStatus.preferentialTypeText(res.couponsMemberRelationMap.preferentialType),
                                        isExpired: res.couponsMemberRelationMap.isExpired,
                                        isUsed: res.couponsMemberRelationMap.isUsed,
                                        isValid: res.couponsMemberRelationMap.isValid
                                    });
                                }
                            }
                            //}
                            //}
                        });
                        //console.log(afterDiscounts)
                        //如果优惠券的使用张数大于0张
                        let fullPriceReduction = this.data.actualPrice;
                        var afterDiscountPrice = 0;
                        var price = 0;
                        var afterDiscountList;
                        var couponsIsHidden = false;
                        if (afterDiscounts.length > 0) {
                            //遍历所有可以使用优惠券的商品，并计算出最大的优惠券
                            afterDiscounts.forEach((afterDiscount) => {
                                price = afterDiscount.price;
                                if (Number(price) >= afterDiscountPrice) {
                                    afterDiscountPrice = price;
                                    afterDiscountList = afterDiscount;
                                }
                            });
                            fullPriceReduction = this.data.fullPriceReduction ? this.data.fullPriceReduction - afterDiscountPrice : this.data.actualPrice - afterDiscountPrice;
                            couponsIsHidden = true;
                        }
                        this.setData({
                            afterDiscount: afterDiscountList,
                            'data.couponsIsHidden': couponsIsHidden,
                            'data.fullPriceReduction': utilHelper.toFixed(fullPriceReduction <= 0 ? 0 : fullPriceReduction, 2)
                        });
                        this.getFullReductionRule();
                    }
                });
        },

        onCoupon() {
            let afterDiscount = this.afterDiscount;
            afterDiscount.fullPriceReduction = this.data.fullPriceReduction;
            afterDiscount.actualPrice = this.data.actualPrice;
            afterDiscount.fullPriceReductionIsHidden = this.data.fullPriceReductionIsHidden;
            afterDiscount.orderDetailList = this.data.orderDetailList;
            afterDiscount.fullPriceReductionAfter = this.data.fullPriceReductionAfter;
            afterDiscount.shopId = this.data.shopId;
            afterDiscount.limitedPrice = this.data.limitedPrice;
            afterDiscount.fullReductionRuleName = this.data.fullReductionRuleName;
            afterDiscount.fullReductionRuleId = this.data.fullReductionRuleId;
            afterDiscount.fullPriceReductionIsHidden = this.data.fullPriceReductionIsHidden;
            afterDiscount.packingCharges = this.data.packingCharges;
            afterDiscount.feeData = this.deliveryAndSelfTaking.feeData;
            afterDiscount.type = 1;
            //console.log(afterDiscount)
            uni.navigateTo({
                url: '../../mine/coupons/coupons?prevData=' + JSON.stringify(afterDiscount)
            });
        },

        selectCommissionReward() {
            console.log(this);
            https
                .request('/api-order/rest/member/order/selectCommissionReward', {
                    actualPrice: !this.data.fullPriceReductionIsHidden && !this.data.couponsIsHidden ? this.data.actualPrice : this.data.fullPriceReduction
                })
                .then((result) => {
                    if (result.success) {
                        this.setData({
                            tipReward: result.data
                        });
                    }
                });
        },

        getWxNotifyTemplate() {
            https.request('/api-order/rest/wxNotifyTemplate/orderModule/list').then((result) => {
                if (result.success) {
                    console.log('订单模块的模板id===>' + result.data);
                    wxNotifyTemplates = result.data;
                }
            });
        },

        updateIsRequestWxNotify() {
            https.request('/api-user/rest/member/updateIsRequestWxNotify').then((result) => {
                if (result.success) {
                    console.log('修改用户的是否需要请求授权服务通知状态为否成功');
                }
            });
        },

        getRequestSubscribeMessage() {
            let self = this;
            console.log('用户的是否需要请求授权服务通知====>' + app.globalData.loginUserInfo.isRequestWxNotify);
            if (this.paymentModeIndex == 1) {
                console.log(this.userInfo.paymentPassword);
                if (!this.userInfo.paymentPassword) {
                    uni.navigateTo({
                        url: '../../mine/security/index/index'
                    });
                    return;
                }
            }
            if (app.globalData.loginUserInfo.isRequestWxNotify) {
                uni.requestSubscribeMessage({
                    tmplIds: wxNotifyTemplates,
                    success(res) {
                        console.log('订单模块的模板授权成功');
                        console.log(res);
                        self.weChatPayTap();
                        self.updateIsRequestWxNotify();
                    },
                    fail(error) {
                        console.log('订单模块的模板授权失败');
                        console.log(error);
                        self.weChatPayTap();
                    }
                });
                return;
            }
            self.weChatPayTap();
        },

        weChatPayTap() {
            //判断店铺是否打烊
            // app.getIsBusiness().then(result => {
            //   if (!result) {
            //     return
            //   }

            authService.checkIsLogin().then((result) => {
                if (result) {
                    //判断店铺是否打烊
                    let startTime = this.shopAddress.shop.startTime;
                    let endTime = this.shopAddress.shop.endTime;
                    let isOperating = this.shopAddress.shop.isOperating;
                    app.globalData.getIsBusiness(startTime, endTime, isOperating).then((result) => {
                        if (!result) {
                            return;
                        }
                        var data = {};
                        if (this.deliveryAndSelfTaking.currentTab == 1) {
                            if (this.shopAddress.shop == null) {
                                toastService.showToast('请选择门店地址');
                                return;
                            }
                            data.shopId = this.shopAddress.shop.id;
                        }
                        if (this.deliveryAndSelfTaking.currentTab == 0) {
                            if (this.deliveryAndSelfTaking.deliveryAddress == null) {
                                toastService.showToast('请选择配送地址');
                                return;
                            }
                            data.deliveryAddressId = this.deliveryAndSelfTaking.deliveryAddress.id;
                            data.deliveryFee = this.deliveryAndSelfTaking.feeData;
                        }
                        var list = this.data.orderDetailList;
                        var orderDetailList = [];
                        data.shoppingCartIdList = [];
                        for (var key in list) {
                            orderDetailList.push({
                                goodsId: list[key].goodsId,
                                goodsName: list[key].goodsName,
                                specList: list[key].specList,
                                number: list[key].number
                            });
                            if (this.payType) {
                                data.shoppingCartIdList.push(list[key].id);
                            }
                        }
                        //console.log(this.data)
                        data.orderDetailListStr = JSON.stringify(orderDetailList);
                        data.actualPrice =
                            this.data.fullPriceReductionIsHidden || this.data.couponsIsHidden || this.deliveryAndSelfTaking.isThereADiscount
                                ? this.data.fullPriceReduction
                                : this.data.actualPrice;
                        //data.actualPrice = data.actualPrice + this.data.feeData;
                        data.remark = this.remarks;
                        data.shoppingWay = this.deliveryAndSelfTaking.currentTab == 0 ? 2 : 1;
                        data.shopId = this.initData.shopId;
                        //console.log(data)
                        if (this.data.couponsIsHidden) {
                            data.couponsId = this.afterDiscount.couponsId;
                            data.couponsMemberRelationId = this.afterDiscount.id;
                            data.couponsDescription = this.afterDiscount.couponsName;
                        }
                        //console.log(this.data)
                        if (this.data.fullReductionRuleId) {
                            data.fullReductionRuleId = this.data.fullReductionRuleId;
                            data.fullReductionRuleDescription = this.data.fullReductionRuleName;
                        }
                        var that = this;
                        //获取订单的配送方式
                        let orderMode = this.modeTabList[this.currentTab].modeName;
                        let addressMode =
                            this.deliveryAndSelfTaking.currentTab == 1
                                ? orderMode + '门店：' + this.shopAddress.shop.name
                                : orderMode +
                                  '地址：' +
                                  this.deliveryAndSelfTaking.deliveryAddress.province +
                                  this.deliveryAndSelfTaking.deliveryAddress.city +
                                  this.deliveryAndSelfTaking.deliveryAddress.area +
                                  this.deliveryAndSelfTaking.deliveryAddress.street;
                        // toastService.showModal("【" + orderMode + "】订单", "订单确认后将无法更改!\r\n" + addressMode, function comfirm() {
                        //   toastService.showLoading();
                        //   https.request('/api-order/rest/member/order/insert', data).then(result => {
                        //     toastService.hideLoading();
                        //     if (result.success) {
                        //       // toastService.showLoading("正在支付...", true);
                        //       //console.log(result.data)
                        //       that.toPay4Applet(result.data.id, result.data.orderNo, result.data.actualPrice);
                        //     }
                        //   })
                        // }, null)
                        if (this.userInfo.type == 1) {
                            this.setData({
                                isVipDialogShow: true,
                                isPayJson: data
                            });
                        } else {
                            toastService.showLoading();
                            https.request('/api-order/rest/member/order/insert', data).then((result) => {
                                toastService.hideLoading();
                                if (result.success) {
                                    // toastService.showLoading("正在支付...", true);
                                    //console.log(result.data)
                                    that.toPay4Applet(result.data.id, result.data.orderNo, result.data.actualPrice);
                                }
                            });
                        }
                    });
                    return;
                }
                app.globalData.checkIsAuth('scope.userInfo');
            });
        },

        toPay4Applet(id, orderNo, actualPrice) {
            toastService.showLoading('正在加载...', true);
            authService.getOpenId().then((openId) => {
                //console.log(openId)
                // if (!openId) {
                //   toastService.showToast("登录用户错误，请重新登录");
                //   return
                // }
                openId = this.userInfo.openId;
                if (this.paymentModes[this.paymentModeIndex].value == 1) {
                    this.weChatPay(id, orderNo, actualPrice, openId);
                }
                if (this.paymentModes[this.paymentModeIndex].value == 2) {
                    this.setData({
                        showPayPwdInput: true,
                        payFocus: true
                    });
                    toastService.hideLoading();
                    this.setData({
                        balanceId: id,
                        balanceOrderNo: orderNo,
                        balanceActualPrice: actualPrice,
                        balanceOpenId: openId
                    });
                    // if (this.data.pwdVal.length >= 6){
                    //   this.balancePay(id, orderNo, actualPrice,openId);
                    // }
                }
            });
        },

        weChatPay(id, orderNo, actualPrice, openId) {
            https
                .request('/api-order/rest/member/wxPay/toPay4Applet', {
                    openid: openId,
                    type: 1,
                    paymentMode: 1,
                    out_trade_no: orderNo,
                    total_fee: this.data.fullPriceReductionIsHidden ? this.data.fullPriceReduction : actualPrice
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
                                toastService.showSuccess('支付成功', true);
                                let timeout = setTimeout(() => {
                                    uni.redirectTo({
                                        url: '../../order/detail/detail?id=' + id
                                    });
                                    clearTimeout(timeout);
                                }, 1000);
                            },
                            fail(res) {
                                toastService.showError('支付失败', true);
                                let timeout = setTimeout(() => {
                                    uni.redirectTo({
                                        url: '../../order/detail/detail?id=' + id
                                    });
                                    clearTimeout(timeout);
                                }, 1000);
                            }
                        });
                    }
                });
        },

        inputPwd: function (e) {
            this.setData({
                pwdVal: e.detail.value
            });
            if (e.detail.value.length >= 6) {
                toastService.showLoading('正在加载...');
                this.balancePay();
            }
        },

        /**
         * 获取焦点
         */
        getFocus: function () {
            this.setData({
                payFocus: true
            });
        },

        balancePay() {
            var password = base64.encode(this.pwdVal);
            this.setData({
                pwdVal: '',
                payFocus: true
            });
            https
                .request('/api-order/rest/member/platformPay/byBalance', {
                    openid: this.balanceOpenId,
                    type: 1,
                    paymentMode: 2,
                    paymentPassword: password,
                    out_trade_no: this.balanceOrderNo,
                    total_fee: this.data.fullPriceReductionIsHidden ? this.data.fullPriceReduction : this.balanceActualPrice
                })
                .then((result) => {
                    toastService.hideLoading();
                    if (result.success) {
                        toastService.showSuccess('支付成功', true);
                        this.hidePwdLayer();
                        let timeout = setTimeout(() => {
                            uni.redirectTo({
                                url: '../../order/detail/detail?id=' + this.balanceId
                            });
                            clearTimeout(timeout);
                        }, 1000);
                    }
                });
        },

        bindtouchend(e) {
            console.log('触摸结束');
            this.showPwdLayer();
        },

        balancePayFail() {
            this.hidePwdLayer();
            toastService.showError('支付失败', true);
            let timeout = setTimeout(() => {
                uni.redirectTo({
                    url: '../../order/detail/detail?id=' + this.balanceId
                });
                clearTimeout(timeout);
            }, 1000);
        },

        hidePwdLayer() {
            this.setData({
                showPayPwdInput: false,
                payFocus: false,
                pwdVal: ''
            });
        },

        showPwdLayer() {
            this.setData({
                showPayPwdInput: true,
                payFocus: true,
                pwdVal: ''
            });
        },

        /**
         * 隐藏支付密码输入层
         */
        forgetThePassword: function () {
            uni.navigateTo({
                url: '../../mine/security/index/index'
            });
        },

        //获取满减规则
        getFullReductionRule(data) {
            toastService.showLoading();
            https
                .request('/api-promotion/rest/fullReductionRule/list', {
                    pageNo: -1,
                    pageSize: 1,
                    shopId: this.data.shopId
                })
                .then((result) => {
                    toastService.hideLoading();
                    if (result.success) {
                        //获取配送费，配送费不作为满减条件
                        let feeData = app.globalData.deliveryAndSelfTaking.reducedDeliveryTotalPrice ? app.globalData.deliveryAndSelfTaking.reducedDeliveryTotalPrice : 0;
                        // if (app.deliveryAndSelfTaking.currentTab == 1) {
                        //   feeData = app.deliveryAndSelfTaking.feeData ? app.deliveryAndSelfTaking.feeData : 0;
                        // }

                        if (!this.isPreviousPage) {
                            this.data.packingCharges = 0;
                            for (var key in this.data.orderDetailList) {
                                this.data.orderDetailList[key].totalPrice = utilHelper.toFixed(this.data.orderDetailList[key].price * this.data.orderDetailList[key].number, 2);
                                this.data.packingCharges = utilHelper.toFixed(
                                    this.data.packingCharges + this.data.orderDetailList[key].packingCharges * this.data.orderDetailList[key].number
                                );
                            }
                        }
                        let reducedDeliveryPrice = 0;
                        console.log('获取商家配送费=====' + this.reducedDeliveryPrice);
                        if (this.reducedDeliveryPrice >= feeData) {
                            reducedDeliveryPrice = 0;
                        } else {
                            reducedDeliveryPrice = feeData - this.reducedDeliveryPrice;
                        }
                        console.log('获取用户地址配送费====' + feeData);
                        console.log('获取根据商家的配送费得出最终配送费====' + reducedDeliveryPrice);
                        console.log('获取优惠券后的价格======' + this.data.fullPriceReduction);
                        let isReducedDeliveryPrice = false;
                        let isThereADiscount;
                        if (feeData != reducedDeliveryPrice) {
                            isReducedDeliveryPrice = true;
                            isThereADiscount = true;
                        }
                        this.data.actualPrice = this.data.actualPrice ? utilHelper.toFixed(Number(this.data.actualPrice), 2) : 0;
                        this.data.fullPriceReductionAfter = this.data.actualPrice;
                        this.data.discountPrice = 0;
                        this.data.fullPriceReductionBefore = this.data.fullPriceReduction > 0 ? utilHelper.toFixed(this.data.fullPriceReduction - feeData, 2) : 0;
                        console.log('获取减去配送费优惠价格1======' + this.data.fullPriceReductionBefore);
                        this.data.fullPriceReductionIsHidden = this.data.fullPriceReductionIsHidden;
                        this.data.limitedPrice = 0;
                        this.data.fullPriceReduction = this.data.fullPriceReductionBefore;
                        for (let i = 0; i < result.data.records.length; i++) {
                            //总价格减去配送费大于满减金额则进行满减操作
                            if (Number(this.data.fullPriceReductionBefore) >= result.data.records[i].limitedPrice) {
                                //判断当前满减价格limitedPrice和上一个满减价格对比，如果大于就进行认证
                                if (result.data.records[i].limitedPrice > this.data.limitedPrice) {
                                    this.data.limitedPrice = result.data.records[i].limitedPrice;
                                    this.data.fullPriceReduction = utilHelper.toFixed(this.data.fullPriceReductionBefore - result.data.records[i].reducedPrice, 2);
                                    this.data.fullReductionRuleName = result.data.records[i].name;
                                    this.data.fullReductionRuleId = result.data.records[i].id;
                                    this.data.fullPriceReductionIsHidden = true;
                                    this.data.reducedPrice = result.data.records[i].reducedPrice;
                                    this.data.fullPriceReductionAfter = utilHelper.toFixed(this.data.fullPriceReductionBefore - result.data.records[i].reducedPrice, 2);
                                }
                            }
                        }
                        //console.log(reducedDeliveryPrice)
                        //console.log(this.data.data.fullPriceReduction - feeData);
                        if (app.globalData.deliveryAndSelfTaking.currentTab == 0) {
                            this.data.fullPriceReduction = this.data.fullPriceReduction + reducedDeliveryPrice;
                        }
                        console.log('获取最终满减后的价格' + this.data.fullPriceReduction);
                        this.setData({
                            data: this.data,
                            'deliveryAndSelfTaking.isReducedDeliveryPrice': isReducedDeliveryPrice,
                            'deliveryAndSelfTaking.reducedDeliveryTotalPrice': feeData,
                            'deliveryAndSelfTaking.feeData': reducedDeliveryPrice,
                            'deliveryAndSelfTaking.isThereADiscount': isThereADiscount
                        });
                        // this.selectCommissionReward();
                    }
                });
        },

        //获取满减规则
        getFullReductionRuleMode(data) {
            toastService.showLoading();
            https
                .request('/api-promotion/rest/fullReductionRule/list', {
                    pageNo: -1,
                    pageSize: 1,
                    shopId: data.shopId
                })
                .then((result) => {
                    toastService.hideLoading();
                    if (result.success) {
                        let feeData = data.feeData;
                        let reducedDeliveryTotalPrice = feeData;
                        let isReducedDeliveryPrice = false;
                        let actualPrice = this.data.actualPrice;
                        let fullPriceReductionBefore = actualPrice - feeData;
                        //data.actualPrice=this.data.data.actualPrice;
                        console.log('总商品金额' + actualPrice);
                        console.log('总商品金额fullPriceReductionBefore' + fullPriceReductionBefore);
                        let couponPrice = this.afterDiscount ? this.afterDiscount.price : 0;
                        let couponAfter = 0;
                        let reducedDeliveryPrice = this.reducedDeliveryPrice;
                        console.log('用户地址配送费======' + feeData);
                        console.log('商家配送费======' + reducedDeliveryPrice);
                        let reducedPrice = 0;
                        if (reducedDeliveryPrice >= data.feeData) {
                            couponAfter = utilHelper.toFixed(fullPriceReductionBefore - couponPrice, 2);
                            reducedPrice = 0;
                        } else {
                            reducedPrice = feeData - reducedDeliveryPrice;
                            couponAfter = utilHelper.toFixed(fullPriceReductionBefore - couponPrice, 2);
                        }
                        if (reducedDeliveryTotalPrice != reducedPrice) {
                            isReducedDeliveryPrice = true;
                        }
                        //actualPrice = actualPrice-reducedPrice;
                        //couponAfter=actualPrice-reducedPrice;
                        console.log('计算商家后的配送费======' + reducedPrice);
                        console.log('总商品金额+上了配送费===' + actualPrice);
                        console.log('优惠券金额=======' + couponPrice);
                        console.log('计算后的价格=======' + couponAfter);
                        data.limitedPrice = 0;
                        data.fullPriceReduction = couponAfter;
                        for (let i = 0; i < result.data.records.length; i++) {
                            //console.log(result.data.records[i].limitedPrice)
                            //总价格减去配送费大于满减金额则进行满减操作
                            if (Number(couponAfter) >= result.data.records[i].limitedPrice) {
                                //判断当前满减价格limitedPrice和上一个满减价格对比，如果大于就进行认证
                                if (result.data.records[i].limitedPrice > data.limitedPrice) {
                                    data.limitedPrice = result.data.records[i].limitedPrice;
                                    data.fullPriceReduction = utilHelper.toFixed(couponAfter - result.data.records[i].reducedPrice, 2);
                                    data.fullReductionRuleName = result.data.records[i].name;
                                    data.fullReductionRuleId = result.data.records[i].id;
                                    data.fullPriceReductionIsHidden = true;
                                    data.reducedPrice = result.data.records[i].reducedPrice;
                                    data.fullPriceReductionAfter = utilHelper.toFixed(this.data.fullPriceReduction - result.data.records[i].reducedPrice, 2);
                                }
                            }
                        }
                        console.log('获取最终配送费=========' + reducedPrice);
                        console.log('获取最终价格=========' + data.fullPriceReduction);
                        data.fullPriceReduction = data.fullPriceReduction + reducedPrice;
                        app.globalData.deliveryAndSelfTaking.feeData = reducedPrice;
                        app.globalData.deliveryAndSelfTaking.isThereADiscount = false;
                        if (reducedDeliveryTotalPrice != reducedPrice) {
                            app.globalData.deliveryAndSelfTaking.isThereADiscount = true;
                        }
                        console.log('获取最终价格再加上配送费=========' + data.fullPriceReduction);
                        console.log('获取最终价格=========' + data.actualPrice);
                        this.setData({
                            data,
                            'deliveryAndSelfTaking.reducedDeliveryTotalPrice': reducedDeliveryTotalPrice,
                            'deliveryAndSelfTaking.isThereADiscount': app.globalData.deliveryAndSelfTaking.isThereADiscount,
                            'deliveryAndSelfTaking.isReducedDeliveryPrice': isReducedDeliveryPrice,
                            'deliveryAndSelfTaking.feeData': reducedPrice
                        });
                    }
                });
        },

        getPsf() {
            let isReducedDeliveryPrice = false;
            let feeData = app.globalData.deliveryAndSelfTaking.reducedDeliveryTotalPrice;
            let reducedDeliveryTotalPrice = app.globalData.deliveryAndSelfTaking.reducedDeliveryTotalPrice;
            console.log('用户地址配送费' + reducedDeliveryTotalPrice);
            if (this.reducedDeliveryPrice >= reducedDeliveryTotalPrice) {
                feeData = 0;
            } else {
                feeData = feeData - this.reducedDeliveryPrice;
            }
            console.log('用户最终支付的配送费' + feeData);
            if (reducedDeliveryTotalPrice != feeData) {
                isReducedDeliveryPrice = true;
            }
            this.setData({
                'deliveryAndSelfTaking.reducedDeliveryTotalPric': reducedDeliveryTotalPrice,
                'deliveryAndSelfTaking.isReducedDeliveryPrice': isReducedDeliveryPrice,
                'deliveryAndSelfTaking.feeData': feeData
            });
        },

        goToRecharge(e) {
            this.setData({
                isVipDialogShow: false
            });
            uni.navigateTo({
                url: '/pages/mine/member/recharge/recharge'
            });
        },

        customerServiceWechat() {
            console.log('占位：函数 customerServiceWechat 未声明');
        },

        goToPay() {
            console.log('占位：函数 goToPay 未声明');
        },

        trueFun() {
            console.log('占位：函数 true 未声明');
        }
    }
};
</script>
<style>
@import './pay.css';
</style>
