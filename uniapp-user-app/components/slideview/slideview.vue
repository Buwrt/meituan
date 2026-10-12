<template>
    <!-- slide-view/slide-view.wxml -->

    <view :class="'weui-slideview weui-movable-view ' + (icon ? 'weui-slideview_icon' : '') + ' ' + extClass" style="width: 100%; height: 100%; overflow: hidden">
        <view
            :change:disable="handler.disableChange"
            :disable="disable"
            :change:prop="handler.sizeReady"
            :prop="size"
            @touchstart="parseEventDynamicCode($event, handler.touchstart)"
            @touchmove="parseEventDynamicCode($event, handler.touchmove)"
            @touchend="parseEventDynamicCode($event, handler.touchend)"
            class="weui-slideview__left left"
            style="width: 100%"
        >
            <slot></slot>
        </view>
        <view class="weui-slideview__right right">
            <view class="weui-slideview__buttons" style="height: 100%; width: 100%" v-if="buttonsClone && buttonsClone.length">
                <view :class="'btn weui-slideview__btn__wrp ' + item.className + ' ' + item.extClass" v-for="(item, index) in buttonsClone" :key="index">
                    <view @tap="parseEventDynamicCode($event, handler.hideButton)" :data-data="item.data" :data-index="index" class="weui-slideview__btn">
                        <text v-if="!icon">{{ item.text }}</text>
                        <image class="weui-slideview__btn__icon" v-else :src="item.src" />
                    </view>
                </view>
            </view>
        </view>
    </view>
</template>
<script module="handler" lang="wxs" src="@/components/slideview/slideview.wxs"></script>
<script>
export default {
    data() {
        return {
            size: null,
            buttonsClone: []
        };
    },
    options: {
        addGlobalClass: true,
        multipleSlots: true
    },
    props: {
        extClass: {
            type: String,
            default: ''
        },
        buttons: {
            type: Array,
            default: () => []
        },
        disable: {
            type: Boolean,
            default: false
        },
        icon: {
            type: Boolean,
            default: false
        }
    },
    mounted() {
        // 处理小程序 ready 生命周期
        this.$nextTick(() => this.ready());
    },
    methods: {
        ready: function ready() {
            this.updateRight();
            this.addClassNameForButton();
        },

        updateRight: function updateRight() {
            var that = this;
            var query = uni.createSelectorQuery().in(this);
            query
                .select('.left')
                .boundingClientRect(function (res) {
                    //console.log('right res', res);
                    var btnQuery = uni.createSelectorQuery().in(that);
                    btnQuery
                        .selectAll('.btn')
                        .boundingClientRect(function (rects) {
                            //console.log('btn rects', rects);
                            that.setData({
                                size: {
                                    buttons: rects,
                                    button: res
                                }
                            });
                        })
                        .exec();
                })
                .exec();
        },

        addClassNameForButton: function addClassNameForButton() {
            var _data = this;
            var buttons = _data.buttons;
            var icon = _data.icon;
            buttons.forEach(function (btn) {
                if (icon) {
                    btn.className = '';
                } else if (btn.type === 'warn') {
                    btn.className = 'weui-slideview__btn-group_warn';
                } else {
                    btn.className = 'weui-slideview__btn-group_default';
                }
            });
            this.setData({
                buttonsClone: buttons
            });
        },

        buttonTapByWxs: function buttonTapByWxs(data) {
            this.$emit(
                'buttontap',
                {
                    detail: data
                },
                {}
            );
        },

        hide: function hide() {
            this.$emit(
                'hide',
                {
                    detail: {}
                },
                {}
            );
        },

        show: function show() {
            this.$emit(
                'show',
                {
                    detail: {}
                },
                {}
            );
        }
    },
    created: function () {},
    watch: {
        buttons: {
            handler: function observer(newVal) {
                this.buttonsClone = this.clone(this.buttons);
                this.addClassNameForButton();
            },

            immediate: true,
            deep: true
        }
    }
};

/***/
</script>
<style>
@import './slideview.css';
</style>
