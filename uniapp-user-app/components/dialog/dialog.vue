<template>
    <view>
        <view @tap="close" :class="'weui-mask ' + (!showClone ? 'weui-mask_hidden' : '')" v-if="mask" @touchmove.stop.prevent="trueFun"></view>
        <view v-if="showClone" @tap="close" :class="'weui-dialog__wrp ' + extClass">
            <view :class="'weui-dialog ' + extClass" @tap.stop.prevent="stopEvent">
                <view class="weui-dialog__hd">
                    <view class="weui-dialog__title">
                        {{ title }}
                        <slot name="title"></slot>
                    </view>
                </view>
                <view class="weui-dialog__bd">
                    <slot></slot>
                </view>
                <view class="weui-dialog__ft" v-if="buttonsClone && buttonsClone.length">
                    <block v-if="buttonsClone && buttonsClone.length">
                        <view
                            :class="'weui-dialog__btn ' + item.className + ' ' + item.extClass"
                            :data-index="index"
                            @tap="buttonTap"
                            v-for="(item, index) in buttonsClone"
                            :key="index"
                        >
                            {{ item.text }}
                        </view>
                        <!-- <view class="weui-dialog__btn" bindtap="confirm">确认</view> -->
                    </block>
                    <slot name="footer" v-else></slot>
                </view>
            </view>
            <view class="close-view-class" v-if="buttonsClone && buttonsClone.length <= 0">
                <text class="iconfont icon55" @tap="close"></text>
            </view>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            innerShow: false,
            maskClosable: false,
            buttonsClone: '',
            showClone: false
        };
    },
    options: {
        multipleSlots: true,
        addGlobalClass: true
    },
    props: {
        title: {
            type: String,
            default: ''
        },
        extClass: {
            type: String,
            default: ''
        },
        maskClosable: {
            type: Boolean,
            default: true
        },
        mask: {
            type: Boolean,
            default: true
        },
        show: {
            type: Boolean,
            default: false
        },
        buttons: {
            type: Array,
            default: () => []
        }
    },
    mounted() {
        // 处理小程序 ready 生命周期
        this.$nextTick(() => this.ready());
    },
    methods: {
        ready: function ready() {
            var buttons = this.buttons;
            var len = buttons.length;
            buttons.forEach(function (btn, index) {
                if (len === 1) {
                    btn.className = 'weui-dialog__btn_primary';
                } else if (index === 0) {
                    btn.className = 'weui-dialog__btn_default';
                } else {
                    btn.className = 'weui-dialog__btn_primary';
                }
            });
            this.setData({
                buttonsClone: buttons
            });
        },

        buttonTap: function buttonTap(e) {
            var index = e.currentTarget.dataset.index;
            this.$emit(
                'buttontap',
                {
                    detail: {
                        index: index,
                        item: this.buttons[index]
                    }
                },
                {}
            );
        },

        close: function close() {
            var data = this;
            //console.log(data)
            if (!data.maskClosable) {
                return;
            }
            this.setData({
                showClone: false
            });
            this.$emit(
                'close',
                {
                    detail: {}
                },
                {}
            );
        },

        stopEvent: function stopEvent() {},

        closeButton: function close() {
            this.setData({
                showClone: false
            });
            this.$emit(
                'close',
                {
                    detail: {}
                },
                {}
            );
        },

        trueFun() {
            console.log('占位：函数 true 未声明');
        }
    },
    created: function () {},
    watch: {
        buttons: {
            handler: function (newVal, oldVal) {
                this.buttonsClone = newVal;
            },

            immediate: true,
            deep: true
        },

        show: {
            handler: function (newVal, oldVal) {
                this.showClone = newVal;
            },

            immediate: true
        }
    }
};

/***/
</script>
<style>
@import './dialog.css';
</style>
