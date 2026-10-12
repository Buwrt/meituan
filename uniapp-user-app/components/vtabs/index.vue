<template>
    <view class="weui-vtabs">
        <view :class="'weui-vtabs-bar__wrp ' + tabBarClass">
            <scroll-view
                :scroll-y="ifScrolling"
                class="weui-vtabs-bar__scrollview"
                :scroll-into-view="'weui-vtabs-item__' + currentView"
                :style="'height:' + contentHeight + 'px;'"
            >
                <view class="weui-vtabs-bar__content">
                    <block v-for="(item, index) in vtabs" :key="index">
                        <view
                            :id="'weui-vtabs-item__' + index"
                            class="weui-vtabs-bar__item"
                            :data-index="index"
                            @tap="handleTabClick"
                            :style="
                                'position: relative;background-color: ' +
                                (activeTabClone === index ? tabBarActiveBgColor : tabBarInactiveBgColor) +
                                ';color: ' +
                                (activeTabClone === index ? tabBarActiveTextColor : tabBarInactiveTextColor) +
                                ';'
                            "
                        >
                            <view :class="'weui-vtabs-bar__title ' + (activeTabClone === index ? activeClass : '')">
                                <text>{{ item.name }}</text>
                            </view>
                            <view
                                :style="
                                    'position: absolute;top:0;background-color: ' +
                                    (activeTabClone === index ? tabBarActiveTextColor : tabBarInactiveBgColor) +
                                    ';color: ' +
                                    (activeTabClone === index ? tabBarActiveTextColor : tabBarInactiveTextColor) +
                                    ';width: 4px;height: 100%;'
                                "
                            ></view>
                        </view>
                    </block>
                </view>
            </scroll-view>
        </view>
        <view class="weui-vtabs-content__wrp">
            <scroll-view
                :scroll-y="ifScrolling"
                class="weui-vtabs-content__scrollview"
                :scroll-top="contentScrollTop"
                :scroll-with-animation="animation"
                @scroll="handleContentScroll"
                :style="'height:' + contentHeight + 'px;'"
            >
                <view class="weui-vtabs-content">
                    <slot></slot>
                </view>
            </scroll-view>
        </view>
    </view>
</template>

<script>
export default {
    unicomGroup: ['vtabs-content'],
    data() {
        return {
            currentView: 0,
            contentScrollTop: 0,
            heightRecords_var: [],
            contentHeight_var: {},
            activeTabClone: 0
        };
    },
    options: {
        addGlobalClass: true,
        pureDataPattern: /^_/,
        multipleSlots: true
    },
    props: {
        vtabs: {
            type: Array,
            default: () => []
        },
        tabBarClass: {
            type: String,
            default: ''
        },
        activeClass: {
            type: String,
            default: ''
        },
        tabBarLineColor: {
            type: String,
            default: '#ff0000'
        },
        tabBarInactiveTextColor: {
            type: String,
            default: '#000000'
        },
        tabBarActiveTextColor: {
            type: String,
            default: '#ff0000'
        },
        tabBarInactiveBgColor: {
            type: String,
            default: '#eeeeee'
        },
        tabBarActiveBgColor: {
            type: String,
            default: '#ffffff'
        },
        activeTab: {
            type: Number,
            default: 0
        },
        contentHeight: {
            type: Number,
            default: 100
        },
        animation: {
            type: Boolean,
            default: true
        },
        ifScrolling: {
            type: Boolean,
            default: true
        }
    },
    watch: {
        activeTab: function activeTab(_activeTab) {
            this.scrollTabBar(_activeTab);
        },

        activeTab: {
            handler: function (newVal, oldVal) {
                this.activeTabClone = newVal;
            },

            immediate: true
        }
    },
    relations: {
        '../vtabs-content/index': {
            type: 'child',
            linked: function linked(target) {
                var that = this;
                target.calcHeight(function (rect) {
                    that._contentHeight[target.data.tabIndex] = rect.height;
                    if (that._calcHeightTimer) {
                        clearTimeout(that._calcHeightTimer);
                    }
                    that._calcHeightTimer = setTimeout(function () {
                        that.calcHeight();
                    }, 100);
                });
            },
            unlinked: function unlinked(target) {
                delete this._contentHeight[target.data.tabIndex];
            }
        }
    },
    mounted() {
        // 处理小程序 attached 生命周期
        this.attached();
    },
    methods: {
        attached: function attached() {},

        calcHeight: function calcHeight() {
            var length = this.vtabs.length;
            var _contentHeight = this._contentHeight;
            var _heightRecords = [];
            var temp = 0;
            for (var i = 0; i < length; i++) {
                _heightRecords[i] = temp + (_contentHeight[i] || 0);
                temp = _heightRecords[i];
            }
            this._heightRecords = _heightRecords;
        },

        scrollTabBar: function scrollTabBar(index) {
            var len = this.vtabs.length;
            if (len === 0) {
                return;
            }
            var currentView = index < 6 ? 0 : index - 5;
            if (currentView >= len) {
                currentView = len - 1;
            }
            this.setData({
                currentView: currentView
            });
        },

        handleTabClick: function handleTabClick(e) {
            var _heightRecords = this._heightRecords;
            var index = e.currentTarget.dataset.index;
            var contentScrollTop = _heightRecords[index - 1] || 0;
            this.$emit('tabclick', {
                detail: {
                    index: index
                }
            });
            this.setData({
                activeTabClone: index,
                contentScrollTop: contentScrollTop
            });
        },

        handleContentScroll: function handleContentScroll(e) {
            var _heightRecords = this._heightRecords;
            if (_heightRecords.length === 0) {
                return;
            }
            var length = this.vtabs.length;
            var scrollTop = e.detail.scrollTop;
            var index = 0;
            if (scrollTop >= _heightRecords[0]) {
                for (var i = 1; i < length; i++) {
                    if (scrollTop >= _heightRecords[i - 1] && scrollTop < _heightRecords[i]) {
                        index = i;
                        break;
                    }
                }
            }
            if (index !== this.activeTab) {
                this.$emit('change', {
                    detail: {
                        index: index
                    }
                });
                this.setData({
                    activeTabClone: index
                });
            }
        }
    },
    created: function () {}
};

/***/
</script>
<style>
@import './index.css';
</style>
