<template>
    <view :class="'weui-navigation-bar ' + extClass">
        <view :class="'weui-navigation-bar__placeholder ' + (ios ? 'ios' : 'android')" :style="'padding-top: ' + statusBarHeight + 'px;visibility: hidden;'"></view>
        <view
            :class="'weui-navigation-bar__inner ' + (ios ? 'ios' : 'android')"
            :style="
                'padding-top: ' + statusBarHeight + 'px; color: ' + color + ';background: ' + background + ';' + displayStyle + ';' + innerPaddingRight + ';' + innerWidth + ';'
            "
        >
            <view class="weui-navigation-bar__left" :style="leftWidth">
                <block v-if="back">
                    <view class="weui-navigation-bar__buttons">
                        <view @tap="backFun" class="weui-navigation-bar__button weui-navigation-bar__btn_goback"></view>
                    </view>
                </block>
                <block v-else>
                    <slot name="left"></slot>
                </block>
            </view>

            <view class="weui-navigation-bar__center">
                <view v-if="loading" class="weui-navigation-bar__loading">
                    <view class="weui-loading" :style="'width:' + size.width + 'rpx;height:' + size.height + 'rpx;'"></view>
                </view>
                <block v-if="title">
                    <text>{{ title }}</text>
                </block>
                <block v-else>
                    <slot name="center"></slot>
                </block>
            </view>

            <view class="weui-navigation-bar__right">
                <slot name="right"></slot>
            </view>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            displayStyle: '',
            ios: '',
            statusBarHeight: '',
            innerWidth: '',
            innerPaddingRight: '',
            leftWidth: '',

            size: {
                width: '',
                height: ''
            }
        };
    },
    options: {
        multipleSlots: true,
        // 在组件定义时的选项中启用多slot支持
        addGlobalClass: true
    },
    /**
     * 组件的属性列表
     */
    props: {
        extClass: {
            type: String,
            default: ''
        },
        title: {
            type: String,
            default: ''
        },
        background: {
            type: String,
            default: ''
        },
        color: {
            type: String,
            default: ''
        },
        back: {
            type: Boolean,
            default: true
        },
        loading: {
            type: Boolean,
            default: false
        },
        animated: {
            // 显示隐藏的时候opacity动画效果
            type: Boolean,
            default: true
        },
        show: {
            // 显示隐藏导航，隐藏的时候navigation-bar的高度占位还在
            type: Boolean,

            default: true
        },
        // back为true的时候，返回的页面深度
        delta: {
            type: Number,
            default: 1
        }
    },
    mounted() {
        // 处理小程序 attached 生命周期
        this.attached();
    },
    /**
     * 组件的方法列表
     */
    methods: {
        attached() {
            const isSupport = !!uni.getMenuButtonBoundingClientRect;
            const rect = uni.getMenuButtonBoundingClientRect ? uni.getMenuButtonBoundingClientRect() : null;
            uni.getSystemInfo({
                success: (res) => {
                    const ios = !!(res.system.toLowerCase().search('ios') + 1);
                    this.setData({
                        ios,
                        statusBarHeight: res.statusBarHeight,
                        innerWidth: isSupport ? `width:${rect.left}px` : '',
                        innerPaddingRight: isSupport ? `padding-right:${res.windowWidth - rect.left}px` : '',
                        leftWidth: isSupport ? `width:${res.windowWidth - rect.left}px` : ''
                    });
                }
            });
        },

        showChangeFun(show) {
            const animated = this.animated;
            let displayStyle = '';
            if (animated) {
                displayStyle = `opacity: ${show ? '1' : '0'};-webkit-transition:opacity 0.5s;transition:opacity 0.5s;`;
            } else {
                displayStyle = `display: ${show ? '' : 'none'}`;
            }
            this.setData({
                displayStyle
            });
        },

        backFun() {
            const data = this;
            if (data.delta) {
                uni.navigateBack({
                    delta: data.delta
                });
            }
            this.$emit(
                'back',
                {
                    detail: {
                        delta: data.delta
                    }
                },
                {}
            );
        }
    },
    created: function () {},
    watch: {
        show: {
            handler: function (show) {
                const animated = this.animated;
                let displayStyle = '';
                if (animated) {
                    displayStyle = `opacity: ${show ? '1' : '0'};-webkit-transition:opacity 0.5s;transition:opacity 0.5s;`;
                } else {
                    displayStyle = `display: ${show ? '' : 'none'}`;
                }
                this.setData({
                    displayStyle
                });
            },

            immediate: true
        }
    }
};
</script>
<style>
@import './navigation-bar.css';
</style>
