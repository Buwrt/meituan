<template>
    <view>
        <block v-if="link">
            <view
                @tap="navigateTo"
                :class="'weui-cell weui-cell_access ' + extClass + ' ' + outerClass + (inForm ? ' weui-cell-inform' : '') + (inline ? '' : ' .weui-cell_label-block')"
                :hover-class="hover ? 'weui-cell_active' : ''"
            >
                <view :class="'weui-cell__hd ' + iconClass">
                    <block v-if="icon">
                        <image :src="icon" class="weui-cell__icon" mode="aspectFit"></image>
                    </block>
                    <block v-else>
                        <slot name="icon"></slot>
                    </block>
                    <block v-if="inForm">
                        <block v-if="title">
                            <view class="weui-label">{{ title }}</view>
                        </block>
                        <block v-else>
                            <slot name="title"></slot>
                        </block>
                    </block>
                    <block v-else>
                        <block v-if="title">{{ title }}</block>
                        <block v-else>
                            <slot name="title"></slot>
                        </block>
                    </block>
                </view>
                <view class="weui-cell__bd">
                    <block v-if="value">{{ value }}</block>
                    <block v-else>
                        <slot></slot>
                    </block>
                </view>
                <view :class="'weui-cell__ft weui-cell__ft_in-access ' + footerClass">
                    <block v-if="footer">{{ footer }}</block>
                    <block v-else>
                        <slot name="footer"></slot>
                    </block>
                </view>
            </view>
        </block>
        <block v-else>
            <view
                @tap="navigateTo"
                :class="'weui-cell ' + (showError && error ? 'weui-cell_warn' : '') + ' ' + (inForm ? 'weui-cell-inform' : '') + ' ' + extClass + ' ' + outerClass"
                :hover-class="hover ? 'weui-cell_active' : ''"
            >
                <view :class="'weui-cell__hd ' + iconClass">
                    <block v-if="icon">
                        <image :src="icon" class="weui-cell__icon" mode="aspectFit"></image>
                    </block>
                    <block v-else>
                        <slot name="icon"></slot>
                    </block>
                    <block v-if="inForm">
                        <block v-if="title">
                            <view class="weui-label">{{ title }}</view>
                        </block>
                        <block v-else>
                            <slot name="title"></slot>
                        </block>
                    </block>
                    <block v-else>
                        <block v-if="title">{{ title }}</block>
                        <block v-else>
                            <slot name="title"></slot>
                        </block>
                    </block>
                </view>
                <view class="weui-cell__bd">
                    <block v-if="value">{{ value }}</block>
                    <block v-else>
                        <slot></slot>
                    </block>
                </view>
                <view :class="'weui-cell__ft ' + footerClass">
                    <block v-if="footer">{{ footer }}</block>
                    <block v-else>
                        <slot name="footer"></slot>
                    </block>
                    <icon v-if="showError && error" type="warn" size="23" color="#E64340"></icon>
                </view>
            </view>
        </block>
    </view>
</template>

<script>
import mpCells from '../cells/cells';

export default {
    components: {
        mpCells
    },
    unicomGroup: ['cells'],
    data() {
        return {
            inForm: false,
            error: '',
            outerClass: ''
        };
    },
    options: {
        addGlobalClass: true,
        multipleSlots: true
    },
    props: {
        hover: {
            type: Boolean,
            default: false
        },
        link: {
            type: Boolean,
            default: false
        },
        extClass: {
            type: String,
            default: ''
        },
        iconClass: {
            type: String,
            default: ''
        },
        icon: {
            type: String,
            default: ''
        },
        title: {
            type: String,
            default: ''
        },
        value: {
            type: String,
            default: ''
        },
        showError: {
            type: Boolean,
            default: false
        },
        prop: {
            type: String,
            default: ''
        },
        url: {
            type: String,
            default: ''
        },
        footerClass: {
            type: String,
            default: ''
        },
        footer: {
            type: String,
            default: ''
        },
        inline: {
            type: Boolean,
            default: true
        }
    },
    relations: {
        '../form/form': {
            type: 'ancestor'
        },
        '../cells/cells': {
            type: 'ancestor'
        }
    },
    methods: {
        setError: function setError(error) {
            this.setData({
                error: error || false
            });
        },
        setInForm: function setInForm() {
            this.setData({
                inForm: true
            });
        },
        setOuterClass: function setOuterClass(className) {
            this.setData({
                outerClass: className
            });
        },
        navigateTo: function navigateTo() {
            var that = this;
            var data = this;
            if (data.url && data.link) {
                uni.navigateTo({
                    url: data.url,
                    success: function success(res) {
                        that.$emit(
                            'navigatesuccess',
                            {
                                detail: res
                            },
                            {}
                        );
                    },
                    fail: function fail(_fail) {
                        that.$emit(
                            'navigateerror',
                            {
                                detail: _fail
                            },
                            {}
                        );
                    }
                });
            }
        }
    },
    created: function () {}
};

/***/
</script>
<style>
@import './cell.css';
</style>
