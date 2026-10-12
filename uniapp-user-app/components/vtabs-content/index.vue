<template>
    <view class="weui-vtabs-content__item" :id="'weui-vtabs-content__' + tabIndex">
        <slot></slot>
    </view>
</template>

<script>
export default {
    data() {
        return {};
    },
    unicomGroup: ['vtabs'],
    options: {
        addGlobalClass: true,
        multipleSlots: true
    },
    props: {
        tabIndex: {
            type: Number,
            default: 0
        }
    },
    relations: {
        '../vtabs/index': {
            type: 'parent'
        }
    },
    mounted() {
        // 处理小程序 attached 生命周期
        this.attached();
    },
    methods: {
        attached: function attached() {},

        calcHeight: function calcHeight(callback) {
            var query = uni.createSelectorQuery().in(this);
            query
                .select('.weui-vtabs-content__item')
                .boundingClientRect(function (rect) {
                    if (callback) {
                        callback(rect);
                    }
                })
                .exec();
        }
    },
    created: function () {}
};

/***/
</script>
<style>
@import './index.css';
</style>
