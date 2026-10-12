<template>
    <view>
        <scroll-view
            @scrolltoupper="onPullDownRefresh"
            upper-threshold="-100"
            :style="'height:' + winHeight + 'px'"
            @scrolltolower="onReachBottom"
            lower-threshold="20"
            :scroll-x="false"
            :scroll-y="true"
            class="scroll-views"
        >
            <view class="delivery-address-list" v-if="addressList.length > 0">
                <view class="my-address">我的地址</view>
                <view class="delivery-address-item" v-for="(item, index) in addressList" :key="index">
                    <view class="address-detail-info">
                        <view class="address-name out_of_range one_row">{{ item.province }}{{ item.city }}{{ item.area }} {{ item.street }}</view>
                        <view class="personal-info">
                            <view class="default-address" v-if="item.isDefault">默认地址</view>
                            <!-- <view class="address-tag theme-bg">家</view> -->
                            <view class="phone-number">{{ item.phone }}</view>
                            <view class="username-class">{{ item.realname }}</view>
                        </view>
                    </view>

                    <text class="iconfont iconbianji-copy theme-color" @tap="editAddressTap" :data-data="item" :data-key="index"></text>
                </view>
            </view>
            <is-show-tip
                :isShow="addressList.length <= 0"
                @bindTap="goToDrink"
                iconfont=""
                buttonText="去喝一杯"
                :top="45"
                :bottom="10"
                :text="addressList.length <= 0"
                tipText="您还没有地址信息"
                :button="addressList.length > 0"
            ></is-show-tip>
        </scroll-view>
        <navigator url="../insert/insert" class="insert-address-view theme-color self-adaption">+新增地址</navigator>
    </view>
</template>

<script>
import isShowTip from '../../../components/is-show-tip/is-show-tip';
import GlobalConfig from '../../../utils/global-config';
import https from '../../../utils/http';
import authService from '../../../utils/auth';
import toastService from '../../../utils/toast.service';
//获取应用实例
const app = getApp();
var pageNo = -1;
var prevList = [];
export default {
    components: {
        isShowTip
    },
    data() {
        return {
            list: [],
            isUpdate: false,

            //是否修改
            isInsert: false,

            isDelete: false,
            winHeight: '',
            addressList: '',
            refreshKey: ''
        };
    },
    /**
     * 生命周期函数--监听页面加载
     */
    onLoad: function (options) {
        pageNo = 1;
        this.getDeliveryAddressList();
    },
    /**
     * 生命周期函数--监听页面初次渲染完成
     */
    onReady: function () {
        this.selfAdaption();
    },
    /**
     * 生命周期函数--监听页面显示
     */
    onShow: function () {
        // if (this.data.isOperation){
        //   //this.getDeliveryAddressList();
        //   this.data.list[this.data.refreshKey] = this.data.refreshData;
        //   this.setData({
        //     list: this.data.list,
        //     isOperation: false
        //   })
        // }
        // if (this.data.isUpdate) {
        //   this.data.list.forEach(key => {
        //     if (this.data.refreshData.isDefault == 1) {
        //       key.isDefault = 0;
        //     }
        //   })
        //   this.data.list[this.data.refreshKey] = this.data.refreshData;
        //   this.setData({
        //     list: this.data.list,
        //     isUpdate: false
        //   })
        // }
        // if (this.data.isInsert) {
        //   toastService.showLoading("正在加载...", true);
        //   this.setData({
        //     list: [],
        //     isInsert: false
        //   })
        //   this.getDeliveryAddressList();
        // }
        // if (this.data.isDelete) {
        //   this.data.list.splice(this.data.refreshKey, 1);
        //   this.setData({
        //     list: this.data.list,
        //     isDelete: false
        //   })
        // }
    },
    /**
     * 生命周期函数--监听页面隐藏
     */
    onHide: function () {},
    /**
     * 生命周期函数--监听页面卸载
     */
    onUnload: function () {},
    /**
     * 页面相关事件处理函数--监听用户下拉动作
     */
    onPullDownRefresh: function () {
        //pageNo = 1;
        // 显示顶部刷新图标
        uni.showNavigationBarLoading();
        this.list = [];
        // 设置数组元素
        this.getDeliveryAddressList();
        // 隐藏导航栏加载框
        uni.hideNavigationBarLoading();
        // 停止下拉动作
        uni.stopPullDownRefresh();
    },
    /**
     * 页面上拉触底事件的处理函数
     */
    onReachBottom: function () {
        // // 显示加载图标
        // toastService.showLoading();
        // // 页数+1
        // pageNo = pageNo + 1;
        // https.request('/api-user/rest/member/deliveryAddress/list', {
        //   pageNo: pageNo,
        //   pageSize: pageSize
        // }).then(result => {
        //   toastService.hideLoading();
        //   if (result.success) {
        //     var list = this.data.list;
        //     if (result.data.records.length > 0) {
        //       result.data.records.forEach(res => {
        //         this.data.list.push(
        //           res
        //         )
        //       })
        //       this.setData({
        //         list: this.data.list
        //       });
        //       return false;
        //     }
        //     toastService.showToast("没有更多啦~");
        //     pageNo = pageNo - 1;
        //   }
        // })
    },
    /**
     * 用户点击右上角分享
     */
    onShareAppMessage: function () {},
    methods: {
        selfAdaption() {
            var that = this;
            let winHeight = 0;
            let height = 0;
            uni.getSystemInfo({
                success: function (res) {
                    //获取到用户的手机信息
                    winHeight = res.windowHeight;
                    // 获取需要减去的dom结构的高度信息
                    uni.createSelectorQuery()
                        .in(uni)
                        .selectAll('.self-adaption')
                        .boundingClientRect(function (rects) {
                            rects.forEach(function (rect, index) {
                                height = height + rect.height;
                            });
                            that.setData({
                                winHeight: winHeight - height
                            });
                        })
                        .exec();
                }
            });
        },

        getDeliveryAddressList() {
            toastService.showLoading();
            https
                .request('/api-user/rest/member/deliveryAddress/list', {
                    pageNo: -1,
                    pageSize: 10
                })
                .then((result) => {
                    toastService.hideLoading();
                    if (result.success) {
                        this.setData({
                            addressList: result.data.records
                        });
                    }
                });
        },

        editAddressTap(e) {
            uni.navigateTo({
                url: '../edit/edit?detail=' + JSON.stringify(e.currentTarget.dataset.data)
            });
            this.setData({
                refreshKey: e.currentTarget.dataset.key
            });
        },

        onPullDownRefresh() {
            console.log('占位：函数 onPullDownRefresh 未声明');
        },

        onReachBottom() {
            console.log('占位：函数 onReachBottom 未声明');
        },

        goToDrink() {
            console.log('占位：函数 goToDrink 未声明');
        }
    }
};
</script>
<style>
@import './index.css';
</style>
