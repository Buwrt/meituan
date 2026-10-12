<template>
    <view>
        <view :class="'weui-search-bar ' + extClass">
            <view class="weui-search-bar__form">
                <view class="weui-search-bar__box">
                    <icon class="weui-icon-search_in-box" type="search" size="12"></icon>
                    <input
                        type="text"
                        class="weui-search-bar__input"
                        :placeholder="placeholder ? placeholder : '搜索'"
                        :value="valueClone"
                        :focus="focusClone"
                        @blur="inputBlur"
                        @focus="inputFocus"
                        @input="inputChange"
                    />
                    <view class="weui-icon-clear" v-if="valueClone.length > 0" @tap="clearInput">
                        <icon type="clear" size="12"></icon>
                    </view>
                </view>
                <label class="weui-search-bar__label" v-if="!searchState" @tap="showInput">
                    <icon class="weui-icon-search" type="search" size="12"></icon>
                    <view class="weui-search-bar__text">{{ placeholder ? placeholder : '搜索' }}</view>
                </label>
            </view>
            <view v-if="cancel && searchState" class="weui-search-bar__cancel-btn" @tap="hideInput">{{ cancelText }}</view>
        </view>
        <mp-cells :class="'searchbar-result  ' + extClass" v-if="searchState && result.length > 0">
            <mp-cell @tap.native="selectResult($event, { index })" :data-index="index" hover v-for="(item, index) in result" :key="index">
                <view>{{ item.text }}</view>
            </mp-cell>
        </mp-cells>
    </view>
</template>

<script>
import mpCells from '../cells/cells';
import mpCell from '../cell/cell';

export default {
    components: {
        mpCells,
        mpCell
    },
    data() {
        return {
            result: [],
            searchState: false,
            valueClone: '',
            focusClone: false
        };
    },
    options: {
        addGlobalClass: true
    },
    props: {
        extClass: {
            type: String,
            default: ''
        },
        focus: {
            type: Boolean,
            default: false
        },
        placeholder: {
            type: String,
            default: ''
        },
        value: {
            type: String,
            default: ''
        },
        search: {
            type: Function,
            default: null
        },
        throttle: {
            type: Number,
            default: 500
        },
        cancelText: {
            type: String,
            default: '取消'
        },
        cancel: {
            type: Boolean,
            default: true
        }
    },
    lastSearch: Date.now(),
    mounted() {
        // 处理小程序 attached 生命周期
        this.attached();
    },
    methods: {
        attached: function attached() {
            if (this.focus) {
                this.setData({
                    searchState: true
                });
            }
        },

        clearInput: function clearInput() {
            this.setData({
                valueClone: ''
            });
            this.$emit('clear');
        },

        inputFocus: function inputFocus(e) {
            this.$emit('focus', {
                detail: e.detail
            });
        },

        inputBlur: function inputBlur(e) {
            this.setData({
                focusClone: false
            });
            this.$emit('blur', {
                detail: e.detail
            });
        },

        showInput: function showInput() {
            this.setData({
                focusClone: true,
                searchState: true
            });
        },

        hideInput: function hideInput() {
            this.setData({
                searchState: false,
                valueClone: ''
            });
            this.$emit('hide');
        },

        inputChange: function inputChange(e) {
            var that = this;
            this.setData({
                valueClone: e.detail.value
            });
            this.$emit('input', {
                detail: e.detail
            });
            if (Date.now() - this.lastSearch < this.throttle) {
                return;
            }
            if (typeof this.search !== 'function') {
                return;
            }
            this.lastSearch = Date.now();
            this.timerId = setTimeout(function () {
                that.search(e.detail.value)
                    .then(function (json) {
                        that.setData({
                            result: json
                        });
                    })
                    .catch(function (err) {
                        console.log('search error', err);
                    });
            }, this.throttle);
        },

        selectResult: function selectResult(e, _dataset) {
            /* ---处理dataset begin--- */
            this.handleDataset(e, _dataset);
            /* ---处理dataset end--- */
            var index = e.currentTarget.dataset.index;
            var item = this.result[index];
            this.$emit('selectresult', {
                detail: {
                    index: index,
                    item: item
                }
            });
        }
    },
    created: function () {},
    watch: {
        value: {
            handler: function (newVal, oldVal) {
                this.valueClone = newVal;
            },

            immediate: true
        },

        focus: {
            handler: function (newVal, oldVal) {
                this.focusClone = newVal;
            },

            immediate: true
        }
    }
};

/***/
</script>
<style>
@import './searchbar.css';
</style>
