<template>
    <view>
        <view v-if="mask" :class="'weui-mask ' + (showClone ? '' : 'weui-mask_hidden') + ' ' + maskClass" @tap="closeActionSheet" @touchmove.stop.prevent="catchtouchmove"></view>
        <view :class="'weui-actionsheet ' + (showClone ? 'weui-actionsheet_toggle' : '') + ' ' + extClass">
            <!-- 标题 -->
            <block v-if="title">
                <view class="weui-actionsheet__title">
                    <view class="weui-actionsheet__title-text">{{ title }}</view>
                </view>
            </block>
            <slot name="title" v-else></slot>
            <view
                :class="!showCancel && index === actionsClone.length - 1 ? 'weui-actionsheet__action' : 'weui-actionsheet__menu'"
                v-for="(actionItem, index) in actionsClone"
                :key="index"
            >
                <block v-if="utils.isNotSlot(actionItem)">
                    <view
                        :class="'weui-actionsheet__cell ' + (item.type === 'warn' ? 'weui-actionsheet__cell_warn' : '')"
                        :data-groupindex="index"
                        :data-index="actionIndex"
                        :data-value="item.value"
                        v-if="item.show"
                        @tap="parseEventDynamicCode($event, !item.isBindTap ? 'buttonTap' : '')"
                        v-for="(item, actionIndex) in actionItem"
                        :key="item.text"
                    >
                        <text v-if="item.icon" :class="'iconfont ' + item.icon" :decode="true">&nbsp;</text>

                        {{ item.text }}

                        <text v-if="item.desc" class="actionItem__desc">{{ item.desc }}</text>
                    </view>
                </block>

                <slot :name="actionItem" v-else></slot>
            </view>
            <!-- 取消按钮 -->
            <view class="weui-actionsheet__action" v-if="showCancel">
                <view class="weui-actionsheet__cell" data-type="close" id="iosActionsheetCancel" @tap="closeActionSheet">{{ cancelText }}</view>
            </view>
        </view>
    </view>
</template>
<script module="utils" lang="wxs">
var join = function(a,b) {
    return a+b
};
var isNotSlot = function(v) {
    return typeof v !== 'string'
}
module.exports = {
    join: join,
    isNotSlot: isNotSlot
}
</script>
<script>
export default {
    data() {
        return {
            actionIndex: 0,
            actionItem: [],
            actionsClone: [],
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
        showCancel: {
            type: Boolean,
            default: true
        },
        cancelText: {
            type: String,
            default: '取消'
        },
        maskClass: {
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
        actions: {
            type: Array,
            default: () => []
        }
    },
    methods: {
        groupChangeFun: function _groupChange(e) {
            if (e.length > 0 && typeof e[0] !== 'string' && !(e[0] instanceof Array)) {
                this.setData({
                    actionsClone: [this.actions]
                });
            }
        },

        buttonTap: function buttonTap(e) {
            var _e$currentTarget$data = e.currentTarget.dataset;
            var value = _e$currentTarget$data.value;
            var groupindex = _e$currentTarget$data.groupindex;
            var index = _e$currentTarget$data.index;
            this.$emit('actiontap', {
                detail: {
                    value: value,
                    groupindex: groupindex,
                    index: index
                }
            });
        },

        closeActionSheet: function closeActionSheet(e) {
            var type = e.currentTarget.dataset.type;
            if (this.maskClosable || type) {
                this.setData({
                    showClone: false
                });
                this.$emit('close');
            }
        },

        catchtouchmove() {
            console.log('占位：函数 catchtouchmove 未声明');
        }
    },
    created: function () {},
    watch: {
        actions: {
            handler: function _groupChange(e) {
                this.actionsClone = this.clone(this.actions);
                if (e.length > 0 && typeof e[0] !== 'string' && !(e[0] instanceof Array)) {
                    this.setData({
                        actionsClone: [this.actions]
                    });
                }
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
@import './actionsheet.css';
</style>
