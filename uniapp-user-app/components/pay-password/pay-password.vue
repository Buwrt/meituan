<template>
    <!-- 密码输入框 -->
    <view v-if="showPayPwdInput">
        <view class="bg_layer"></view>
        <view class="password_dialog_main">
            <view class="input_title">
                <view class="input_back" @tap.stop.prevent="hidePayLayer"><text></text></view>
                <text>输入支付密码</text>
            </view>
            <view class="password_dialog_tip"><text>使用会员卡余额支付需要验证身份，验证通过后才可进行支付。</text></view>
            <view class="password_dialog_row" @tap.stop.prevent="getFocus">
                <view class="password_dialog_item_input" v-for="(item, i) in 6" :key="i">
                    <text v-if="pwdValClone.length > i"></text>
                </view>
            </view>
            <view class="password_dialog_forget_pwd" @tap.stop.prevent="hidePayLayer">忘记密码</view>
            <input class="password_dialog_input_control" password type="number" :focus="payFocusClone" @input="inputPwd" maxlength="6" />
        </view>
    </view>
</template>

<script>
// component.js
export default {
    data() {
        return {
            from: 'component',
            showPayPwdInput: false,
            i: '',
            payFocusClone: false,
            pwdValClone: ''
        };
    },
    props: {
        isShow: {
            type: Boolean,
            default: false
        },
        pwdVal: {
            type: Number,
            default: ''
        },
        payFocus: {
            type: Boolean,
            default: false
        }
    },
    mounted() {
        // 处理小程序 ready 生命周期
        this.$nextTick(() => this.ready());
    },
    methods: {
        ready() {},

        /**
         * 显示支付密码输入层
         */
        showInputLayer: function () {
            this.setData({
                showPayPwdInput: true,
                payFocusClone: true
            });
        },

        /**
         * 隐藏支付密码输入层
         */
        hidePayLayer: function () {
            var val = this.pwdVal;
            this.setData(
                {
                    showPayPwdInput: false,
                    payFocusClone: false,
                    pwdValClone: ''
                },
                function () {
                    uni.showToast({
                        title: val
                    });
                }
            );
        },

        /**
         * 获取焦点
         */
        getFocus: function () {
            this.setData({
                payFocusClone: true
            });
        },

        /**
         * 输入密码监听
         */
        inputPwd: function (e) {
            this.setData({
                pwdValClone: e.detail.value
            });
            if (e.detail.value.length >= 6) {
                this.hidePayLayer();
            }
        }
    },
    created: function () {},
    watch: {
        payFocus: {
            handler: function (newVal, oldVal) {
                this.payFocusClone = newVal;
            },

            immediate: true
        },

        pwdVal: {
            handler: function (newVal, oldVal) {
                this.pwdValClone = newVal;
            },

            immediate: true
        }
    }
};
</script>
<style>
@import './pay-password.css';
</style>
