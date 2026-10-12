<template>
    <view>
        <view class="top-carousel-swiper" v-if="carouselUrls.length > 0">
            <swiper :indicator-dots="indicatorDots" class="carousel-swiper" :autoplay="autoplay" :interval="interval" :duration="duration" :indicator-active-color="afterColor">
                <block v-for="(item, index) in carouselUrls" :key="index">
                    <swiper-item class="carousel-swiper-item">
                        <image :src="item.imagePath" class="carousel-image" mode="aspectFill" :data-imageLinkUrl="item.imageLinkUrl" @tap="carouseCommodityDetailTap" />
                    </swiper-item>
                </block>
            </swiper>
        </view>
        <view class="menu-items">
            <view class="menu-item" @tap="bindToMenu" :data-shopid="item.shipId" hover-class="hover-class-public" v-for="(item, index) in classifyList" :key="index">
                <image mode="aspectFit" class="menu-image" :src="item.iconUrl" v-if="item.icon"></image>

                <view class="menu-title out_of_range one_row">{{ item.name }}</view>
            </view>
        </view>

        <view class="like-items">
            <view
                class="like-item"
                hover-class="hover-class-public"
                @tap="commodityDetailTap"
                :data-shopid="item.shopId"
                :data-id="item.id"
                v-for="(item, index) in recommendGoodsList"
                :key="index"
            >
                <image :src="item.mainImage" mode="scaleToFill" class="icon-like-class"></image>

                <view class="fullname-class">
                    <text class="name-text out_of_range two_row">{{ item.name }}</text>
                </view>

                <view class="like-money-view">
                    <view class="like-money">
                        <text>￥</text>
                        <text>
                            {{ item.price }}
                            <text class="score-money" :decode="true">&nbsp;&nbsp;(可用积分等额兑换)</text>
                        </text>
                    </view>
                </view>

                <view class="like-money-view">
                    <text class="latelyMonthlySales out_of_range one_row">已售：{{ item.latelyMonthlySales }}</text>
                    <view class="insert-car" @tap.stop.prevent="openSpecifications" :data-index="index">
                        <text class="iconfont icongouwuche1 theme-bg"></text>
                    </view>
                </view>
            </view>
        </view>
        <view v-if="isMore" class="list-ismore">—————— 没有更多啦 ——————</view>
        <mp-halfScreenDialog :show="specificationsDialog" extClass="extClassSpecifications">
            <view slot="title">选择规格</view>
            <view slot="desc">
                <view class="goods-info-view">
                    <image :src="goods.mainImage" mode="aspectFill" class="commodity-image"></image>
                    <view>
                        <view class="goods-info-name out_of_range one_row">{{ goods.name }}</view>
                        <view class="goods-info-specListString">已选：{{ specListString }}</view>
                        <view class="goods-info-price">￥{{ priceAfter }}</view>
                    </view>
                </view>
                <scroll-view scroll-y style="height: 52vh">
                    <view class="commdity-name-type-view">
                        <!-- <view class="commdity-name">{{data.name}}</view> -->
                        <view class="commdity-type-item" v-if="specList" v-for="(item, key) in specList" :key="key">
                            <view class="commdity-type-name">{{ key }}</view>

                            <radio-group class="radio-group" @change="radioChange" :data-firstIndex="key">
                                <label
                                    :class="
                                        'group-label theme-border ' +
                                        (!item.stock ? 'disabled-group-label' : '') +
                                        ' ' +
                                        (item.checked ? 'active theme-bg' : 'theme-color-border') +
                                        ' out_of_range one_row'
                                    "
                                    v-for="(item, index) in item"
                                    :key="index"
                                >
                                    <radio :value="index" :checked="item.checked" :disabled="!item.stock" class="radio" />

                                    {{ item.name }}
                                </label>
                            </radio-group>
                        </view>
                        <is-show-tip :isShow="!specList" :top="20" :bottom="20" :text="!specLis" tipText="暂无规格"></is-show-tip>
                    </view>
                </scroll-view>

                <view slot="footer">
                    <view class="good-choice-btn theme-bg" @tap="insertShoppingCart" :data-goodsid="goods.id">我选好了</view>
                </view>
            </view>
        </mp-halfScreenDialog>
    </view>
</template>

<script>
import mpHalfScreenDialog from '../../../../components/half-screen-dialog/half-screen-dialog';
import mpBadge from '../../../../components/badge/badge';
import isShowTip from '../../../../components/is-show-tip/is-show-tip';
import GlobalConfig from '../../../../utils/global-config';
import https from '../../../../utils/http';
import toastService from '../../../../utils/toast.service';
import authService from '../../../../utils/auth';
const app = getApp();
var pageNo = 1;
var pageSize = 10;
export default {
    components: {
        mpHalfScreenDialog,
        mpBadge,
        isShowTip
    },
    data() {
        return {
            indicatorDots: true,
            autoplay: true,
            interval: 5000,
            duration: 1000,

            //beforeColor: "white",//指示点颜色,
            afterColor: '#f1a142',

            //当前选中的指示点颜色
            opacity: 0.4,

            recommendGoodsList: [],
            specList: [],
            isMore: false,
            carouselUrls: '',
            classifyList: '',
            priceAfter: '',
            specListString: '',
            specificationsDialog: false,

            goods: {
                mainImage: '',
                name: '',
                id: ''
            },

            specLis: ''
        };
    },
    /**
     * 生命周期函数--监听页面加载
     */
    onLoad: function (options) {
        pageNo = 1;
        this.getSystemUsageRecord();
        this.getRecommendGoodsList();
    },
    /**
     * 生命周期函数--监听页面初次渲染完成
     */
    onReady: function () {},
    /**
     * 生命周期函数--监听页面显示
     */
    onShow: function () {
        //pageNo=1;
        app.globalData.getShoppingCarNumber();
        this.getClassifyList();
        //this.getRecommendGoodsList();
        this.getCarouselList();
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
        pageNo = 1;
        this.getRecommendGoodsList();
    },
    /**
     * 页面上拉触底事件的处理函数
     */
    onReachBottom: function () {
        console.log('1111111111');
        toastService.showLoading('正在加载...');
        var that = this;
        pageNo = pageNo + 1;
        https
            .request('/api-mall/rest/pointsMall/goods/recommendGoodsList', {
                pageNo: pageNo,
                pageSize: pageSize
            })
            .then((result) => {
                toastService.hideLoading();
                if (result.success) {
                    console.log(result.data);
                    if (result.data.records.length <= 0) {
                        pageNo = pageNo - 1;
                        that.setData({
                            isMore: result.data.hasNextPage ? false : true
                        });
                        //toastService.showToast("没有更多啦~");
                        return;
                    }
                    result.data.records.forEach(function (item, index) {
                        item.mainImage = GlobalConfig.ossUrl + item.mainImage;
                        that.recommendGoodsList.push(item);
                    });
                    console.log(this.recommendGoodsList);
                    this.setData({
                        recommendGoodsList: this.recommendGoodsList,
                        isMore: result.data.hasNextPage ? false : true
                    });
                }
            });
    },
    /**
     * 用户点击右上角分享
     */
    onShareAppMessage: function () {},
    methods: {
        carouseCommodityDetailTap(e) {
            let _this = this;
            authService.checkIsLogin().then((result) => {
                console.log(result);
                if (result) {
                    _this.getUserInfo(e);
                    return;
                }
                app.globalData.checkIsAuth('scope.userInfo');
            });
        },

        getUserInfo: function (e) {
            https.request('/api-user/rest/member/getLoginMemberInfo', {}).then((result) => {
                if (result.success) {
                    uni.navigateTo({
                        url: e.currentTarget.dataset.imagelinkurl
                    });
                } else {
                    app.globalData.checkIsAuth('scope.userInfo');
                }
            });
        },

        getCarouselList() {
            https
                .request('/api-promotion/rest/advertisement/list', {
                    type: 3,
                    pageNo: -1,
                    pageSize: 20
                })
                .then((result) => {
                    if (result.success) {
                        result.data.records.forEach(function (item, index) {
                            item.imagePath = GlobalConfig.ossUrl + item.imagePath;
                        });
                        this.setData({
                            carouselUrls: result.data.records
                        });
                    }
                });
        },

        getClassifyList() {
            https
                .request('/api-mall/rest/pointsMall/menu/list', {
                    pageNo: 1,
                    pageSize: 8
                })
                .then((result) => {
                    if (result.success) {
                        result.data.records.forEach(function (item, index) {
                            item.iconUrl = GlobalConfig.ossUrl + item.icon;
                        });
                        this.setData({
                            classifyList: result.data.records
                        });
                    }
                });
        },

        getRecommendGoodsList() {
            toastService.showLoading();
            https
                .request('/api-mall/rest/pointsMall/goods/recommendGoodsList', {
                    pageNo: pageNo,
                    pageSize: pageSize
                })
                .then((result) => {
                    toastService.hideLoading();
                    uni.stopPullDownRefresh();
                    if (result.success) {
                        result.data.records.forEach(function (item, index) {
                            item.mainImage = GlobalConfig.ossUrl + item.mainImage;
                        });
                        this.recommendGoodsList = [];
                        this.setData({
                            recommendGoodsList: result.data.records,
                            isMore: result.data.hasNextPage ? false : true
                        });
                    }
                });
        },

        commodityDetailTap(e) {
            console.log(e.currentTarget.dataset.id);
            uni.navigateTo({
                url: '../detail/detail?id=' + e.currentTarget.dataset.id
            });
        },

        bindToMenu(e) {
            uni.navigateTo({
                url: '../menu/menu'
            });
        },

        getSystemUsageRecord() {
            https
                .request('/api-util/rest/systemUsageRecord/insert', {
                    type: 'intoPointsMall'
                })
                .then((result) => {
                    if (result.success) {
                    }
                });
        },

        openSpecifications(e) {
            this.selectByGoodsId(e.currentTarget.dataset.index);
        },

        radioChange(e) {
            //获取复选框选中的下标值
            var checkValue = e.detail.value;
            //获取第一级分类的下标值
            let firstIndex = e.currentTarget.dataset.firstindex;
            //获取所有分类信息
            let specList = this.specList;
            //console.log(specList)
            //遍历分类信息给第一级分类为false，提交的时候对应各级分类
            for (var j in specList[firstIndex]) {
                specList[firstIndex][j].checked = false;
            }
            //给选中的第二级分类的checked设置为true
            for (var i in checkValue) {
                specList[firstIndex][checkValue[i]].checked = true;
                //console.log(specList[firstIndex][checkValue[i]])
            }
            let price = this.goods.price;
            let specListString = '';
            for (let key in specList) {
                for (let keyof in specList[key]) {
                    //console.log(specList[key][keyof].price)
                    if (specList[key][keyof].checked) {
                        price = price + specList[key][keyof].price;
                        specListString = (specListString ? specListString + '/' : specListString) + specList[key][keyof].name;
                    }
                }
            }
            console.log(specListString);
            this.setData({
                specList: specList,
                priceAfter: price,
                specListString: specListString
            });
        },

        insertShoppingCart(e) {
            authService.checkIsLogin().then((result) => {
                toastService.showLoading();
                if (result) {
                    let goodsSpecs = {};
                    let specList = this.specList;
                    for (let key in specList) {
                        for (let keyof in specList[key]) {
                            //拼接查询规格等的json数据格式，查询商品规格等对应的价格
                            if (specList[key][keyof].checked) {
                                goodsSpecs[key] = specList[key][keyof].name;
                            }
                        }
                    }
                    //console.log(goodsSpecs)
                    let data = {
                        goodsId: e.currentTarget.dataset.goodsid,
                        specList: JSON.stringify(goodsSpecs)
                    };
                    toastService.hideLoading();
                    https.request('/api-mall/rest/member/pointsMall/shoppingCart/insert', data, authService.getToken()).then((result) => {
                        if (result.success) {
                            toastService.showToast('加入购物车成功');
                            this.setData({
                                specificationsDialog: false
                            });
                            app.globalData.getShoppingCarNumber();
                        }
                    });
                    return;
                }
                toastService.hideLoading();
                app.globalData.checkIsAuth('scope.userInfo');
            });
        },

        setTabBarBadge(num) {
            if (num) {
                uni.setTabBarBadge({
                    index: 2,
                    text: String(num)
                });
                return;
            }
            uni.removeTabBarBadge({
                index: 2
            });
        },

        selectByGoodsId(index) {
            let goods = this.recommendGoodsList[index];
            https
                .request('/api-mall/rest/pointsMall/goodsSpecificationOption/selectByGoodsId', {
                    goodsId: goods.id
                })
                .then((result) => {
                    if (result.success) {
                        //重新设置商品的规格等数据的格式
                        //let goodsSpecs = {};
                        let specList = result.data;
                        let price = goods.price;
                        let specListString = '';
                        for (let key in specList) {
                            let isChecked = true;
                            for (let keyof in specList[key]) {
                                //拼接查询规格等的json数据格式，查询商品规格等对应的价格
                                specList[key][keyof].checked = false;
                                //设置每个规格的第一个选项为选中，当库存为0时则选中下一个
                                if (specList[key][keyof].stock == 1 && isChecked) {
                                    specList[key][keyof].checked = true;
                                    //选中的规格价钱在商品价钱的基础上累加
                                    price = price + specList[key][keyof].price;
                                    specListString = (specListString ? specListString + '/' : specListString) + specList[key][keyof].name;
                                    isChecked = false;
                                }
                            }
                        }
                        console.log(goods);
                        console.log(specList);
                        this.setData({
                            specListString: specListString ? specListString : '暂无规格',
                            specList: JSON.stringify(specList) == '{}' ? null : specList,
                            goods: goods,
                            specificationsDialog: true,
                            priceAfter: price
                        });
                    }
                });
        }
    }
};
</script>
<style>
@import './index.css';
</style>
