<template>
    <view>
        <scroll-view :scroll-y="true">
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
                        <text class="name">{{ data.name }}</text>
                        <text :decode="true" class="price">￥{{ data.price }}</text>
                    </view>

                    <view class="collect-in">
                        <button class="go-to-shop theme-bg" @tap="businessTap" :data-id="data.shopId" size="mini" style="font-size: 24rpx">进店</button>
                        <view class="flex_column">
                            <text
                                :class="'iconfont iconshoucang1 ' + (isCollect ? 'is-collect' : '')"
                                @tap="parseEventDynamicCode($event, isCollect ? 'bindDeleteCollect' : 'bindInsertCollect')"
                                :data-goodsid="data.id"
                            ></text>
                            <text :class="'shoucang-text ' + (isCollect ? 'is-collect' : 'not-collect')">收藏</text>
                        </view>
                    </view>
                </view>

                <!-- <view class="commdity-engname">{{data.name}}</view>
		<view class="commdity-type-item" wx:for="{{specList}}" wx:key="index" wx:for-index='key'>
			<view class="commdity-type-name">{{key}}</view>
			<radio-group class="radio-group" bindchange="radioChange" data-firstIndex="{{key}}">
				<label wx:for="{{item}}" wx:key="index" class="group-label theme-color-border {{!item.stock?'disabled-group-label':''}} {{item.checked?'active theme-bg':''}} out_of_range one_row">
					<radio value="{{index}}" checked="{{item.checked}}" disabled="{{!item.stock}}" class="radio" /> {{item.name}}
				</label>
			</radio-group>
		</view> -->
            </view>
            <view class="commodity-detail-view">
                <view class="detail-title">
                    <view>商品详情</view>
                    <view v-if="!shopInfo.isOutofDeliveryRange && shopInfo.isOperatingOfShop && shopInfo.shop.isOperating">
                        <view class="stepper" v-if="data.isShopCart">
                            <block>
                                <view class="flex_center car_reduce_add reduce-class" @tap="bindMinus" :data-cartId="data.cartId" :data-number="data.number">－</view>
                                <input disabled type="number" :value="data.number" class="radd-reduce-input" />
                                <view
                                    class="flex_center car_reduce_add add-class"
                                    :data-goodsId="data.id"
                                    @tap="parseEventDynamicCode($event, data.goodsStatus != 4 ? 'openSpecifications' : '')"
                                >
                                    ＋
                                </view>
                            </block>
                        </view>
                        <block v-if="!data.isShopCart">
                            <view class="insert-view theme-bg" :data-goodsId="data.id" @tap="parseEventDynamicCode($event, data.goodsStatus != 4 ? 'openSpecifications' : '')">
                                ＋
                            </view>
                        </block>
                    </view>
                </view>
                <view class="commodity-detail">
                    {{ data.detail }}
                </view>
            </view>
            <navigator class="navigator-class" :url="'../../mine/share/index/index?inviterId=' + userInfo.id">
                <view class="invite-wrapper">
                    <image
                        :src="'https://siam-hangzhou.oss-cn-hangzhou.aliyuncs.com/data/images/business/share-invite/share_mine.png?v=' + timestamp"
                        mode="aspectFill"
                        class="invite-image"
                    ></image>
                </view>
            </navigator>
        </scroll-view>
        <view class="shopping-cart-detail">
            <view class="content-fullReductionRuleName">
                {{
                    !shopInfo.isOutofDeliveryRange
                        ? shopInfo.isOperatingOfShop && shopInfo.shop.isOperating
                            ? fullPriceReductionIsHidden
                                ? fullReductionRuleName
                                : '暂无优惠'
                            : '商家已休息，暂不接单'
                        : '超出配送范围'
                }}
            </view>
            <view :class="'shopping-cart-content ' + (shopInfo.isOutofDeliveryRange || !shopInfo.isOperatingOfShop || !shopInfo.shop.isOperating ? 'isOutofDeliveryRange' : '')">
                <view class="shopping-cart-left" @tap="parseEventDynamicCode($event, shopInfo.isOutofDeliveryRange || !shopInfo.isOperatingOfShop ? '' : 'openShoppingCart')">
                    <view>
                        <view :class="'iconfont icongouwuche1 highlight ' + (shoppingCartList.length <= 0 ? 'theme-other-bg' : 'theme-bg')">
                            <view class="num" v-if="totalNum > 0">{{ totalNum }}</view>
                        </view>
                    </view>
                    <view>
                        <view class="shopping-cart-totalPrice">
                            <view :class="fullPriceReductionIsHidden ? 'fullPriceReductionClass' : 'totalPrice'" v-if="shoppingCartList.length > 0">￥{{ totalPrice }}</view>
                            <view class="full-price-reduction" v-if="fullPriceReductionIsHidden && shoppingCartList.length > 0">￥{{ fullPriceReduction }}</view>
                            <view class="not-full-price-reduction" v-if="shoppingCartList.length <= 0">暂未选购商品</view>
                        </view>
                        <!-- <view class="shopping-cart-desc">另需配送费￥{{deliveryPrice}}元</view> -->
                    </view>
                </view>
                <view
                    :class="(isStartDeliveryPrice ? 'theme-bg' : 'shopping-cart-bg') + ' shopping-cart-right'"
                    @tap="
                        parseEventDynamicCode(
                            $event,
                            shoppingCartList.length <= 0 || !isStartDeliveryPrice || shopInfo.isOutofDeliveryRange || !shopInfo.isOperatingOfShop ? '' : 'goToPay'
                        )
                    "
                >
                    {{ isStartDeliveryPrice ? '去结算' : '差 ￥' + priceDifference + ' 起送' }}
                </view>
            </view>
            <mp-halfScreenDialog :show="shoppingCartDialog" extClass="extClassShoppingCart" @close="closeShoppingCart">
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
                                    <view class="flex_center car_reduce_add reduce-class" @tap="bindMinus" :data-cartId="item.id" :data-number="item.number">－</view>
                                    <input disabled type="number" :value="item.number" class="add-reduce-input" />
                                    <view @tap="bindPlus" class="flex_center car_reduce_add add-class" :data-num="index + ',' + item.number">＋</view>
                                </view>
                            </view>
                        </view>
                        <view class="goodsName-packingCharges">
                            <text class="goodsName">包装费</text>
                            <text class="goodsPrice">￥{{ packingCharges }}</text>
                            <text></text>
                        </view>
                    </scroll-view>
                </view>
            </mp-halfScreenDialog>
            <mp-halfScreenDialog :show="specificationsDialog" extClass="extClassSpecifications">
                <view slot="title">选择规格</view>
                <view slot="desc">
                    <view class="goods-info-view">
                        <image :src="data.mainImage" mode="aspectFill" class="commodity-image"></image>
                        <view>
                            <view class="goods-info-name">{{ data.name }}</view>
                            <view class="goods-info-specListString">已选：{{ specListString }}</view>
                            <view class="goods-info-price">￥{{ priceAfter }}</view>
                        </view>
                    </view>
                    <scroll-view scroll-y style="height: 56vh">
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
                        </view>
                    </scroll-view>
                    <view slot="footer">
                        <view class="good-choice-btn theme-bg" @tap="insertShoppingCart">我选好了</view>
                    </view>
                </view>
            </mp-halfScreenDialog>
        </view>
    </view>
</template>

<script>
import mpHalfScreenDialog from '../../../components/half-screen-dialog/half-screen-dialog';
import GlobalConfig from '../../../utils/global-config';
import https from '../../../utils/http';
import authService from '../../../utils/auth';
import toastService from '../../../utils/toast.service';
import utilHelper from '../../../utils/util';
import dateHelper from '../../../utils/date-helper';
//获取应用实例
const app = getApp();
var totalPrice = 0;
var totalNum = 0;
export default {
    components: {
        mpHalfScreenDialog
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
            shopId: '',
            timestamp: '',
            specList: '',
            priceAfter: '',
            fullPriceReduction: '',
            fullPriceReductionIsHidden: '',
            fullReductionRuleName: '',
            winHeight: '',
            goodsId: '',

            shopInfo: {
                isOutofDeliveryRange: '',
                isOperatingOfShop: '',

                shop: {
                    isOperating: ''
                }
            },

            isItNear: '',

            data: {
                name: '',
                price: '',
                shopId: '',
                id: '',
                isShopCart: '',
                cartId: '',
                number: '',
                detail: '',
                mainImage: ''
            },

            totalNum: '',
            isStartDeliveryPrice: '',
            packingCharges: '',
            fullReductionRuleList: '',
            specListString: '',
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
            shopId: options.shopId,
            userInfo: app.globalData.loginUserInfo
        });
        this.getCommodityDetails(options.id);
        this.getGoodsCollect(options.id);
    },
    /**
     * 生命周期函数--监听页面初次渲染完成
     */
    onReady: function () {},
    /**
     * 生命周期函数--监听页面显示
     */
    onShow: function () {
        this.getShopInfo();
        this.getTimestamp();
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
        getTimestamp() {
            var timestamp = dateHelper.getTimestamp();
            console.log(timestamp);
            this.setData({
                timestamp: timestamp
            });
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
            let price = this.data.price;
            for (let key in specList) {
                for (let keyof in specList[key]) {
                    //console.log(specList[key][keyof].price)
                    if (specList[key][keyof].checked) {
                        price = price + specList[key][keyof].price;
                    }
                }
            }
            // let goodsSpecs = {};
            // for(let i=0;i<specList.length;i++){
            //   for (let j = 0; j < specList[i].length;j++){
            //     //拼接查询规格等的json数据格式，查询商品规格等对应的价格
            //     if (specList[i][j].checked) {
            //       goodsSpecs[specList[i].spec] = specList[i][j].name;
            //     }
            //   }
            // }
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
                        fullPriceReduction = totalPrice + this.data.packingCharges - this.fullReductionRuleList[i].reducedPrice;
                        fullReductionRuleName = this.fullReductionRuleList[i].name;
                        fullPriceReductionIsHidden = true;
                    }
                }
            }
            this.setData({
                specList: specList,
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
                        goodsId: this.goodsId,
                        specList: JSON.stringify(goodsSpecs),
                        shopId: this.shopInfo.shop.id
                    };
                    https.request('/api-goods/rest/member/shoppingCart/insert', data, authService.getToken()).then((result) => {
                        if (result.success) {
                            this.setData({
                                specificationsDialog: false
                            });
                            this.getShoppingCartList();
                            //wx.navigateBack(1)
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
                    //判断店铺是否打烊
                    let startTime = this.shopInfo.shop.startTime;
                    let endTime = this.shopInfo.shop.endTime;
                    let isOperating = this.shopInfo.shop.isOperating;
                    app.globalData.getIsBusiness(startTime, endTime, isOperating).then((result) => {
                        if (!result) {
                            return;
                        }
                        self.toPay();
                    });
                    return;
                }
                app.globalData.checkIsAuth('scope.userInfo');
            });
        },

        toPay() {
            var list = this.shoppingCartList;
            var orderDetail = {};
            orderDetail.actualPrice = this.totalPrice;
            orderDetail.shopId = this.shopId;
            orderDetail.orderDetailList = [];
            for (var key in list) {
                orderDetail.orderDetailList.push({
                    goodsId: list[key].goodsId,
                    specList: list[key].specList,
                    number: list[key].number,
                    goodsName: list[key].goodsName,
                    restructure: list[key].restructure,
                    price: list[key].price,
                    id: list[key].id,
                    packingCharges: list[key].packingCharges
                });
            }
            uni.navigateTo({
                url: '../pay/pay?orderDetail=' + JSON.stringify(orderDetail) + '&payType=store'
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

        openSpecifications(e) {
            this.setData({
                specificationsDialog: true,
                goodsId: e.currentTarget.dataset.goodsid
            });
            this.getCommodityDetails(e.currentTarget.dataset.goodsid);
        },

        getShopInfo() {
            https
                .request('/api-merchant/rest/shop/detail', {
                    id: this.shopId,
                    position: app.globalData.deliveryAndSelfTaking.location
                })
                .then((result) => {
                    if (result.success && result.data) {
                        this.getFullReductionRule(result.data);
                    }
                });
        },

        //获取满减规则
        getFullReductionRule(shopInfo) {
            console.log(shopInfo);
            let fullPriceReduction = 0;
            let fullReductionRuleName = '';
            let fullPriceReductionIsHidden = false;
            let limitedPrice = 0;
            for (let i = 0; i < shopInfo.fullReductionRuleList.length; i++) {
                if (totalPrice + this.packingCharges >= shopInfo.fullReductionRuleList[i].limitedPrice) {
                    if (shopInfo.fullReductionRuleList[i].limitedPrice > limitedPrice) {
                        limitedPrice = shopInfo.fullReductionRuleList[i].limitedPrice;
                        fullPriceReduction = totalPrice - shopInfo.fullReductionRuleList[i].reducedPrice;
                        fullReductionRuleName = shopInfo.fullReductionRuleList[i].name;
                        fullPriceReductionIsHidden = true;
                    }
                }
            }
            var shopWithinImgs = shopInfo.shop.shopWithinImg.split(',');
            for (let i = 0; i < shopWithinImgs.length; i++) {
                shopWithinImgs[i] = GlobalConfig.ossUrl + shopWithinImgs[i];
            }
            shopInfo.shop.shopLogoImg = GlobalConfig.ossUrl + shopInfo.shop.shopLogoImg;
            shopInfo.shop.shopWithinImgs = shopWithinImgs;
            //shopInfo.shop.isOperating = false;
            //shopInfo.shop.isOutofDeliveryRange = true;
            app.globalData.getIsItNear(shopInfo.shop.startTime, shopInfo.shop.endTime);
            //isOutofDeliveryRange true为超出范围
            //isOperatingOfShop false为打烊
            this.setData({
                shopInfo: shopInfo,
                fullPriceReduction: utilHelper.toFixed(fullPriceReduction, 2),
                fullReductionRuleName: fullReductionRuleName,
                fullPriceReductionIsHidden: fullPriceReductionIsHidden,
                isItNear: app.globalData.deliveryAndSelfTaking.isItNear
            });
            this.getShoppingCartList();
        },

        getShoppingCartList() {
            https
                .request('/api-goods/rest/member/shoppingCart/list', {
                    shopId: this.shopId,
                    pageNo: -1,
                    pageSize: 20
                })
                .then((result) => {
                    toastService.hideLoading();
                    if (result.success && result.data) {
                        var packingCharges = 0;
                        var totalNum = 0;
                        var totalPrice = 0;
                        var number = 0;
                        this.data.isShopCart = false;
                        result.data.records.forEach((result, index) => {
                            let specList = '';
                            for (var key in JSON.parse(result.specList)) {
                                specList = (specList ? specList + '/' : specList) + JSON.parse(result.specList)[key];
                            }
                            result.restructure = specList;
                            totalNum = totalNum + result.number;
                            result.goodsPrices = utilHelper.toFixed(Number(result.goodsPrice) * result.number, 2);
                            totalPrice += result.price * result.number; //初始化被选中的商品的总金额
                            result.disable = result.goodsStatus == 1 || result.goodsStatus == 3 || result.goodsStatus == 4 ? true : false;
                            packingCharges =
                                result.goodsStatus == 1 || result.goodsStatus == 3 || result.goodsStatus == 4
                                    ? packingCharges
                                    : packingCharges + result.packingCharges * result.number;
                            if (this.data.id == result.goodsId) {
                                number = number + result.number;
                                this.data.isShopCart = true;
                                this.data.cartId = result.id;
                                this.data.number = number;
                            }
                        });
                        totalPrice = utilHelper.toFixed(totalPrice, 2);
                        var isStartDeliveryPrice = false;
                        var priceDifference = 0;
                        this.shopInfo.shop.startDeliveryPrice = this.shopInfo.shop.startDeliveryPrice ? this.shopInfo.shop.startDeliveryPrice : 0;
                        if (totalPrice + packingCharges >= this.shopInfo.shop.startDeliveryPrice) {
                            isStartDeliveryPrice = true;
                        }
                        console.log('获取当前店铺的起送费====', this.shopInfo.shop.startDeliveryPrice);
                        priceDifference = this.shopInfo.shop.startDeliveryPrice - (totalPrice + packingCharges);
                        this.setData({
                            data: this.data,
                            totalNum: totalNum,
                            priceDifference: utilHelper.toFixed(priceDifference, 2),
                            isStartDeliveryPrice: isStartDeliveryPrice,
                            shoppingCartList: result.data.records,
                            packingCharges: packingCharges,
                            totalPrice: utilHelper.toFixed(totalPrice + packingCharges, 2)
                        });
                        this.getShopCartFullReductionRule();
                    }
                });
        },

        //获取满减规则
        getShopCartFullReductionRule() {
            https
                .request('/api-promotion/rest/fullReductionRule/list', {
                    pageNo: -1,
                    pageSize: 1,
                    shopId: this.shopId
                })
                .then((result) => {
                    if (result.success) {
                        let fullPriceReduction = 0;
                        let fullReductionRuleName = '';
                        let fullPriceReductionIsHidden = false;
                        let limitedPrice = 0;
                        for (let i = 0; i < result.data.records.length; i++) {
                            if (this.totalPrice >= result.data.records[i].limitedPrice) {
                                if (result.data.records[i].limitedPrice > limitedPrice) {
                                    limitedPrice = result.data.records[i].limitedPrice;
                                    fullPriceReduction = this.totalPrice - result.data.records[i].reducedPrice;
                                    fullReductionRuleName = result.data.records[i].name;
                                    fullPriceReductionIsHidden = true;
                                }
                            }
                        }
                        this.setData({
                            fullPriceReduction: utilHelper.toFixed(fullPriceReduction, 2),
                            fullReductionRuleName: fullReductionRuleName,
                            fullPriceReductionIsHidden: fullPriceReductionIsHidden,
                            fullReductionRuleList: result.data.records
                        });
                    }
                });
        },

        getCommodityDetails(id) {
            https
                .request('/api-goods/rest/goods/selectById', {
                    id: id,
                    position: app.globalData.deliveryAndSelfTaking.location
                })
                .then((result) => {
                    if (result.success && result.data) {
                        //获取商品的详细图片，转换以轮播图的数据格式
                        let carouselUrls = result.data.subImages.split(',');
                        for (let i = 0; i < carouselUrls.length; i++) {
                            this.carouselUrls.push(GlobalConfig.ossUrl + carouselUrls[i]);
                        }
                        result.data.mainImage = GlobalConfig.ossUrl + result.data.mainImage;
                        this.setData({
                            data: result.data,
                            carouselUrls: this.carouselUrls,
                            priceAfter: result.data.price
                        });
                        this.selectByGoodsId(id);
                    }
                });
        },

        selectByGoodsId(goodsId) {
            https
                .request('/api-goods/rest/goodsSpecificationOption/selectByGoodsId', {
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
                            specListString: specListString,
                            specList: specList
                        });
                    }
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
                .request('/api-goods/rest/member/goodsCollect/selectByGoodsId', {
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
            https
                .request('/api-goods/rest/member/goodsCollect/insert', {
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
        },

        bindDeleteCollect(e) {
            https
                .request('/api-goods/rest/member/goodsCollect/delete', {
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
