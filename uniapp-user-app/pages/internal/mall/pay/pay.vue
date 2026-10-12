<template>
    <view>
        <view class="top-detail" v-if="tipReward">
            <view class="current-tips" @tap.stop.prevent="customerServiceWechat">
                {{ tipReward }}
            </view>
        </view>
        <view class="ask-for-delivery-detail margin-border-radius">
            <view class="ask-for-delivery-title">收货地址</view>
            <view class="ask-for-delivery" @tap="openAddressDialog">
                <view class="ask-for-delivery-house-view" v-if="deliveryAndSelfTaking.deliveryAddress" data-radioIndex="0">
                    <view class="ask-for-delivery-view">
                        <view class="ask-for-delivery-house">
                            <text class="iconfont icondizhi"></text>
                            <view class="right-class out_of_range two_row">
                                {{ deliveryAndSelfTaking.deliveryAddress.province }}{{ deliveryAndSelfTaking.deliveryAddress.city }} {{ deliveryAndSelfTaking.deliveryAddress.area
                                }}{{ deliveryAndSelfTaking.deliveryAddress.street }}
                            </view>
                        </view>
                        <view class="ask-for-delivery-address-detail">
                            <text class="iconfont icon002dianhua"></text>
                            <view class="right-class">
                                <text>{{ deliveryAndSelfTaking.deliveryAddress.phone }}</text>
                                <text :decode="true">
                                    &nbsp;&nbsp;&nbsp;{{ deliveryAndSelfTaking.deliveryAddress.realname }}{{ deliveryAndSelfTaking.deliveryAddress.sex == 0 ? '先生' : '女士' }}
                                </text>
                            </view>
                        </view>
                    </view>
                    <text class="iconfont iconhtbArrowright02"></text>
                </view>
                <view class="please-address" v-else data-radioIndex="0">
                    <text>请选择地址</text>
                    <text class="iconfont iconhtbArrowright02"></text>
                </view>
            </view>
        </view>
        <view class="shopping-commodity-details margin-border-radius">
            <view class="commodity-name-price-detail" v-for="(item, index) in data.orderDetailList" :key="index">
                <view class="commodity-name-types">
                    <view class="commodity-name">{{ item.goodsName }}</view>
                    <view class="commodity-types">{{ item.restructure }}</view>
                </view>

                <view class="commodity-totalnum-price">
                    <view class="commodity-totalnum">x{{ item.number }}</view>
                    <view class="commodity-price">￥{{ item.price }}</view>
                </view>
            </view>
            <view class="commodity-name-price-detail">
                <view class="commodity-name-types">
                    <view class="commodity-name">运费</view>
                </view>
                <view class="commodity-totalnum-price">
                    <view></view>
                    <view class="commodity-types commodity-delivery-tip theme-color">免运费</view>
                </view>
            </view>
            <view class="commodity-name-price-detail" v-if="radioIndex == 1">
                <view class="commodity-name-types">
                    <view class="commodity-name">配送费</view>
                    <!-- <view class="commodity-types commodity-delivery-tip theme-color">面价满55元免配送费</view> -->
                </view>
                <view class="commodity-totalnum-price">
                    <view></view>
                    <view class="commodity-price">￥{{ feeData }}</view>
                </view>
            </view>
            <view class="commodity-name-price-detail" v-if="data.fullPriceReductionIsHidden">
                <view class="commodity-name-types">
                    <view class="commodity-name">满减</view>
                    <!-- <view class="commodity-types commodity-delivery-tip theme-color">面价满55元免配送费</view> -->
                </view>
                <view class="commodity-totalnum-price">
                    <view></view>
                    <view class="commodity-price full-reduction-view theme-color-border">{{ data.fullReductionRuleName }}</view>
                </view>
            </view>
            <view class="view-line"></view>
            <view class="total-money-view">
                <text class="money-icon commodity-price">总计：</text>
                <view :class="data.fullPriceReductionIsHidden || data.couponsIsHidden ? 'fullPriceReductionClass' : ''">
                    <text :class="'commodity-price ' + (data.fullPriceReductionIsHidden || data.couponsIsHidden ? 'strike_through' : '')">￥{{ data.actualPrice }}</text>
                </view>
                <text class="commodity-price" v-if="data.fullPriceReductionIsHidden || data.couponsIsHidden">￥{{ data.fullPriceReduction }}</text>
            </view>
        </view>
        <view class="pay-mode-view margin-border-radius">
            <view class="pay-mode-title">优惠券</view>
            <view class="choose-pay-mode" @tap="parseEventDynamicCode($event, afterDiscount ? 'onCoupon' : '')">
                <view class="theme-color after-discount flex_end" v-if="afterDiscount">
                    <view class="flex_end">
                        <text class="coupons-name-text" :decode="true">
                            {{ afterDiscount.couponsName ? afterDiscount.couponsName : availableCouponSize > 0 ? '未使用优惠券&nbsp;&nbsp;可用' + availableCouponSize + '张' : '' }}
                        </text>
                        <text :decode="true">{{ afterDiscount.couponsName ? '&nbsp;&nbsp;优惠' + afterDiscount.price + '元' : '' }}</text>
                    </view>
                </view>
                <view class="theme-color after-discount" v-else>
                    <text class="out_of_range one_row">无可用优惠券</text>
                </view>
                <text class="iconfont iconhtbArrowright02"></text>
            </view>
        </view>
        <!-- <view class="pay-mode-view margin-border-radius">
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
                <radio-group class="radio-group-address" @change="radioChangePayment" :data-firstIndex="key">
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
        <view class="ask-for-remarks-view margin-border-radius">
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
                <view class="total-money" v-if="!data.fullPriceReductionIsHidden && !data.couponsIsHidden">￥{{ data.actualPrice }}</view>
                <view class="total-money" v-else>￥{{ data.fullPriceReduction }}</view>
            </view>
            <view class="go-pay theme-bg" hover-class="hover-class-public" @tap="weChatPayTap">去支付</view>
        </view>
        <mp-actionSheet @actiontap="goToPay" :show="dialogShow" :actions="paymentModes" :title="title"></mp-actionSheet>
        <mp-halfScreenDialog :show="addressDialog" extClass="extClassSpecifications">
            <view slot="title">选择地址</view>
            <view slot="desc">
                <scroll-view scroll-y style="height: 66vh">
                    <radio-group class="radio-group-address" @change="radioChangeAddress" :data-firstIndex="key">
                        <label class="radio-label-address" v-for="(item, index) in addressList" :key="index">
                            <radio :value="index" :checked="item.id == deliveryAndSelfTaking.deliveryAddress.id" class="radio" />

                            <view class="choose-address-dialog">
                                <view class="address-detail">{{ item.province }}{{ item.city }}{{ item.area }}{{ item.street }}</view>
                                <view>
                                    <text class="address-name-phone">{{ item.realname }}</text>
                                    <text class="address-name-phone">{{ item.phone }}</text>
                                </view>
                            </view>

                            <text class="iconfont iconbianji-copy" @tap.stop.prevent="editAddress" :data-data="item" :data-key="index"></text>
                        </label>
                    </radio-group>
                </scroll-view>
                <view slot="footer" class="footer-btns">
                    <view class="good-choice-btn theme-color-border" @tap="insertAddress">新增</view>
                    <view class="good-choice-btn theme-bg" @tap="confirmChooseAddress">确认选择</view>
                </view>
            </view>
        </mp-halfScreenDialog>
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
import mpActionSheet from '../../../../components/actionsheet/actionsheet';
import mpHalfScreenDialog from '../../../../components/half-screen-dialog/half-screen-dialog';
import mpDialog from '../../../../components/dialog/dialog';
import https from '../../../../utils/http';
import authService from '../../../../utils/auth';
import toastService from '../../../../utils/toast.service';
import systemStatus from '../../../../utils/system-status';
import dateHelper from '../../../../utils/date-helper';
import utilHelper from '../../../../utils/util';
import base64 from '../../../../utils/base64';
const app = getApp();
export default {
    components: {
        mpActionSheet,
        mpHalfScreenDialog,
        mpDialog
    },
    data() {
        return {
            orderToken: '',
            time: '10:00',
            isChoose: false,

            //是否选择派送方式
            isFirstShop: false,

            //是否选择的是门店
            isFirstAddress: false,

            inputLength: 0,
            deliveryData: {},
            title: '请选择支付方式',

            paymentModes: [
                {
                    checked: false,
                    value: 1,
                    text: '微信支付',
                    show: true,
                    icon: 'iconwechat_pay'
                },
                {
                    checked: true,
                    value: 2,
                    text: '平台积分',
                    show: true,
                    icon: 'iconjifen'
                }
            ],

            paymentModeIndex: 1,
            dialogShow: false,
            addressDialog: false,
            remarks: '',
            isVipDialogShow: false,

            deliveryAndSelfTaking: {
                deliveryAddress: {
                    province: '',
                    city: '',
                    area: '',
                    street: '',
                    phone: '',
                    realname: '',
                    sex: 0,
                    id: ''
                }
            },

            payType: '',
            initData: '',
            timestamp: '',
            addressList: '',
            addrIndex: '',
            refreshKey: '',
            userInfo: '',

            afterDiscount: {
                couponsName: false,
                price: ''
            },

            availableCouponSize: '',

            data: {
                fullPriceReduction: '',
                couponsIsHidden: false,
                orderDetailList: [],
                fullPriceReductionIsHidden: '',
                fullReductionRuleName: '',
                actualPrice: ''
            },

            tipReward: '',
            payFocus: false,
            pwdVal: '',
            showPayPwdInput: false,
            isPayJson: '',
            balanceId: '',
            balanceOrderNo: '',
            balanceActualPrice: '',
            balanceOpenId: '',
            radioIndex: 0,
            feeData: '',
            maskClosable: '',
            i: '',
            holdKeyboard: '',
            adjustPosition: '',
            buttons: '',
            extClass: ''
        };
    },
    /**
     * 生命周期函数--监听页面加载
     */
    onLoad: function (options) {
        console.log(app.globalData.deliveryAndSelfTaking);
        this.setData({
            deliveryAndSelfTaking: app.globalData.deliveryAndSelfTaking
        });
        var prePageData = JSON.parse(options.orderDetail);
        prePageData.actualPrice = prePageData.actualPrice + 0;
        let time = dateHelper.formatTime('hms');
        //this.getFullReductionRule(data);
        this.getCouponsMemberRelation(prePageData);
        this.setData({
            payType: options.payType,
            time: time,
            initData: prePageData
        });

        // this.getShopAddressList();
        // setTimeout(out=>{
        //   this.getCouponsMemberRelation();
        // },1000)
    },
    /**
     * 生命周期函数--监听页面初次渲染完成
     */
    onReady: function () {},
    /**
     * 生命周期函数--监听页面显示
     */
    onShow: function () {
        this.setData(app.globalData.deliveryAndSelfTaking);
        this.getLoginMemberInfo();
        this.createOrderToken();
        setTimeout((time) => {
            this.selectCommissionReward();
            clearTimeout(time);
        }, 1500);
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

        openAddressDialog(e) {
            this.getDeliveryAddressList();
            this.setData({
                addressDialog: true
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
                        this.setData({
                            addressList: result.data.records
                        });
                    }
                });
        },

        radioChangeAddress(e) {
            console.log(e);
            this.setData({
                addrIndex: e.detail.value
            });
        },

        choosePayModeTap() {
            let paymentModes = this.paymentModes;
            let actualPrice =
                !this.data.fullPriceReductionIsHidden && !this.data.couponsIsHidden && !this.deliveryAndSelfTaking.isThereADiscount
                    ? this.data.actualPrice
                    : this.data.fullPriceReduction;
            console.log('积分余额=', this.userInfo.points);
            console.log('平台余额=', this.userInfo.balance);
            console.log('需支付金额=', actualPrice);
            paymentModes[1].desc = '';
            paymentModes[1].isBindTap = true;
            if (actualPrice > this.userInfo.points) {
                console.log('积分不足');
                paymentModes[1].desc = '积分不足';
                paymentModes[1].isBindTap = false;
            }
            if (!this.userInfo.paymentPassword) {
                paymentModes[1].desc = '未设置支付密码,去设置';
                paymentModes[1].isBindTap = false;
            }
            // this.setData({
            //   dialogShow: true,
            //   maskClosable: false,
            //   title: "请选择支付方式",
            //   paymentModes: paymentModes
            // })
            toastService.hideLoading();
            console.log(paymentModes);
            this.setData({
                paymentModes: paymentModes
            });
        },

        close() {
            this.setData({
                isVipDialogShow: false
            });
            toastService.showLoading();
            var _this = this;
            https.request('/api-mall/rest/member/pointsMall/order/insert', _this.isPayJson).then((result) => {
                toastService.hideLoading();
                if (result.success) {
                    // toastService.showLoading("正在支付...", true);
                    //console.log(result.data)
                    _this.toPay4Applet(result.data.id, result.data.orderNo, result.data.actualPrice);
                }
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

        closeDialog: function () {
            this.setData({
                dialogShow: false
            });
        },

        radioChangePayment(e) {
            console.log(e);
            var that = this;
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

        confirmChooseAddress(e) {
            console.log(e);
            app.globalData.deliveryAndSelfTaking.deliveryAddress = this.addressList[this.addrIndex];
            this.setData({
                addressDialog: false,
                'deliveryAndSelfTaking.deliveryAddress': this.addressList[this.addrIndex]
            });
        },

        insertAddress() {
            uni.navigateTo({
                url: '/pages/address/insert/insert'
            });
        },

        editAddress(e) {
            uni.navigateTo({
                url: '/pages/address/edit/edit?detail=' + JSON.stringify(e.currentTarget.dataset.data)
            });
            this.setData({
                refreshKey: e.currentTarget.dataset.key
            });
        },

        remarksInput(e) {
            this.setData({
                inputLength: e.detail.value.length,
                remarks: e.detail.value
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

        createOrderToken: function (e) {
            toastService.showLoading();
            https.request('/api-mall/rest/member/pointsMall/order/createOrderToken', {}).then((result) => {
                if (result.success) {
                    this.orderToken = result.data;
                }
            });
        },

        //this.getFullReductionRule(data);
        getCouponsMemberRelation(prePageData) {
            console.log(prePageData);
            toastService.showLoading();
            https
                .request('/api-mall/rest/member/pointsMall/couponsMemberRelation/list', {
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
                        var availableCouponSize = 0;
                        result.data.records.forEach((res) => {
                            //判断优惠券是否过期和是否已经使用
                            //if (!res.couponsMemberRelationMap.isExpired && !res.couponsMemberRelationMap.isUsed) {
                            //console.log(this.data)
                            //判断是否已经满减，如果已经满减就取满减之后的字段值，反之取总额
                            //if (this.data.data.actualPrice && !this.data.data.fullPriceReduction) {

                            //判断当前优惠券是折扣还是满减券,1等于折扣,2等于满减
                            if (res.couponsMemberRelationMap.preferentialType == 1) {
                                availableCouponSize++;
                                //遍历当前订单的商品
                                prePageData.orderDetailList.forEach((order) => {
                                    console.log(order);
                                    console.log(order.price);
                                    //遍历当前优惠券绑定的优惠商品
                                    res.goodsList.forEach((goods) => {
                                        console.log(order.price);
                                        //console.log(goods)
                                        //console.log(order)
                                        //判断当前订单的商品是否等于优惠券绑定的优惠商品
                                        //等于则进行优惠
                                        if (order.goodsId == goods.id) {
                                            // console.log(order.price)
                                            // console.log(order.number)
                                            // console.log(res.couponsMemberRelationMap.discountAmount != 0 ? utilHelper.toFixed((order.price / order.number) - ((order.price / order.number) * res.couponsMemberRelationMap.discountAmount), 2) : (order.price / order.number))
                                            afterDiscounts.push({
                                                id: res.couponsMemberRelationMap.id,
                                                price:
                                                    res.couponsMemberRelationMap.discountAmount != 0
                                                        ? utilHelper.toFixed(order.price - order.price * res.couponsMemberRelationMap.discountAmount, 2)
                                                        : order.price,
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
                                });
                                return;
                            }
                            //判断如果是优惠券满减的话这就进行优惠券的总价满减
                            //console.log(res)
                            if (res.couponsMemberRelationMap.preferentialType == 2) {
                                if (prePageData.fullPriceReduction) {
                                    if (prePageData.fullPriceReduction >= res.couponsMemberRelationMap.limitedPrice) {
                                        //console.log(res.couponsMemberRelationMap.limitedPrice)
                                        afterDiscounts.push({
                                            id: res.couponsMemberRelationMap.id,
                                            couponsId: res.couponsMemberRelationMap.couponsId,
                                            price: utilHelper.toFixed(prePageData.fullPriceReduction, 2) - res.couponsMemberRelationMap.reducedPrice,
                                            couponsName: res.couponsMemberRelationMap.couponsName,
                                            preferentialType: systemStatus.preferentialTypeText(res.couponsMemberRelationMap.preferentialType),
                                            isExpired: res.couponsMemberRelationMap.isExpired,
                                            isUsed: res.couponsMemberRelationMap.isUsed,
                                            isValid: res.couponsMemberRelationMap.isValid
                                        });
                                    }
                                    return;
                                }
                                if (prePageData.actualPrice >= res.couponsMemberRelationMap.limitedPrice) {
                                    afterDiscounts.push({
                                        id: res.couponsMemberRelationMap.id,
                                        couponsId: res.couponsMemberRelationMap.couponsId,
                                        price: prePageData.actualPrice - res.couponsMemberRelationMap.reducedPrice,
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
                        prePageData.couponsIsHidden = false;
                        prePageData.actualFullPriceReduction = prePageData.actualPrice;
                        console.log(prePageData.actualPrice);
                        if (afterDiscounts.length > 0) {
                            var afterDiscountPrice = 0;
                            var price = 0;
                            var afterDiscountList; //遍历所有可以使用优惠券的商品，并计算出最大的优惠券
                            afterDiscounts.forEach((afterDiscount) => {
                                price = Number(afterDiscount.price);
                                if (Number(price) >= afterDiscountPrice) {
                                    afterDiscountPrice = price;
                                    afterDiscountList = afterDiscount;
                                }
                            });
                            //console.log(this.data.data.fullPriceReduction)
                            //console.log(afterDiscountPrice)
                            //let fullPriceReduction = prePageData.fullPriceReduction ? (prePageData.fullPriceReduction - afterDiscountPrice) : (prePageData.actualPrice - afterDiscountPrice)
                            //console.log(prePageData.packingCharges)
                            //获取总金额（加配送费）减去优惠券的最后优惠价格
                            let fullPriceReduction = Number(prePageData.actualPrice) + Number(prePageData.packingCharges) - afterDiscountPrice;
                            prePageData.couponsIsHidden = true; //设置使用优惠券
                            prePageData.fullPriceReduction = utilHelper.toFixed(fullPriceReduction, 2);
                            //设置使用优惠券后价格属性，满减会使用到
                            prePageData.actualFullPriceReduction = prePageData.fullPriceReduction;
                            // prePageData.actualPrice=fullPriceReduction;
                            console.log(prePageData.actualFullPriceReduction);
                            this.setData({
                                afterDiscount: afterDiscountList,
                                availableCouponSize: availableCouponSize,
                                'data.fullPriceReduction': utilHelper.toFixed(fullPriceReduction <= 0 ? 0 : fullPriceReduction, 2),
                                'data.couponsIsHidden': true
                                // "data.afterDiscount":afterDiscountPrices
                            });
                        }
                        //调用满减优惠券方法（先试用优惠券再满减）
                        this.getFullReductionRule(prePageData);
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
            afterDiscount.limitedPrice = this.data.limitedPrice;
            afterDiscount.fullReductionRuleName = this.data.fullReductionRuleName;
            afterDiscount.fullReductionRuleId = this.data.fullReductionRuleId;
            afterDiscount.fullPriceReductionIsHidden = this.data.fullPriceReductionIsHidden;
            // afterDiscount.packingCharges = this.data.data.packingCharges;
            afterDiscount.type = 2;
            //console.log(afterDiscount)
            uni.navigateTo({
                url: '../../mine/coupons/coupons?prevData=' + JSON.stringify(afterDiscount)
            });
        },

        selectCommissionReward() {
            https
                .request('/api-mall/rest/member/pointsMall/order/selectCommissionReward', {
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

        /**
         * 获取焦点
         */
        getFocus: function () {
            this.setData({
                payFocus: true
            });
        },

        inputPwd: function (e) {
            this.setData({
                pwdVal: e.detail.value
            });
            if (e.detail.value.length >= 6) {
                toastService.showLoading('正在加载...');
                this.pointsPay();
                // if(this.data.paymentModes[this.data.paymentModeIndex].value==1){
                //   this.balancePay();
                // }
                // if(this.data.paymentModes[this.data.paymentModeIndex].value==2){
                //   this.pointsPay();
                // }
            }
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
                    type: 4,
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
                                url: '../../mall/order/detail/detail?id=' + this.balanceId
                            });
                            clearTimeout(timeout);
                        }, 1000);
                    }
                });
        },

        pointsPay() {
            var password = base64.encode(this.pwdVal);
            this.setData({
                pwdVal: '',
                payFocus: true
            });
            https
                .request('/api-mall/rest/member/pointsMall/platformPay/byPoints', {
                    openid: this.balanceOpenId,
                    type: 4,
                    paymentMode: 3,
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
                                url: '../../mall/order/detail/detail?id=' + this.balanceId
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
                    url: '../../mall/order/detail/detail?id=' + this.balanceId
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

        weChatPayTap() {
            //判断店铺是否打烊
            var data = {};
            if (this.deliveryAndSelfTaking.deliveryAddress == null) {
                toastService.showToast('请选择配送地址');
                return;
            }
            data.deliveryAddressId = this.deliveryAndSelfTaking.deliveryAddress.id;
            data.deliveryFee = this.deliveryAndSelfTaking.feeData;
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
            data.orderDetailList = orderDetailList;
            data.actualPrice = this.data.fullPriceReductionIsHidden || this.data.couponsIsHidden ? this.data.fullPriceReduction : this.data.actualPrice;
            // data.packingCharges = this.data.data.packingCharges;
            data.remark = this.remarks;
            data.shoppingWay = Number(this.radioIndex + 1);
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
            let addressMode =
                '配送地址：' +
                this.deliveryAndSelfTaking.deliveryAddress.province +
                this.deliveryAndSelfTaking.deliveryAddress.city +
                this.deliveryAndSelfTaking.deliveryAddress.area +
                this.deliveryAndSelfTaking.deliveryAddress.street;
            // toastService.showModal("积分商城订单", "订单确认后将无法更改!\r\n确认下单吗?", function comfirm() {

            //防重令牌
            data.orderToken = this.orderToken;
            if (this.userInfo.type == 1) {
                this.setData({
                    isVipDialogShow: true,
                    isPayJson: data
                });
            } else {
                toastService.showLoading();
                https.request('/api-mall/rest/member/pointsMall/order/insert', data).then((result) => {
                    toastService.hideLoading();
                    if (result.success) {
                        // toastService.showLoading("正在支付...", true);
                        //console.log(result.data)
                        that.toPay4Applet(result.data.id, result.data.orderNo, result.data.actualPrice);
                    }
                });
            }
            // }, null)
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
                if (this.paymentModes[this.paymentModeIndex].value == 2 || this.paymentModes[this.paymentModeIndex].value == 3) {
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
                    type: 4,
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
                                        url: '../../mall/order/detail/detail?id=' + id
                                    });
                                    clearTimeout(timeout);
                                }, 1000);
                            },
                            fail(res) {
                                toastService.showError('支付失败', true);
                                let timeout = setTimeout(() => {
                                    uni.redirectTo({
                                        url: '../../mall/order/detail/detail?id=' + id
                                    });
                                    clearTimeout(timeout);
                                }, 1000);
                            }
                        });
                    }
                });
        },

        // toPay4Applet(id, orderNo, actualPrice) {
        //   toastService.showLoading("正在加载...", true);
        //   authService.getOpenId().then(openId => {
        //     //console.log(openId)
        //     if (openId) {
        //       https.request('/api-order/rest/member/order/cancelOrder', {
        //         openid: openId,
        //         type: 1,
        //         out_trade_no: orderNo,
        //         total_fee: this.data.data.fullPriceReductionIsHidden ? this.data.data.fullPriceReduction : actualPrice
        //       }).then(result => {
        //         toastService.hideLoading();
        //         if (result.success) {
        //           //console.log(result)
        //           wx.requestPayment({
        //             appId: result.data.appid,
        //             timeStamp: result.data.timeStamp,
        //             nonceStr: result.data.nonceStr,
        //             package: result.data.package,
        //             signType: 'MD5',
        //             paySign: result.data.paySign,
        //             success(res) {
        //               toastService.showSuccess("支付成功", true);
        //               let timeout = setTimeout(() => {
        //                 wx.redirectTo({
        //                   url: '../order-details/order-details?id=' + id,
        //                 });
        //                 clearTimeout(timeout);
        //               }, 1000)
        //             },
        //             fail(res) {
        //               toastService.showError("支付失败", true);
        //               let timeout = setTimeout(() => {
        //                 wx.redirectTo({
        //                   url: '../order-details/order-details?id=' + id,
        //                 });
        //                 clearTimeout(timeout);
        //               }, 1000)
        //             }
        //           })
        //         }
        //       })
        //       return
        //     }
        //     toastService.showToast("登录用户错误，请重新登录");
        //   });
        // },
        //获取满减规则
        getFullReductionRule(data) {
            console.log(data);
            toastService.showLoading();
            https
                .request('/api-mall/rest/pointsMall/fullReductionRule/list', {
                    pageNo: -1,
                    pageSize: 1
                })
                .then((result) => {
                    toastService.hideLoading();
                    if (result.success) {
                        //this.getCouponsMemberRelation();
                        //获取配送费，配送费不作为满减条件
                        let feeData = 0;

                        // if (!this.data.isPreviousPage) {
                        //   data.packingCharges = 0;
                        //   for (var key in data.orderDetailList) {
                        //     //data.orderDetailList[key].price = data.orderDetailList[key].price * data.orderDetailList[key].number;
                        //     data.packingCharges = data.packingCharges + (data.orderDetailList[key].packingCharges * data.orderDetailList[key].number);
                        //   }
                        // }

                        data.actualPrice = data.actualPrice;
                        data.fullPriceReductionAfter = data.actualPrice;
                        data.discountPrice = 0;
                        data.fullPriceReduction = data.fullPriceReduction;
                        data.actualFullPriceReduction = data.actualFullPriceReduction;
                        data.fullPriceReductionIsHidden = false;
                        data.limitedPrice = 0;
                        console.log(data.actualFullPriceReduction);
                        for (let i = 0; i < result.data.records.length; i++) {
                            //console.log(result.data.records[i].limitedPrice)
                            //总价格减去配送费大于满减金额则进行满减操作（配送费不计算入内）
                            if (Number(data.actualFullPriceReduction - feeData) >= result.data.records[i].limitedPrice) {
                                //判断当前满减价格limitedPrice和上一个满减价格对比，如果大于就进行认证
                                if (result.data.records[i].limitedPrice > data.limitedPrice) {
                                    data.limitedPrice = result.data.records[i].limitedPrice;
                                    data.fullPriceReduction = data.actualFullPriceReduction
                                        ? utilHelper.toFixed(data.actualFullPriceReduction - result.data.records[i].reducedPrice, 2)
                                        : data.actualFullPriceReduction;
                                    data.fullReductionRuleName = result.data.records[i].name;
                                    data.fullReductionRuleId = result.data.records[i].id;
                                    data.fullPriceReductionIsHidden = true;
                                    data.fullPriceReductionAfter = utilHelper.toFixed(data.actualFullPriceReduction - result.data.records[i].reducedPrice, 2);
                                }
                            }
                        }
                        this.setData({
                            data
                        });
                    }
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
