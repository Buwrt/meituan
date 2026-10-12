<template>
    <view :class="extClass">
        <view v-if="title" class="weui-cells__title">{{ title }}</view>
        <view :class="'weui-cells weui-cells_after-title ' + (checkboxCount > 0 && checkboxIsMulti ? 'weui-cells_checkbox' : '')">
            <slot></slot>
        </view>
        <view v-if="footer" class="weui-cells__tips">{{ footer }}</view>
        <template v-else><slot name="footer"></slot></template>
    </view>
</template>

<script>
export default {
    unicomGroup: ['checkbox-group'],
    data() {
        return {
            firstItem: null,
            checkboxCount: 0,
            checkboxIsMulti: false
        };
    },
    options: {
        addGlobalClass: true,
        multipleSlots: true
    },
    props: {
        title: {
            type: String,
            default: ''
        },
        extClass: {
            type: String,
            default: ''
        },
        footer: {
            type: String,
            default: ''
        }
    },
    relations: {
        '../cell/cell': {
            type: 'descendant',
            linked: function linked(target) {
                if (!this.firstItem) {
                    this.firstItem = target;
                }
                if (target !== this.firstItem) {
                    target.setOuterClass('weui-cell_wxss');
                }
            }
        },
        '../checkbox-group/checkbox-group': {
            type: 'descendant',
            linked: function linked(target) {
                this.setData({
                    checkboxCount: this.checkboxCount + 1,
                    checkboxIsMulti: target.data.multi
                });
            },
            unlinked: function unlinked(target) {
                this.setData({
                    checkboxCount: this.checkboxCount - 1,
                    checkboxIsMulti: target.data.multi
                });
            }
        }
    },
    methods: {
        setCellMulti: function setCellMulti(multi) {
            this.setData({
                checkboxIsMulti: multi
            });
        }
    },
    created: function () {}
};

/***/
</script>
<style>
@import './cells.css';
</style>
