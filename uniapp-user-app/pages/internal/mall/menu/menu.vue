<template>
    <view>
        <view v-if="menuList.length > 0" class="mp-vtabs-view">
            <mp-vtabs
                :vtabs="menuList"
                :activeTab="activeTab"
                tabBarClass="mp-vtabs-class"
                @tabclick="onTabCLick"
                @change="onChange"
                tabBarLineColor="#004ca0"
                tabBarActiveTextColor="#004ca0"
                :contentHeight="contentHeight"
            >
                <block v-for="(menu, menuIndex) in menuList" :key="menuIndex">
                    <mp-vtabs-content :tabIndex="menuIndex">
                        <view :class="'vtabs-content-item ' + (menuList.length - 1 == menuIndex ? 'is-end-item' : '')">
                            <view class="commodity-type position-sticky">
                                <view class="categoryName-view out_of_range one_row">{{ menu.name }}</view>
                            </view>
                            <block v-for="(chil, goodsIndex) in menu.goodsList" :key="goodsIndex">
                                <view :class="'commodity-item ' + (chil.goodsStatus == 4 ? 'isEnd' : '')" :hover-class="chil.goodsStatus == 4 ? '' : 'hover-class-public'">
                                    <view
                                        class="commodity-name-view"
                                        @tap="parseEventDynamicCode($event, chil.goodsStatus == 4 ? '' : 'commodityDetailTap')"
                                        :data-id="chil.goodsId"
                                    >
                                        <image :src="chil.mainImage ? chil.mainImage : '../../assets/images/load-image.png'" mode="aspectFill" class="commodity-image"></image>
                                        <view class="sell-out" v-if="chil.goodsStatus == 4">售罄</view>
                                        <view class="commodity-name-english-view">
                                            <view class="commodity-name out_of_range two_row">{{ chil.goodsName }}</view>
                                            <view class="commodity-english"><text></text></view>
                                            <view class="money-view">￥{{ chil.goodsPrice }}</view>
                                        </view>
                                    </view>
                                </view>

                                <view class="money-insert-view">
                                    <!-- <view class="insert-view theme-bg">＋</view> -->
                                    <view class="stepper" v-if="chil.number > 0">
                                        <block>
                                            <text class="reduce-class" @tap="bindMinus" :data-cartId="chil.cartId" :data-number="chil.number">－</text>
                                            <input disabled type="number" :value="chil.number" class="reduce-class" />
                                            <text
                                                class="add-class"
                                                :data-goodsId="chil.goodsId"
                                                @tap.stop.prevent="parseEventDynamicCode($event, chil.goodsStatus != 4 ? 'openSpecifications' : '')"
                                            >
                                                ＋
                                            </text>
                                        </block>
                                    </view>
                                    <block v-else>
                                        <view
                                            :class="'insert-view theme-bg ' + (chil.goodsStatus == 4 ? 'isEnd' : '')"
                                            :data-goodsId="chil.goodsId"
                                            @tap="parseEventDynamicCode($event, chil.goodsStatus != 4 ? 'openSpecifications' : '')"
                                        >
                                            ＋
                                        </view>
                                    </block>
                                </view>
                            </block>
                        </view>
                    </mp-vtabs-content>
                </block>
            </mp-vtabs>
        </view>
        <mp-halfScreenDialog :show="specificationsDialog" extClass="extClassSpecifications">
            <view slot="title">选择规格</view>
            <view slot="desc">
                <view class="goods-info-view">
                    <image :src="goodsInfo.mainImage" mode="aspectFill" class="commodity-image"></image>
                    <view>
                        <view class="goods-info-name out_of_range one_row">{{ goodsInfo.name }}</view>
                        <view class="goods-info-specListString">已选：{{ specListString }}</view>
                        <view class="goods-info-price">￥{{ priceAfter }}</view>
                    </view>
                </view>
                <scroll-view scroll-y style="height: 56vh">
                    <view class="commdity-name-type-view">
                        <view class="commdity-name">{{ data.name }}</view>
                        <view class="commdity-engname">{{ data.name }}</view>
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
                    <view class="good-choice-btn theme-bg" @tap="insertShoppingCart">我选好了</view>
                </view>
            </view>
        </mp-halfScreenDialog>
    </view>
</template>

<script>
import mpVtabs from '../../../../components/vtabs/index';
import mpVtabsContent from '../../../../components/vtabs-content/index';
import mpHalfScreenDialog from '../../../../components/half-screen-dialog/half-screen-dialog';
import isShowTip from '../../../../components/is-show-tip/is-show-tip';
import GlobalConfig from '../../../../utils/global-config';
import https from '../../../../utils/http';
import toastService from '../../../../utils/toast.service';
import authService from '../../../../utils/auth';
import utilHelper from '../../../../utils/util';
import CustomPage from '../../../../base/CustomPage';
import dateHelper from '../../../../utils/date-helper';
var menuList = [];
var app = getApp();
export default {
    components: {
        mpVtabs,
        mpVtabsContent,
        mpHalfScreenDialog,
        isShowTip
    },
    data() {
        return {
            specificationsDialog: false,
            contentHeight: '',
            menuList: '',
            goodsId: '',

            goodsInfo: {
                mainImage: '',
                name: ''
            },

            priceAfter: '',
            specListString: '',
            specList: '',
            shoppingCartDialog: false,
            activeTab: '',
            menuIndex: 0,

            menu: {
                name: '',
                goodsList: []
            },

            goodsIndex: 0,

            chil: {
                goodsStatus: 0,
                goodsId: '',
                mainImage: false,
                goodsName: '',
                goodsPrice: '',
                number: 0,
                cartId: ''
            },

            data: {
                name: ''
            }
        };
    },
    /**
     * 生命周期函数--监听页面加载
     */
    onLoad: function (options) {
        this.getMenuList();
        this.topViewHeight();
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
        topViewHeight() {
            let windowHeight = app.globalData.systemInfoSync.windowHeight;
            let screenTop = app.globalData.systemInfoSync.screenTop;
            this.setData({
                contentHeight: windowHeight + screenTop / 2 - 5
            });
            console.log(app.globalData.systemInfoSync);
        },

        getMenuList() {
            toastService.showLoading();
            var self = this;
            https.request('/api-mall/rest/pointsMall/menu/listWithGoods', {}).then((result) => {
                toastService.hideLoading();
                if (result.success && result.data) {
                    //console.log(result.data)
                    result.data.forEach((aitem, index) => {
                        aitem.goodsList.forEach((bitem) => {
                            bitem.isShopCart = false;
                            bitem.mainImage = GlobalConfig.ossUrl + bitem.mainImage;
                        });
                    });
                    menuList = result.data;
                    this.setData({
                        menuList: menuList
                    });
                }
            });
        },

        openSpecifications(e) {
            toastService.showLoading();
            this.setData({
                specificationsDialog: true,
                goodsId: e.currentTarget.dataset.goodsid
            });
            this.getCommodityDetails(e.currentTarget.dataset.goodsid);
        },

        getCommodityDetails(id) {
            https
                .request('/api-mall/rest/pointsMall/goods/selectById', {
                    id: id,
                    pageNo: -1,
                    pageSize: 20
                })
                .then((result) => {
                    toastService.hideLoading();
                    if (result.success && result.data) {
                        //获取商品的详细图片，转换以轮播图的数据格式
                        //console.log(result.data)
                        result.data.mainImage = GlobalConfig.ossUrl + result.data.mainImage;
                        this.setData({
                            goodsInfo: result.data,
                            priceAfter: result.data.price
                        });
                        this.selectByGoodsId(id);
                    }
                });
        },

        commodityDetailTap(e) {
            console.log(e.currentTarget.dataset.id);
            uni.navigateTo({
                url: '../detail/detail?id=' + e.currentTarget.dataset.id
            });
        },

        selectByGoodsId(goodsId) {
            console.log(this);
            https
                .request('/api-mall/rest/pointsMall/goodsSpecificationOption/selectByGoodsId', {
                    goodsId: goodsId
                })
                .then((result) => {
                    if (result.success && result.data) {
                        //重新设置商品的规格等数据的格式
                        //let goodsSpecs = {};
                        let specList = result.data;
                        let price = 0;
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
                            specList: JSON.stringify(specList) == '{}' ? null : specList
                        });
                    }
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
            let price = this.goodsInfo.price;
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
            // let totalPrice = price * this.data.commodityNum;
            // let fullPriceReduction = 0,
            //    fullReductionRuleName = "",
            //    fullPriceReductionIsHidden = false,
            //    limitedPrice = 0;
            // //console.log(this.data.fullReductionRuleList)
            // for (let i = 0; i < this.data.shopInfo.fullReductionRuleList.length; i++) {
            //    //console.log(this.data.fullReductionRuleList[i].limitedPrice)
            //    if (totalPrice >= this.data.shopInfo.fullReductionRuleList[i].limitedPrice) {
            //       if (this.data.shopInfo.fullReductionRuleList[i].limitedPrice > limitedPrice) {
            //          limitedPrice = this.data.shopInfo.fullReductionRuleList[i].limitedPrice;
            //          fullPriceReduction = (totalPrice + this.data.data.packingCharges) - this.data.shopInfo.fullReductionRuleList[i].reducedPrice;
            //          fullReductionRuleName = this.data.shopInfo.fullReductionRuleList[i].name;
            //          fullPriceReductionIsHidden = true;
            //       }
            //    }
            // }

            // for (var key in specList) {
            //   console.log(specList[key])
            //   specListString = (specListString ? specListString + "/" : specListString) + specList[key];
            // }
            //console.log(specList)
            this.setData({
                specList: specList,
                specListString: specListString,
                priceAfter: price
                // totalPrice: utilHelper.toFixed(totalPrice, 2),
                // fullPriceReduction: utilHelper.toFixed(fullPriceReduction, 2),
                // fullPriceReductionIsHidden: fullPriceReductionIsHidden,
                // fullReductionRuleName: fullReductionRuleName
            });
        },

        insertShoppingCart(e) {
            // if(!goodsSpecs){
            //   toastService.showToast("请选择规格");
            //   return
            // }
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
                        goodsId: this.goodsInfo.id,
                        specList: JSON.stringify(goodsSpecs)
                    };
                    https.request('/api-mall/rest/member/pointsMall/shoppingCart/insert', data).then((result) => {
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

        onTabCLick() {
            console.log('占位：函数 onTabCLick 未声明');
        },

        onChange() {
            console.log('占位：函数 onChange 未声明');
        }
    }
};
</script>
<style>
@import './menu.css';
</style>
