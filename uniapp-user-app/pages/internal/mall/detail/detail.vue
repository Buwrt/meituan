<template>
    <view>
        <view class="wodejl-view" @tap="bindShare">分享商品>></view>

        <swiper :indicator-dots="indicatorDots" :autoplay="autoplay" class="carousel-swiper" :interval="interval" :duration="duration" :indicator-active-color="afterColor">
            <block v-for="(item, index) in carouselUrls" :key="index">
                <swiper-item class="carousel-image">
                    <image :src="item" class="carousel-image" mode="scaleToFill" @tap="uploadImage" />
                </swiper-item>
            </block>
        </swiper>
        <view class="commdity-name-type-view">
            <view class="display_flex_between">
                <view class="flex_column commdity-name">
                    <text class="name out_of_range two_row">{{ data.name }}</text>
                    <text :decode="true" class="price">
                        ￥{{ priceAfter }}
                        <text class="score-money" :decode="true">&nbsp;&nbsp;(可用积分等额兑换)</text>
                    </text>
                </view>
                <view class="flex_column collect-in">
                    <text
                        :class="'iconfont iconshoucang1 ' + (isCollect ? 'is-collect' : '')"
                        @tap="parseEventDynamicCode($event, isCollect ? 'bindDeleteCollect' : 'bindInsertCollect')"
                        :data-goodsid="data.id"
                    ></text>
                    <text :class="'shoucang-text ' + (isCollect ? 'is-collect' : 'not-collect')">收藏</text>
                </view>
            </view>
        </view>
        <view class="commodity-detail-view">
            <view class="guige-yunfei guige-view">
                <text class="commodity-detail">{{ specListString ? '已选择' : '' }}规格</text>
                <view class="specifications-line flex_end" @tap="openSpecifications" data-type="car">
                    <view :class="specListString ? 'guige-specListString theme-color' : 'goods-info-specListString'">{{ specListString ? specListString : '未选择商品规格' }}</view>
                    <text class="iconfont iconhtbArrowright02"></text>
                </view>
            </view>
        </view>
        <view class="commodity-detail-view">
            <view class="guige-yunfei">
                <text class="commodity-detail">月销</text>
                <text class="goods-info-specListString guige-specListString" :decode="true">{{ data.latelyMonthlySales }}&nbsp;件</text>
            </view>
        </view>
        <view class="commodity-detail-view">
            <view class="guige-yunfei">
                <text class="commodity-detail">运费</text>
                <text class="goods-info-specListString guige-specListString">免运费</text>
            </view>
        </view>

        <view class="commodity-detail-view">
            <view class="detail-title">
                <view>商品详情</view>
            </view>
            <view class="commodity-detail">
                <text>{{ data.detail ? data.detail : '暂无详情' }}</text>
            </view>
        </view>
        <!-- <navigator class="navigator-class" url="../../mine/share/index/index?inviterId={{userInfo.id}}">
   <view class="invite-wrapper">
      <image src="https://siam-hangzhou.oss-cn-hangzhou.aliyuncs.com/data/images/business/share-invite/share_mine.png"
         mode="aspectFill" class="invite-image"></image>
   </view>
</navigator> -->
        <view class="commdity-image-list">
            <image mode="widthFix" :src="item" class="detail-images" v-for="(item, index) in detailImages" :key="index"></image>
        </view>
        <text style="padding-bottom: 100rpx"></text>
        <view class="shopping-cart-detail">
            <mp-halfScreenDialog :show="shoppingCartDialog" extClass="extClassShoppingCart">
                <view slot="title">已选商品</view>
                <view slot="desc">
                    <scroll-view style="height: 55vh" scroll-y>
                        <view class="shoppingCart-item" v-for="(item, index) in shoppingCartList" :key="index">
                            <view class="goodsName-restructure-view">
                                <view class="goodsName out_of_range one_row">{{ item.goodsName }}</view>
                                <view class="restructure out_of_range one_row">{{ item.restructure }}</view>
                            </view>

                            <view class="goodsPrice-number-view">
                                <view class="goodsPrice">￥{{ item.price }}</view>
                                <view class="stepper">
                                    <text class="reduce-class" @tap="bindMinus" :data-cartId="item.id" :data-number="item.number">－</text>
                                    <input disabled type="number" :value="item.number" class="reduce-class" />
                                    <text @tap="bindPlus" class="add-class" :data-num="index + ',' + item.number">＋</text>
                                </view>
                            </view>
                        </view>
                        <!-- <view class="goodsName-packingCharges">
                  <text class="goodsName">包装费</text>
                  <text class="goodsPrice">￥{{packingCharges}}</text>
                  <text></text>
               </view> -->
                    </scroll-view>
                </view>
            </mp-halfScreenDialog>
            <mp-halfScreenDialog :show="specificationsDialog" extClass="extClassSpecifications">
                <view slot="title">选择规格</view>
                <view slot="desc">
                    <view class="goods-info-view">
                        <image :src="data.mainImage" mode="aspectFill" class="commodity-image"></image>
                        <view>
                            <view class="goods-info-name out_of_range one_row">{{ data.name }}</view>
                            <view class="goods-info-specListString">已选：{{ specListString }}</view>
                            <view class="goods-info-price">￥{{ priceAfter }}</view>
                        </view>
                    </view>
                    <scroll-view scroll-y style="height: 52vh">
                        <view class="commdity-name-type-view">
                            <!-- <view class="commdity-name">{{data.name}}</view> -->
                            <view class="commdity-type-item" v-for="(item, key) in specList" :key="key">
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
                            <is-show-tip :isShow="specList == null" :top="20" :bottom="20" :text="specList == null" tipText="暂无规格"></is-show-tip>
                        </view>
                    </scroll-view>
                    <view slot="footer">
                        <view class="good-choice-btn theme-bg" @tap="chooseSpecifications">我选好了</view>
                    </view>
                </view>
            </mp-halfScreenDialog>
        </view>
        <view class="shopping-detail-view">
            <view class="top_tips">
                <!-- <view>包装费：￥{{data.packingCharges}}</view> -->
                <view v-if="fullPriceReductionIsHidden">满减：{{ fullReductionRuleName }}</view>
            </view>
            <view class="buy-shopping-cart-view">
                <!-- <view class="cz-button">
            <view class="immediate-purchase theme-bg" bindtap="goToPay" hover-class="hover-class-public">
               立即购买(￥{{fullPriceReductionIsHidden?fullPriceReduction:(data.price)}})</view>
            <view class="add-to-cart theme-color-border" hover-class="hover-class-public" bindtap="insertShoppingCart">
               加入购物车
            </view>
         </view> -->
                <view class="cz-button">
                    <view class="immediate-purchase theme-bg" @tap="openSpecifications" data-type="pay" hover-class="hover-class-public">
                        立即购买(￥{{ fullPriceReductionIsHidden ? fullPriceReduction : data.price }})
                    </view>
                    <view class="add-to-cart theme-color-border" hover-class="hover-class-public" @tap="openSpecifications" data-type="car">加入购物车</view>
                </view>
            </view>
        </view>
    </view>
</template>

<script>
import mpHalfScreenDialog from '../../../../components/half-screen-dialog/half-screen-dialog';
import mpBadge from '../../../../components/badge/badge';
import isShowTip from '../../../../components/is-show-tip/is-show-tip';
import GlobalConfig from '../../../../utils/global-config';
import https from '../../../../utils/http';
import authService from '../../../../utils/auth';
import toastService from '../../../../utils/toast.service';
import utilHelper from '../../../../utils/util';
import dateHelper from '../../../../utils/date-helper';
//获取应用实例
const app = getApp();
var totalPrice = 0;
var totalNum = 0;
export default {
    components: {
        mpHalfScreenDialog,
        mpBadge,
        isShowTip
    },
    data() {
        return {
            carouselUrls: [],
            indicatorDots: true,
            autoplay: true,
            interval: 5000,
            duration: 1000,

            //beforeColor: "white",//指示点颜色,
            afterColor: 'white',

            //当前选中的指示点颜色
            commodityNum: 1,

            totalPrice: 0,
            shoppingCartDialog: false,
            specificationsDialog: false,
            userInfo: app.globalData.loginUserInfo,
            priceDifference: 0,
            shoppingCartList: [],
            chooseType: 'init',
            specList: '',
            specListString: '',
            priceAfter: '',
            fullPriceReduction: '',
            fullPriceReductionIsHidden: '',
            fullReductionRuleName: '',
            winHeight: '',
            fullReductionRuleList: '',

            data: {
                name: '',
                id: '',
                latelyMonthlySales: '',
                detail: false,
                mainImage: '',
                price: ''
            },

            detailImages: '',
            goodsId: '',
            repData: '',
            collectInfo: '',
            isCollect: ''
        };
    },
    /**
     * 生命周期函数--监听页面加载
     */
    onLoad: function (options) {
        this.autoHeigth();
        //console.log(app.globalData.loginUserInfo)
        this.setData({
            userInfo: app.globalData.loginUserInfo
        });
        this.getCommodityDetails(options.id);
        this.getGoodsCollect(options.id);
        // this.chooseSpecifications();
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
            let price = this.data.price;
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
            let totalPrice = price * this.commodityNum;
            let fullPriceReduction = 0;
            let fullReductionRuleName = '';
            let fullPriceReductionIsHidden = false;
            let limitedPrice = 0;
            for (let i = 0; i < this.fullReductionRuleList.length; i++) {
                //console.log(this.data.fullReductionRuleList[i].limitedPrice)
                if (totalPrice >= this.fullReductionRuleList[i].limitedPrice) {
                    if (this.fullReductionRuleList[i].limitedPrice > limitedPrice) {
                        limitedPrice = this.fullReductionRuleList[i].limitedPrice;
                        fullPriceReduction = totalPrice + 0 - this.fullReductionRuleList[i].reducedPrice;
                        fullReductionRuleName = this.fullReductionRuleList[i].name;
                        fullPriceReductionIsHidden = true;
                    }
                }
            }
            this.setData({
                specList: specList,
                specListString: specListString,
                priceAfter: price,
                totalPrice: utilHelper.toFixed(totalPrice, 2),
                fullPriceReduction: utilHelper.toFixed(fullPriceReduction, 2),
                fullPriceReductionIsHidden: fullPriceReductionIsHidden,
                fullReductionRuleName: fullReductionRuleName
            });
        },

        //事件处理函数
        /*点击减号*/
        bindMinus: function (e) {
            toastService.showLoading();
            var that = this;
            let cartId = e.currentTarget.dataset.cartid;
            let number = e.currentTarget.dataset.number;
            //点击减号后,当前商品的数量小于1,就进行删除该商品
            //重新计算被选中的商品的总金额
            totalPrice = 0;
            if (number == 1) {
                toastService.hideLoading();
                toastService.showModal(null, '确定不要这个了吗？', function confirm() {
                    toastService.showLoading();
                    that.updateNumber(cartId, 1, 0, function callback() {
                        totalNum--;
                        if (that.shoppingCartList.length == 1) {
                            that.setData({
                                shoppingCartDialog: false
                            });
                        }
                        that.getShoppingCartList();
                    });
                });
                return;
            }
            this.updateNumber(cartId, 1, 0, function callback() {
                totalNum--;
                that.getShoppingCartList();
            });
        },

        /*点击加号*/
        bindPlus: function (e) {
            toastService.showLoading();
            var that = this;
            let numList = e.currentTarget.dataset.num.split(',');
            let items = this.shoppingCartList;
            totalPrice = 0;
            items[numList[0]].number = Number(numList[1]) + 1; //当前商品的数量+1
            if (items[numList[0]].disable) {
                return;
            }
            this.updateNumber(items[numList[0]].id, 1, 1, function callback() {
                //重新计算被选中的商品的总金额
                totalNum++;
                that.getShoppingCartList();
            });
        },

        updateNumber(id, number, type, callbak) {
            https
                .request('/api-goods/rest/member/shoppingCart/updateNumber', {
                    id: id,
                    number: number,
                    type: type
                })
                .then((result) => {
                    if (result.success) {
                        callbak();
                    }
                });
        },

        insertShoppingCart(e) {
            // if(!goodsSpecs){
            //   toastService.showToast("请选择规格");
            //   return
            // }
            toastService.showLoading();
            authService.checkIsLogin().then((result) => {
                toastService.hideLoading();
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
                        goodsId: this.data.id,
                        specList: JSON.stringify(goodsSpecs)
                    };
                    https.request('/api-mall/rest/member/pointsMall/shoppingCart/insert', data, authService.getToken()).then((result) => {
                        if (result.success) {
                            toastService.showToast('加入购物车成功');
                            this.setData({
                                specificationsDialog: false
                            });
                        }
                    });
                    return;
                }
                app.globalData.checkIsAuth('scope.userInfo');
            });
        },

        goToPay() {
            let self = this;
            authService.checkIsLogin().then((result) => {
                if (result) {
                    self.toPay();
                    return;
                }
                app.globalData.checkIsAuth('scope.userInfo');
            });
        },

        toPay() {
            var data = this.data;
            var specList = this.specList;
            var orderDetail = {};
            var specLists = {};
            var restructure = '';
            //console.log(specList)
            if (data.status == 4) {
                toastService.showToast('商品已售罄');
                return;
            }
            //拼接数据
            for (let key in specList) {
                for (let keyof in specList[key]) {
                    //拼接查询规格等的json数据格式，查询商品规格等对应的价格
                    if (specList[key][keyof].checked) {
                        specLists[key] = specList[key][keyof].name;
                        restructure = (restructure ? restructure + '/' : restructure) + specList[key][keyof].name;
                    }
                }
            }
            //设置和在购物车提交时的数据格式一样
            orderDetail.actualPrice = this.totalPrice;
            orderDetail.packingCharges = 0;
            orderDetail.orderDetailList = [];
            orderDetail.orderDetailList.push({
                goodsId: data.id,
                specList: JSON.stringify(specLists),
                number: this.commodityNum,
                goodsName: data.name,
                restructure: restructure,
                price: this.priceAfter,
                packingCharges: 0
            });
            console.log(orderDetail);
            uni.navigateTo({
                url: '../pay/pay?orderDetail=' + JSON.stringify(orderDetail)
            });
        },

        autoHeigth() {
            var that = this;
            //获取用户手机系统信息
            uni.getSystemInfo({
                success: function (res) {
                    let winHeight = res.windowHeight; //获取高度
                    //获取class为settlement-view并减去这个高度，自适应scrollview的高度
                    const query = uni.createSelectorQuery();
                    query.select('.shopping-cart-detail').boundingClientRect();
                    query.selectViewport().scrollOffset();
                    query.exec(function (res) {
                        that.setData({
                            winHeight: winHeight - (res[0].height + 5)
                        });
                    });
                }
            });
        },

        closeShoppingCart: function () {
            this.setData({
                shoppingCartDialog: false
            });
        },

        openShoppingCart() {
            authService.checkIsLogin().then((result) => {
                if (result) {
                    if (this.shoppingCartList.length > 0) {
                        this.setData({
                            shoppingCartDialog: this.shoppingCartDialog ? false : true
                        });
                    }
                    return;
                }
                app.globalData.checkIsAuth('scope.userInfo');
            });
        },

        //获取满减规则
        getFullReductionRule(commodityDetails) {
            https
                .request('/api-mall/rest/pointsMall/fullReductionRule/list', {
                    pageNo: -1,
                    pageSize: 1
                })
                .then((result) => {
                    if (result.success) {
                        let fullPriceReduction = 0;
                        let fullReductionRuleName = '';
                        let fullPriceReductionIsHidden = false;
                        let limitedPrice = 0;
                        let totalPrice = commodityDetails.price + 0;
                        for (let i = 0; i < result.data.records.length; i++) {
                            if (totalPrice >= result.data.records[i].limitedPrice) {
                                if (result.data.records[i].limitedPrice > limitedPrice) {
                                    limitedPrice = result.data.records[i].limitedPrice;
                                    fullPriceReduction = totalPrice - result.data.records[i].reducedPrice;
                                    fullReductionRuleName = result.data.records[i].name;
                                    fullPriceReductionIsHidden = true;
                                }
                            }
                        }
                        this.setData({
                            fullPriceReduction: utilHelper.toFixed(fullPriceReduction, 2),
                            fullReductionRuleName: fullReductionRuleName,
                            fullPriceReductionIsHidden: fullPriceReductionIsHidden,
                            fullReductionRuleList: result.data.records,
                            totalPrice: commodityDetails.price
                        });
                    }
                });
        },

        getCommodityDetails(id) {
            https
                .request('/api-mall/rest/pointsMall/goods/selectById', {
                    id: id
                })
                .then((result) => {
                    if (result.success && result.data) {
                        //获取商品的详细图片，转换以轮播图的数据格式
                        let carouselUrls = result.data.subImages.split(',');
                        this.carouselUrls = [];
                        for (let i = 0; i < carouselUrls.length; i++) {
                            this.carouselUrls.push(GlobalConfig.ossUrl + carouselUrls[i]);
                        }
                        this.detailImages = [];
                        if (result.data.detailImages) {
                            let detailImages = result.data.detailImages.split(',');
                            for (let i = 0; i < detailImages.length; i++) {
                                this.detailImages.push(GlobalConfig.ossUrl + detailImages[i]);
                            }
                            result.data.mainImage = GlobalConfig.ossUrl + result.data.mainImage;
                        }
                        this.setData({
                            data: result.data,
                            carouselUrls: this.carouselUrls,
                            detailImages: this.detailImages,
                            priceAfter: result.data.price
                        });
                        this.selectByGoodsId(id);
                        this.getFullReductionRule(result.data);
                        this.chooseSpecifications();
                    }
                });
        },

        openSpecifications(e) {
            console.log(e);
            this.setData({
                specificationsDialog: true,
                goodsId: this.data.id,
                chooseType: e.currentTarget.dataset.type
            });
            this.selectByGoodsId(this.data.id);
        },

        selectByGoodsId(goodsId) {
            https
                .request('/api-mall/rest/pointsMall/goodsSpecificationOption/selectByGoodsId', {
                    goodsId: goodsId
                })
                .then((result) => {
                    if (result.success && result.data) {
                        // if(!result.data){
                        //   return;
                        // }
                        //重新设置商品的规格等数据的格式
                        //let goodsSpecs = {};
                        let specList = result.data;
                        let price = this.data.price;
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
                        this.setData({
                            specListString: specListString ? specListString : '暂无规格',
                            specList: JSON.stringify(specList) == '{}' ? null : specList,
                            priceAfter: price
                        });
                    }
                });
        },

        chooseSpecifications(e) {
            toastService.showLoading();
            var _this = this;
            authService.checkIsLogin().then((result) => {
                toastService.hideLoading();
                if (result) {
                    let goodsSpecs = {};
                    let specList = _this.specList;
                    console.log(_this);
                    let price = _this.data.price;
                    for (let key in specList) {
                        for (let keyof in specList[key]) {
                            //拼接查询规格等的json数据格式，查询商品规格等对应的价格
                            if (specList[key][keyof].checked) {
                                goodsSpecs[key] = specList[key][keyof].name;
                                //选中的规格价钱在商品价钱的基础上累加
                                price = price + specList[key][keyof].price;
                            }
                        }
                    }
                    //console.log(goodsSpecs)

                    _this.setData({
                        repData: JSON.stringify(goodsSpecs),
                        specificationsDialog: false
                    });
                    console.log(_this.chooseType);
                    if (this.chooseType == 'pay') {
                        _this.goToPay();
                    } else if (_this.chooseType == 'car') {
                        _this.insertShoppingCart();
                    } else {
                    }
                    toastService.hideLoading();
                    return;
                }
                this.setData({
                    specificationsDialog: false
                });
                app.globalData.checkIsAuth('scope.userInfo');
            });
        },

        insertStore() {
            authService.checkIsLogin().then((result) => {
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
                        goodsId: this.data.id,
                        specList: JSON.stringify(goodsSpecs),
                        number: this.commodityNum
                    };
                    https.request('/api-goods/rest/member/shoppingCart/insert', data, authService.getToken()).then((result) => {
                        if (result.success) {
                            toastService.showSuccess('加入成功');
                            //wx.navigateBack(1)
                        }
                    });
                    return;
                }
                app.globalData.checkIsAuth('scope.userInfo');
            });
        },

        businessTap(e) {
            uni.navigateTo({
                url: '../../menu/index/index?id=' + e.currentTarget.dataset.id
            });
        },

        getGoodsCollect(goodsId) {
            https
                .request('/api-mall/rest/member/pointsMall/goodsCollect/selectByGoodsId', {
                    goodsId: goodsId
                })
                .then((result) => {
                    if (result.success) {
                        this.setData({
                            collectInfo: result.data,
                            isCollect: result.data ? true : false
                        });
                    }
                });
        },

        bindInsertCollect(e) {
            authService.checkIsLogin().then((result) => {
                if (result) {
                    https
                        .request('/api-mall/rest/member/pointsMall/goodsCollect/insert', {
                            goodsId: e.currentTarget.dataset.goodsid
                        })
                        .then((result) => {
                            if (result.success) {
                                this.setData({
                                    isCollect: true
                                });
                                toastService.showToast('收藏成功');
                            }
                        });
                    return;
                }
                app.globalData.checkIsAuth('scope.userInfo');
            });
        },

        bindDeleteCollect(e) {
            authService.checkIsLogin().then((result) => {
                if (result) {
                    https
                        .request('/api-mall/rest/member/pointsMall/goodsCollect/delete', {
                            goodsId: e.currentTarget.dataset.goodsid
                        })
                        .then((result) => {
                            if (result.success) {
                                this.setData({
                                    isCollect: false
                                });
                                toastService.showToast('取消成功');
                            }
                        });
                    return;
                }
                app.globalData.checkIsAuth('scope.userInfo');
            });
        },

        bindShare(e) {
            authService.checkIsLogin().then((result) => {
                if (result) {
                    uni.navigateTo({
                        url: '../../../mine/share/index/index?inviterId=' + this.userInfo.id
                    });
                } else {
                    app.globalData.checkIsAuth('scope.userInfo');
                }
            });
        },

        uploadImage() {
            console.log('占位：函数 uploadImage 未声明');
        }
    }
};
</script>
<style>
@import './detail.css';
</style>
