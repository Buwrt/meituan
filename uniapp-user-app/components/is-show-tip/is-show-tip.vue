<template>
    <view class="internet-error" v-if="isShow" :style="'padding-top: ' + top + '%;padding-bottom:' + bottom + '%;margin-bottom:' + bottom + '%;'">
        <image :src="src" mode="widthFix" class="icon-internet-error" v-if="src"></image>
        <text :class="'iconfont ' + (iconfont ? iconfont : 'iconwushuju')" v-else></text>
        <text v-if="text" class="text-tip-class">{{ tipText }}</text>
        <button size="mini" @tap="bindTap" v-if="button" style="font-size: 24rpx" :class="classText">{{ buttonText }}</button>
    </view>
</template>

<script>
// component.js
export default {
    data() {
        return {
            from: 'component'
        };
    },
    props: {
        isShow: {
            type: Boolean,
            default: false
        },
        src: {
            type: String,
            default: ''
        },
        iconfont: {
            type: String,
            default: ''
        },
        buttonText: {
            type: String,
            default: ''
        },
        top: {
            type: Number,
            default: 0
        },
        bottom: {
            type: Number,
            default: 0
        },
        button: {
            type: Boolean,
            default: false
        },
        text: {
            type: Boolean,
            default: false
        },
        tipText: {
            type: String,
            default: ''
        },
        classText: {
            type: String,
            default: ''
        }
    },
    mounted() {
        // 处理小程序 ready 生命周期
        this.$nextTick(() => this.ready());
    },
    methods: {
        ready() {},

        bindTap: function () {
            // 使用 triggerEvent 方法触发自定义组件事件，指定事件名、detail对象和事件选项
            this.$emit('bindTap');
        }
    },
    created: function () {}
};
</script>
<style>
@import './is-show-tip.css';
</style>
