<template>
    <view :class="'weui-badge ' + extClass + ' ' + (!content ? 'weui-badge_dot' : '')">{{ content }}</view>
</template>

<script>
export default {
    data() {
        return {};
    },
    options: {
        addGlobalClass: true
    },
    props: {
        extClass: {
            type: String,
            default: ''
        },
        content: {
            type: String,
            default: ''
        }
    },
    methods: {},
    created: function () {}
};

/***/
</script>
<style>
@import './badge.css';
</style>
