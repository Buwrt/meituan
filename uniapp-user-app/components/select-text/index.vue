<template>
    <view class="weui-select-text" :style="computed.containerStyle({ showToolTip, showCopyBtn, activeBgColor })" @tap.stop.prevent="stopPropagation">
        <view class="weui-mask init" v-if="maskClone" @tap="close" data-type="tap" @touchmove.stop.prevent="returnFun"></view>
        <text :selectable="!showCopyBtn" :space="space" :decode="decode" @longpress.stop.prevent="handleLongPress">{{ value }}</text>
        <view v-if="showToolTip" :class="'weui-tooltip weui-tooltip__' + placement" :style="'z-index: ' + zIndex + ';'">
            <block v-for="(item, index) in selectTexts" :key="index">
                <text
                    :class="'weui-tooltip-btn-item ' + (selectTexts.length - 1 != index ? 'right' : '')"
                    @tap.stop.prevent="parseEventDynamicCode($event, item.bind ? item.bind : 'handleCurrencyTap')"
                    :data-type="item.type"
                >
                    {{ item.name }}
                </text>
            </block>
        </view>
    </view>
</template>
<script module="computed" lang="wxs" src="@/components/select-text/index.wxs"></script>
<script>
export default {
    data() {
        return {
            showToolTip: false,
            maskClone: false,
            maskClone: false
        };
    },

    options: {
        addGlobalClass: true
    },

    props: {
        space: {
            type: String,
            default: ''
        },
        decode: {
            type: Boolean,
            default: false
        },
        placement: {
            type: String,
            default: 'top'
        },
        showCopyBtn: {
            type: Boolean,
            default: false
        },
        value: {
            type: String,
            default: ''
        },
        zIndex: {
            type: Number,
            default: 9999
        },
        activeBgColor: {
            type: String,
            default: '#DEDEDE'
        },
        onDocumentTap: {
            type: Object,
            default: () => ({})
        },
        selectTexts: {
            type: Object,
            default: () => [
                {
                    name: '复制',
                    type: 'copy',
                    bind: 'handleCopy'
                }
            ]
        },
        mask: {
            type: Boolean,
            default: false
        }
    },

    watch: {
        onDocumentTap: function onDocumentTap() {
            this.setData({
                showToolTip: false,
                maskClone: false
            });
        },

        mask: {
            handler: function (newVal, oldVal) {
                this.maskClone = newVal;
            },

            immediate: true
        }
    },

    methods: {
        handleLongPress: function handleLongPress() {
            if (!this.showCopyBtn) {
                return;
            }
            this.setData({
                showToolTip: true,
                maskClone: true
            });
        },

        handleCopy: function handleCopy() {
            this.setData({
                showToolTip: false,
                maskClone: false
            });
            uni.setClipboardData({
                data: this.value
            });
            this.$emit('copy', {
                detail: {}
            });
        },

        handleCurrencyTap: function handleCurrencyTap(e) {
            this.setData({
                showToolTip: false,
                maskClone: false
            });
            this.$emit('currency', {
                detail: {
                    type: e.target.dataset.type
                }
            });
        },

        close: function close(e) {
            var mask = this.mask;
            if (mask) {
                this.setData({
                    showToolTip: false,
                    maskClone: false
                });
            }
        },

        stopPropagation: function stopPropagation(e) {
            this.$emit('tap', {
                detail: {}
            });
        },

        returnFun() {
            console.log('占位：函数 return 未声明');
        }
    },

    created: function () {}
};

/***/
</script>
<style>
@import './index.css';
</style>
