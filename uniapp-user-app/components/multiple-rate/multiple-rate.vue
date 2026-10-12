<template>
    <view class="multiple-rate">
        <view>
            <view @tap="handleTap" :data-index="index" :class="item" v-for="(item, index) in starArr" :key="index"></view>
            <slot></slot>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            starArr: [],
            isHandleTap: false,
            rateClone: ''
        };
    },
    mixins: [],
    props: {
        rate: {
            type: String,
            default: '0'
        },
        icon: {
            type: String,
            default: 'star'
        },
        disabled: {
            type: Boolean,
            default: false
        }
    },
    mounted() {
        // 处理小程序 attached 生命周期
        this.attached();
    },
    moved: function () {},
    destroyed: function () {},
    methods: {
        attached: function () {
            this.getStarArr();
        },

        getStarArr: function () {
            let starArr = [];
            for (var i = 0; i < this.rate; i++) {
                starArr.push(this.icon);
            }
            for (let j = 0; j < 5 - this.rate; j++) {
                starArr.push(this.icon + '-o');
            }
            this.setData({
                starArr: starArr
            });
        },

        handleTap: function (e) {
            if (this.disabled) {
                console.log(this.rate);
                return;
            }
            this.setData({
                rateClone: Number(e.currentTarget.dataset.index) + 1
            });
            console.log(this.rate);
            this.$emit('change', {
                detail: {
                    value: Number(e.currentTarget.dataset.index) + 1
                }
            });
            this.getStarArr();
        }
    },
    created: function () {},
    watch: {
        rate: {
            handler: function (newVal, oldVal) {
                this.rateClone = newVal;
            },

            immediate: true
        }
    }
};
</script>
<style>
@import './multiple-rate.css';
</style>
