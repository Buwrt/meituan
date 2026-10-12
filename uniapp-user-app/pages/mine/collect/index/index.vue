<template>
    <view>
        <view style="height: 100%">
            <view class="position-sticky">
                <view class="swiper-content">
                    <view class="swiper-tab collect-tab self-adaption">
                        <view class="swiper-tab-item" :data-current="index" @tap="clickCollectTab" hover-class="hover-click-class" v-for="(item, index) in tabList" :key="index">
                            <view :class="'swiper_table_item_view ' + (collectTab == item.modeId ? 'active' : '')" :data-current="index" @tap="clickCollectTab">
                                {{ item.modeName }}
                            </view>
                        </view>
                    </view>
                </view>
                <view>
                    <mp-searchbar @selectresult="selectResult" :search="search" @clear="searchInputClear" @hide="searchInputClear" placeholder="商品名称"></mp-searchbar>
                </view>
                <view class="swiper-content">
                    <view class="swiper-tab self-adaption">
                        <view class="swiper-tab-item" :data-current="index" @tap="clickTab" hover-class="hover-click-class" v-for="(item, index) in modeList" :key="index">
                            <view :class="'swiper_table_item_view ' + (currentTab == item.modeId ? 'active' : '')" :data-current="index" @tap="clickTab">
                                {{ item.modeName }}
                            </view>
                        </view>
                    </view>
                    <view class="manager-checkbox" v-if="!isShowCheckbox" @tap="bindShowCheckbox">管理</view>
                    <view class="manager-checkbox theme-color" v-else @tap="bindShowCheckbox">完成</view>
                </view>
            </view>
            <checkbox-group class="weui-slidecells" @change="checkboxChange">
                <view class="page__bd">
                    <view class="weui-slidecells" v-for="(item, index) in collectList" :key="index">
                        <mp-slideview
                            :buttons="slideButtons"
                            :icon="true"
                            :data-goodsid="item.goodsId"
                            @buttontap="slideButtonTap($event, { goodsid: item.goodsId })"
                            throttle="300"
                        >
                            <view :class="'weui-slidecell ' + (item.disable ? 'isDisable' : '')">
                                <label class="checkbox-group-label" v-if="isShowCheckbox">
                                    <checkbox :value="index" :index="index" class="theme-color theme-border-color" :checked="item.checked" :disabled="item.disable" />
                                </label>
                                <view
                                    class="commdity-item"
                                    :data-id="item.goodsId"
                                    :data-shopid="item.shopId"
                                    @tap.stop.prevent="parseEventDynamicCode($event, collectTab == 0 ? 'bindMemberDetails' : 'bindPointsMallDetails')"
                                >
                                    <image :src="item.mainImage ? item.mainImage : '../../assets/images/load-image.png'" mode="aspectFill" class="commodity-icon"></image>
                                    <view class="sell-out out-store" v-if="item.goodsStatus == 1">未上架</view>
                                    <view class="sell-out out-store" v-else-if="item.goodsStatus == 3">已下架</view>
                                    <view class="sell-out out-store" v-else-if="item.goodsStatus == 4">已售罄</view>
                                    <view class="commdity-types">
                                        <view class="commodity-name-type">
                                            <view class="commdity-name">{{ item.goodsName }}</view>
                                            <view class="types">{{ item.restructure }}</view>
                                        </view>
                                        <view class="commdity-money strike_through" v-if="isSale">￥{{ item.goodsPrice }}</view>
                                        <view class="commdity-money">￥{{ item.isSale ? item.salePrice : item.goodsPrice }}</view>
                                        <text class="is-goods-exists" v-if="!item.isGoodsExists">已失效</text>
                                    </view>
                                </view>
                            </view>
                        </mp-slideview>
                    </view>
                </view>
            </checkbox-group>
        </view>
        <view v-if="isShowCheckbox" class="show-checkbox">
            <view class="checkbox-class">
                <checkbox class="all-checkbox" @tap="allChange" :checked="isAlls">全选</checkbox>
            </view>
            <text class="iconfont iconshanchuguan" @tap="batchDelete"></text>
        </view>
        <is-show-tip :isShow="collectList.length <= 0" :src="noDataTip" :top="25" :bottom="20" :text="collectList.length <= 0" tipText="暂无收藏"></is-show-tip>
    </view>
</template>

<script>
import mpSearchbar from '../../../../components/searchbar/searchbar';
import mpSlideview from '../../../../components/slideview/slideview';
import isShowTip from '../../../../components/is-show-tip/is-show-tip';
import GlobalConfig from '../../../../utils/global-config';
import https from '../../../../utils/http';
import toastService from '../../../../utils/toast.service';
import authService from '../../../../utils/auth';
import utilHelper from '../../../../utils/util';
import CustomPage from '../../../../base/CustomPage';
import dateHelper from '../../../../utils/date-helper';
var base64 = require('../../../../components/images/base64');
export default {
    components: {
        mpSearchbar,
        mpSlideview,
        isShowTip
    },
    data() {
        return {
            inputShowed: false,
            inputVal: '',
            i: 0,
            currentTab: 0,

            modeList: [
                {
                    modeId: 0,
                    modeName: '全部'
                },
                {
                    modeId: 1,
                    modeName: '已买过'
                }
            ],

            tabList: [
                {
                    modeId: 0,
                    modeName: '外卖收藏'
                },
                {
                    modeId: 1,
                    modeName: '商城收藏'
                }
            ],

            collectTab: 0,
            checkValue: 0,
            isShowCheckbox: false,
            collectList: [],
            icon: base64.icon20,

            slideButtons: [
                {
                    src: '/static/assets/common/icon-del.svg' // icon的路径
                }
            ],

            isAlls: false,
            searchGoodsName: '',
            search: '',
            goodsIdList: '',
            isSale: false,
            noDataTip: ''
        };
    },
    /**
     * 生命周期函数--监听页面加载
     */
    onLoad: function (options) {
        this.setData({
            search: this.search.bind(this)
        });
        this.getMemberCollect();
    },
    /**
     * 生命周期函数--监听页面初次渲染完成
     */
    onReady: function () {},
    /**
     * 生命周期函数--监听页面显示
     */
    onShow: function () {},
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
    onPullDownRefresh: function () {},
    /**
     * 页面上拉触底事件的处理函数
     */
    onReachBottom: function () {},
    /**
     * 用户点击右上角分享
     */
    onShareAppMessage: function () {},
    methods: {
        // 滑动切换tab
        bindSlideChange: function (e) {
            this.setData({
                currentTab: e.detail.current
            });
            // this.getHeight();
        },

        //点击切换
        clickTab: function (e) {
            if (this.currentTab === e.target.dataset.current) {
                return false;
            } else {
                if (this.collectTab == 0) {
                    if (e.target.dataset.current == 0) {
                        this.getMemberCollect(false, this.searchGoodsName);
                    } else {
                        this.getMemberCollect(true, this.searchGoodsName);
                    }
                }
                if (this.collectTab == 1) {
                    if (e.target.dataset.current == 0) {
                        this.getPointsMallCollect(false, this.searchGoodsName);
                    } else {
                        this.getPointsMallCollect(true, this.searchGoodsName);
                    }
                }
                this.setData({
                    currentTab: e.target.dataset.current
                });
                // this.getHeight();
            }
        },

        clickCollectTab: function (e) {
            if (this.collectTab === e.target.dataset.current) {
                return false;
            } else {
                if (e.target.dataset.current == 0) {
                    this.getMemberCollect(false, this.searchGoodsName);
                } else {
                    this.getPointsMallCollect(false, this.searchGoodsName);
                }
                this.setData({
                    collectTab: e.target.dataset.current,
                    currentTab: 0,
                    isAlls: false
                });
                // this.getHeight();
            }
        },

        searchFun: function (value) {
            console.log(value);
            return new Promise((resolve, reject) => {
                //判断大标签页
                if (this.collectTab == 0) {
                    //判断小标签页
                    if (this.currentTab == 0) {
                        this.getMemberCollect(false, value);
                    }
                    if (this.currentTab == 1) {
                        this.getMemberCollect(true, value);
                    }
                    resolve([]);
                }
                if (this.collectTab == 1) {
                    //判断小标签页
                    if (this.currentTab == 0) {
                        this.getPointsMallCollect(false, value);
                    }
                    if (this.currentTab == 1) {
                        this.getPointsMallCollect(true, value);
                    }
                    resolve([]);
                }
                this.setData({
                    searchGoodsName: value
                });
            });
        },

        selectResult: function (e) {
            console.log('select result', e.detail);
        },

        searchInputClear(e) {
            console.log(e);
            if (this.collectTab == 0) {
                //判断小标签页
                if (this.currentTab == 0) {
                    this.getMemberCollect(false, '');
                }
                if (this.currentTab == 1) {
                    this.getMemberCollect(true, '');
                }
            }
            if (this.collectTab == 1) {
                //判断小标签页
                if (this.currentTab == 0) {
                    this.getPointsMallCollect(false, '');
                }
                if (this.currentTab == 1) {
                    this.getPointsMallCollect(true, '');
                }
            }
        },

        getPointsMallCollect(isBuy, goodsName) {
            let data = {
                pageNo: -1,
                pageSize: 20
            };
            if (isBuy) {
                data.isBuy = true;
            }
            if (goodsName) {
                data.goodsName = goodsName;
            }
            this.collectList = [];
            https.request('/api-mall/rest/member/pointsMall/goodsCollect/list', data).then((result) => {
                if (result.success) {
                    result.data.records.forEach((item, index) => {
                        item.mainImage = GlobalConfig.ossUrl + item.mainImage;
                        this.collectList.push(item);
                    });
                    this.setData({
                        collectList: this.collectList
                    });
                }
            });
        },

        getMemberCollect(isBuy, goodsName) {
            let data = {
                pageNo: -1,
                pageSize: 20
            };
            if (isBuy) {
                data.isBuy = true;
            }
            if (goodsName) {
                data.goodsName = goodsName;
            }
            this.collectList = [];
            https.request('/api-goods/rest/member/goodsCollect/list', data).then((result) => {
                if (result.success) {
                    result.data.records.forEach((item, index) => {
                        item.mainImage = GlobalConfig.ossUrl + item.mainImage;
                        this.collectList.push(item);
                    });
                    this.setData({
                        collectList: this.collectList
                    });
                }
            });
        },

        bindShowCheckbox(e) {
            this.setData({
                isShowCheckbox: this.isShowCheckbox ? false : true
            });
        },

        bindPointsMallDetails(e) {
            uni.navigateTo({
                url: '../../../mall/detail/detail?id=' + e.currentTarget.dataset.id
            });
        },

        bindMemberDetails(e) {
            uni.navigateTo({
                url: '../../../menu/detail/detail?id=' + e.currentTarget.dataset.id + '&shopId=' + e.currentTarget.dataset.shopid
            });
        },

        checkboxChange(e) {
            console.log(e);
            var values = e.detail.value;
            var deleteIds = [];
            for (var key in values) {
                deleteIds.push(this.collectList[key].goodsId);
            }
            this.setData({
                goodsIdList: deleteIds,
                isAlls: values.length != this.collectList.length ? false : true
            });
        },

        allChange(e) {
            console.log(e);
            var deleteIds = [];
            this.collectList.forEach((item, index) => {
                if (!this.isAlls) {
                    item.checked = true;
                    deleteIds.push(item.goodsId);
                } else {
                    item.checked = false;
                }
            });
            console.log(deleteIds);
            this.setData({
                collectList: this.collectList,
                goodsIdList: deleteIds,
                isAlls: this.isAlls ? false : true
            });
        },

        slideButtonTap(e, _dataset) {
            /* ---处理dataset begin--- */
            this.handleDataset(e, _dataset);
            /* ---处理dataset end--- */
            var deleteIds = [];
            let goodsid = e.currentTarget.dataset.goodsid; //商品下标
            deleteIds.push(goodsid);
            this.setData({
                goodsIdList: deleteIds
            });
            this.batchDelete();
        },

        batchDelete() {
            if (this.goodsIdList.length <= 0) {
                toastService.showToast('请选择要操作的数据');
                return;
            }
            var that = this;
            toastService.showModal(
                null,
                '确定要删除这' + this.goodsIdList.length + '条数据吗?',
                function confirm() {
                    let url = '/api-mall/rest/member/pointsMall/goodsCollect/batchDelete';
                    if (that.collectTab == 0) {
                        url = '/api-goods/rest/member/goodsCollect/batchDelete';
                    }
                    https
                        .request(url, {
                            goodsIdList: that.goodsIdList
                        })
                        .then((result) => {
                            if (result.success) {
                                toastService.showToast(result.message);
                                //判断大标签页
                                if (that.collectTab == 0) {
                                    //判断小标签页
                                    if (that.currentTab == 0) {
                                        that.getMemberCollect();
                                    }
                                    if (that.currentTab == 1) {
                                        that.getMemberCollect(true);
                                    }
                                }
                                if (that.collectTab == 1) {
                                    //判断小标签页
                                    if (that.currentTab == 0) {
                                        that.getPointsMallCollect();
                                    }
                                    if (that.currentTab == 1) {
                                        that.getPointsMallCollect(true);
                                    }
                                }
                            }
                        });
                },
                null,
                true
            );
        }
    }
};
</script>
<style>
@import './index.css';
</style>
