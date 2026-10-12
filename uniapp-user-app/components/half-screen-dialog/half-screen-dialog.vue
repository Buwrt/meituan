<template>
    <view :class="showClone ? 'weui-show' : 'weui-hidden'" @touchmove.stop.prevent="catchtouchmove">
        <view class="weui-mask init" v-if="mask" @tap="parseEventDynamicCode($event, maskClosable ? 'close' : '')" data-type="close" @touchend.stop.prevent="bindtouchend"></view>
        <view :class="'weui-half-screen-dialog ' + extClass">
            <view class="weui-half-screen-dialog__hd">
                <view class="weui-half-screen-dialog__hd__side">
                    <!-- <view class="weui-icon-btn weui-icon-btn_more">更多</view> -->
                </view>
                <view class="weui-half-screen-dialog__hd__main">
                    <block v-if="title">
                        <text class="weui-half-screen-dialog__title">{{ title }}</text>
                        <text class="weui-half-screen-dialog__subtitle">{{ subTitle }}</text>
                    </block>
                    <block v-else>
                        <view class="weui-half-screen-dialog__title"><slot name="title"></slot></view>
                    </block>
                </view>
                <view v-if="closabled" class="weui-half-screen-dialog__hd__side" @tap="close" data-type="close">
                    <view class="weui-icon-btn weui-icon-btn_close">关闭</view>
                </view>
            </view>
            <view class="weui-half-screen-dialog__bd">
                <block v-if="title">
                    <view class="weui-half-screen-dialog__desc">{{ desc }}</view>
                    <view class="weui-half-screen-dialog__tips">{{ tips }}</view>
                </block>
                <slot name="desc" v-else></slot>
            </view>
            <view class="weui-half-screen-dialog__ft">
                <block v-if="buttons && buttons.length">
                    <button
                        :type="item.type"
                        :class="'weui-btn ' + item.className"
                        :data-index="index"
                        @tap="buttonTap"
                        v-for="(item, index) in buttons"
                        :key="item.text + item.index"
                    >
                        {{ item.text }}
                    </button>
                    <!-- <view wx:for="{{buttons}}" class="weui-dialog__btn {{}} {{item.extClass}}" ></view> -->
                </block>
                <slot name="footer" v-else></slot>
            </view>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            showClone: false
        };
    },
    options: {
        multipleSlots: true,
        addGlobalClass: true
    },
    props: {
        closabled: {
            type: Boolean,
            default: true
        },
        title: {
            type: String,
            default: ''
        },
        subTitle: {
            type: String,
            default: ''
        },
        extClass: {
            type: String,
            default: ''
        },
        desc: {
            type: String,
            default: ''
        },
        tips: {
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
    methods: {
        close: function close(e) {
            var type = e.currentTarget.dataset.type;
            console.log(type);
            if (this.maskClosable || type === 'close') {
                this.setData({
                    showClone: false
                });
                this.$emit('close');
            }
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

        bindMark: function bindMark(e) {
            this.$emit('bindmark');
        },

        bindtouchend: function bindtouchend(e) {
            this.$emit('bindtouchend');
        },

        catchtouchmove() {
            console.log('占位：函数 catchtouchmove 未声明');
        }
    },
    created: function () {},
    watch: {
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
@import './half-screen-dialog.css';
</style>
