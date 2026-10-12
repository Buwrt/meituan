<template>
    <view>
        <!-- <view wx:if="{{!netWorkType}}">
	<image src="https://siam-hangzhou.oss-cn-hangzhou.aliyuncs.com/data/images/business/background.jpg" class="top-bg-class" mode="widthFix"></image>
</view> -->
        <view class="content-class" v-if="!netWorkType">
            <view id="page-top-view" class="public-views">
                <image :src="shopInfo.shop.firstPoster" class="slide-image" mode="aspectFill" />
                <view class="business-view">
                    <image :src="shopInfo.shop.shopLogoImg" class="business-image" mode="aspectFill"></image>
                </view>
                <view class="business-info">
                    <view class="business-info-name" @tap="isShopDetailTap">{{ shopInfo.shop.name }}</view>
                    <view class="business-info-other">
                        <text :decode="true">评价&nbsp;&nbsp;{{ shopInfo.shop.serviceRating }}</text>
                        <!-- <text decode="true">&nbsp;&nbsp;&nbsp;月售874单</text>
				<text decode="true">&nbsp;&nbsp;&nbsp;蜂鸟转送约40分钟</text> -->
                    </view>
                    <view @tap="isPromotionTap">
                        <view class="business-discount-info business-info-flex" v-if="shopInfo.promotionList.length > 0">
                            <view class="business-discount out_of_range one_row">
                                <view class="theme-color-border business-discount-list" v-if="index < 4" v-for="(rule, index) in shopInfo.promotionList" :key="index">
                                    {{ rule.name }}
                                </view>
                            </view>
                            <view class="other-promotionList">
                                <text v-if="shopInfo.promotionList.length > 4" :decode="true">等{{ shopInfo.promotionList.length }}个优惠&nbsp;</text>
                                <text class="iconfont iconweibiaoti35-copy"></text>
                            </view>
                        </view>
                        <view v-else-if="shopInfo.shop.reducedDeliveryPrice" class="business-discount-list theme-color">配送费立减{{ shopInfo.shop.reducedDeliveryPrice }}元</view>
                    </view>
                    <view class="notice-view out_of_range one_row">{{ shopInfo.shop.announcement }}</view>
                </view>
            </view>

            <!-- <view class="swiper-tabs-choice self-adaption" style="{{scrollTop?'':'display:none'}}">
		<view class="swiper-tabs-choice-item" wx:for="{{businessModes}}" wx:key="index" data-current="{{index}}" bindtap="clickTab" hover-class='hover-click-class'>
			<view class="swiper_table_item_view {{currentTab==item.modeId?'active':''}}" data-current="{{index}}" bindtap="clickTab">
				{{item.modeName}}
			</view>
		</view>
	</view> -->
            <view id="d_p_s" class="self-adaption position-sticky" style="position: sticky; top: -1; z-index: 1">
                <view class="swiper-tabs-choice" style="position: sticky; top: 0rpx">
                    <view class="swiper-tabs-choice-item" :data-current="index" @tap="clickTab" hover-class="hover-click-class" v-for="(item, index) in businessModes" :key="index">
                        <view :class="'swiper_table_item_view ' + (currentTab == item.modeId ? 'active' : '')" :data-current="index" @tap="clickTab">
                            {{ item.modeName }}
                        </view>
                    </view>
                </view>
            </view>
            <view id="businessRecommend" class="business-recommend-view">
                <view v-if="merchantRecommendGoods.length > 0 && currentTab == 0">
                    <view class="business-recommend-title">商家推荐</view>
                    <view class="business-recommend-scroll-view">
                        <scroll-view :scroll-x="true">
                            <view class="business-recommend-items" hover-class="hover-view-class">
                                <view
                                    :class="'business-recommend-item ' + (goodsItem.goodsStatus == 4 ? 'isEnd' : '')"
                                    v-for="(goodsItem, index) in merchantRecommendGoods"
                                    :key="index"
                                >
                                    <view class="recommend-sell-out" v-if="goodsItem.goodsStatus == 4">售罄</view>

                                    <view
                                        class="business-recommend-detail-view"
                                        @tap="parseEventDynamicCode($event, goodsItem.goodsStatus == 4 ? '' : 'commodityDetailTap')"
                                        :data-id="goodsItem.goodsId"
                                    >
                                        <image :src="goodsItem.mainImage" mode="aspectFill" class="icon-business-recommend-class"></image>
                                    </view>

                                    <view class="business-recommend-money-view">
                                        <!-- <view class="business-recommend-money" bindtap="commodityDetailTap" data-id="{{item.id}}">
									<text>￥</text>
									<text>{{item.goodsPrice}}</text>
								</view>
								<view class="plus-view theme-bg" data-goodsId="{{item.goodsId}}" bindtap="{{item.goodsStatus!=4?'openSpecifications':''}}">+</view> -->
                                        <view class="fullname-class out_of_range one_row">{{ goodsItem.goodsName }}</view>
                                    </view>

                                    <view class="business-recommend-money-view">
                                        <view class="fullname-class out_of_range one_row">月售：{{ goodsItem.latelyMonthlySales }}</view>
                                        <text></text>
                                    </view>

                                    <view class="business-recommend-money-view fullname-stepper">
                                        <view class="fullname-class out_of_range one_row">￥{{ goodsItem.goodsPrice }}</view>
                                        <view class="stepper" v-if="goodsItem.number > 0">
                                            <block v-if="!shopInfo.isOutofDeliveryRange && shopInfo.isOperatingOfShop && shopInfo.shop.isOperating">
                                                <text
                                                    class="flex_center car_reduce_add reduce-class"
                                                    @tap="bindMinus"
                                                    :data-cartId="goodsItem.cartId"
                                                    :data-number="goodsItem.number"
                                                >
                                                    －
                                                </text>
                                                <input disabled type="number" :value="goodsItem.number" class="radd-reduce-input" />
                                                <text
                                                    @tap="parseEventDynamicCode($event, goodsItem.goodsStatus != 4 ? 'openSpecifications' : '')"
                                                    class="flex_center car_reduce_add add-class"
                                                    :data-goodsId="goodsItem.goodsId"
                                                >
                                                    ＋
                                                </text>
                                            </block>
                                        </view>
                                        <block v-if="goodsItem.number <= 0 && !shopInfo.isOutofDeliveryRange && shopInfo.isOperatingOfShop && shopInfo.shop.isOperating">
                                            <view
                                                class="insert-view shangjiatuijianjia theme-bg"
                                                :data-goodsId="goodsItem.goodsId"
                                                @tap="parseEventDynamicCode($event, goodsItem.goodsStatus != 4 ? 'openSpecifications' : '')"
                                            >
                                                ＋
                                            </view>
                                        </block>
                                    </view>
                                </view>
                            </view>
                        </scroll-view>
                    </view>
                </view>
            </view>
            <view v-if="currentTab == 0" class="self-adaption position-sticky" style="height: 93vh">
                <view v-if="menuList.length > 0" id="menu_list" :class="ifScroll ? '' : 'menu_list'" style="height: 93vh">
                    <mp-vtabs
                        :ifScrolling="ifScroll"
                        :vtabs="menuList"
                        :activeTab="activeTab"
                        tabBarClass="mp-vtabs-class-1"
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
                                        <view class="categoryName-view">{{ menu.name }}</view>
                                    </view>
                                    <block v-for="(chil, goodsIndex) in menu.goodsList" :key="goodsIndex">
                                        <view :class="'commodity-item ' + (chil.goodsStatus == 4 ? 'isEnd' : '')" :hover-class="chil.goodsStatus == 4 ? '' : 'hover-class-public'">
                                            <view
                                                class="commodity-name-view"
                                                @tap="parseEventDynamicCode($event, chil.goodsStatus == 4 ? '' : 'commodityDetailTap')"
                                                :data-id="chil.goodsId"
                                            >
                                                <image
                                                    :src="chil.mainImage ? chil.mainImage : '../../assets/images/load-image.png'"
                                                    mode="aspectFill"
                                                    class="commodity-image"
                                                ></image>
                                                <view class="sell-out" v-if="chil.goodsStatus == 4">售罄</view>
                                                <view class="commodity-name-english-view">
                                                    <view class="commodity-name">{{ chil.goodsName }}</view>
                                                    <view class="commodity-english"><text></text></view>
                                                    <view class="money-view">￥{{ chil.goodsPrice }}</view>
                                                </view>
                                            </view>
                                        </view>

                                        <view class="money-insert-view" v-if="!shopInfo.isOutofDeliveryRange && shopInfo.isOperatingOfShop && shopInfo.shop.isOperating">
                                            <!-- <view class="insert-view theme-bg">＋</view> -->
                                            <view class="stepper" v-if="chil.number > 0">
                                                <block>
                                                    <view class="flex_center car_reduce_add reduce-class" @tap="bindMinus" :data-cartId="chil.cartId" :data-number="chil.number">
                                                        －
                                                    </view>
                                                    <input disabled type="number" :value="chil.number" class="radd-reduce-input" />
                                                    <view
                                                        @tap="parseEventDynamicCode($event, chil.goodsStatus != 4 ? 'openSpecifications' : '')"
                                                        class="flex_center car_reduce_add add-class"
                                                        :data-goodsId="chil.goodsId"
                                                    >
                                                        ＋
                                                    </view>
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
                    <!-- <view id="menu_scorll_mask" class="{{ifScroll?'':'menu_scorll_mask'}}"></view> -->
                </view>
                <is-show-tip :isShow="menuList.length <= 0" :src="noDataTip" :top="25" :bottom="20" :text="menuList.length <= 0" tipText="暂无商品"></is-show-tip>
                <view class="shopping-cart-detail" id="shopping-cart-detail">
                    <view class="content-fullReductionRuleName">
                        <!-- {{fullPriceReductionIsHidden?fullReductionRuleName:'暂无优惠'}} -->
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
                    <view :class="'shopping-cart-content ' + (shopInfo.isOutofDeliveryRange || !shopInfo.isOperatingOfShop || !shopInfo.shop.isOperating ? 'isEnd' : '')">
                        <view
                            class="shopping-cart-left"
                            @tap="
                                parseEventDynamicCode($event, !shopInfo.isOutofDeliveryRange && shopInfo.isOperatingOfShop && shopInfo.shop.isOperating ? 'openShoppingCart' : '')
                            "
                        >
                            <view>
                                <view :class="'iconfont icongouwuche1 highlight ' + (shoppingCartList.length <= 0 ? 'theme-other-bg' : 'theme-bg')">
                                    <view class="num" v-if="totalNum > 0">{{ totalNum }}</view>
                                </view>
                            </view>
                            <view>
                                <view class="shopping-cart-totalPrice">
                                    <view :class="fullPriceReductionIsHidden ? 'fullPriceReductionClass' : 'totalPrice'" v-if="shoppingCartList.length > 0">
                                        ￥{{ totalPrice }}
                                    </view>
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
                                    shoppingCartList.length <= 0 ||
                                        !isStartDeliveryPrice ||
                                        shopInfo.isOutofDeliveryRange ||
                                        !shopInfo.isOperatingOfShop ||
                                        !shopInfo.shop.isOperating
                                        ? ''
                                        : 'goToPay'
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
                                            <view class="flex_center car_reduce_add reduce-class" @tap="bindMinus" :data-cartid="item.id" :data-number="item.number">－</view>
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
                                <image :src="goodsInfo.mainImage" mode="aspectFill" class="commodity-image"></image>
                                <view>
                                    <view class="goods-info-name">{{ goodsInfo.name }}</view>
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
                                </view>
                            </scroll-view>
                            <view slot="footer">
                                <view class="good-choice-btn theme-bg" @tap="insertShoppingCart">我选好了</view>
                            </view>
                        </view>
                    </mp-halfScreenDialog>
                </view>
            </view>

            <view v-if="currentTab == 1">
                <view class="evaluate-business-info">
                    <view class="evaluate-info-left">
                        <view class="evaluate-business-info evaluate-info-left-business">
                            <view class="evaluate-business-info business-evaluate">
                                <view class="evaluate-total-score">{{ shopInfo.shop.serviceRating }}</view>
                                <view class="evaluate-total-star">
                                    <view>商家评分</view>
                                    <view>
                                        <multiple-rate :rate="shopInfo.shop.serviceRating > 4 && shopInfo.shop.serviceRating < 5" :disabled="disabled"></multiple-rate>
                                    </view>
                                </view>
                            </view>
                            <!-- <view>
						<view class="evaluate-total-star">味道</view>
						<view class="evaluate-total-num">5.0</view>
					</view>
					<view>
						<view class="evaluate-total-star">包装</view>
						<view class="evaluate-total-num">5.0</view>
					</view> -->
                        </view>
                    </view>
                    <!-- <view class="evaluate-info-right">
				<view class="evaluate-total-star">配送</view>
				<view class="evaluate-total-num">5.0</view>
			</view> -->
                </view>
                <view class="view-line"></view>
                <view class="evaluate-items">
                    <view class="evaluate-item" v-for="(item, appraiseIndex) in appraiseList" :key="appraiseIndex">
                        <image :src="item.headImg" class="evaluate-user-image" mode="widthFix"></image>

                        <view class="evaluate-item-detail">
                            <view class="evaluate-itemu-username-time">
                                <text class="username-detail out_of_range one_row">{{ item.username }}</text>
                                <text class="datetime-detail">{{ item.createTime }}</text>
                            </view>
                            <multiple-rate :rate="item.level" :disabled="disabled"></multiple-rate>
                            <view class="username-detail">
                                {{ item.content }}
                            </view>
                            <view v-if="item.imagesUrls.length > 0">
                                <block v-for="(img, index) in item.imagesUrls" :key="index">
                                    <image :src="img" :data-index="index" :data-imgs="item.imagesUrls" @tap="viewImage" class="images-url" mode="aspectFill"></image>
                                </block>
                            </view>
                            <view class="pl-dz-class">
                                <text></text>
                                <view class="pl-dz-view">
                                    <view class="pl-dz-display" :data-appraiseid="item.id" :data-username="item.username" :data-appraiseindex="appraiseIndex" @tap="appraiseTap">
                                        <text class="iconfont iconpinglun"></text>
                                        <text :decode="true" class="appraise-class-pl-dz">评论&nbsp;&nbsp;</text>
                                    </view>
                                    <view
                                        class="pl-dz-display"
                                        @tap="parseEventDynamicCode($event, item.isGiveLike == 1 ? 'deleteLikeCountTap' : 'giveLikeCountTap')"
                                        :data-appraiseid="item.id"
                                        data-type="1"
                                        :data-appraiseindex="appraiseIndex"
                                    >
                                        <text :class="'iconfont icondianzan ' + (item.isGiveLike == 1 ? 'theme-color' : '')"></text>
                                        <text :class="'appraise-class-pl-dz ' + (item.isGiveLike == 1 ? 'theme-color' : '')">
                                            {{ item.giveLikeCount > 0 ? '点赞' + item.giveLikeCount : '点赞' }}
                                        </text>
                                    </view>
                                </view>
                            </view>
                            <view class="appraise-reply-items" v-if="item.replyPageInfo.list.length > 0" @tap="handleTap">
                                <view class="reply-item list-item" v-for="(item, index) in item.replyPageInfo.list" :key="index">
                                    <mp-select-text
                                        show-copy-btn
                                        :placement="item.placement"
                                        :value="(item.answeredUsername ? item.username + ' 回复 ' + item.answeredUsername : item.username) + '：' + item.content"
                                        :data-username="item.username"
                                        :data-appraiseid="item.appraiseId"
                                        :data-replyid="item.id"
                                        data-type="2"
                                        :data-appraiseindex="appraiseIndex"
                                        :on-document-tap="evt"
                                        @tap.native="
                                            replyTap($event, { username: item.username, appraiseid: item.appraiseId, replyid: item.id, type: '2', appraiseindex: appraiseIndex })
                                        "
                                        @copy="
                                            handleTap($event, { username: item.username, appraiseid: item.appraiseId, replyid: item.id, type: '2', appraiseindex: appraiseIndex })
                                        "
                                        @currency="
                                            handleCurrencyTap($event, {
                                                username: item.username,
                                                appraiseid: item.appraiseId,
                                                replyid: item.id,
                                                type: '2',
                                                appraiseindex: appraiseIndex
                                            })
                                        "
                                        :selectTexts="item.selectTexts"
                                    ></mp-select-text>
                                </view>
                            </view>
                        </view>
                    </view>
                </view>
            </view>

            <view class="swiper-bussiness" v-if="currentTab == 2">
                <!-- <view class="commodity-menu-view" style="{{scrollTop?marginTopMenu:''}}"> -->
                <scroll-view class="swiper-tab scroll-views swiper-tab-bussiness" scroll-y>
                    <!-- <view class="swiper-bussiness-item">
				<view class="swiper-bussiness-title">配送信息</view>
				<view class="swiper-bussiness-info">
					<text class="theme-bg">饿了么专送</text>
					<view>约40分钟送达，距离897m</view>
				</view>
				<view class="swiper-bussiness-info">配送费￥1</view>
			</view> -->
                    <view class="swiper-bussiness-item" v-if="shopInfo.shop.shopWithinImgs.length > 0">
                        <view class="swiper-bussiness-title">商家实景</view>
                        <swiper
                            :indicator-dots="indicatorDots"
                            class="carousel-swiper"
                            :autoplay="autoplay"
                            :interval="interval"
                            :duration="duration"
                            :indicator-active-color="afterColor"
                            style="margin-top: 10rpx"
                        >
                            <block v-for="(item, index) in shopInfo.shop.shopWithinImgs" :key="index">
                                <swiper-item class="carousel-swiper-item">
                                    <image :src="item" class="carousel-image" mode="scaleToFill" :data-imageLinkUrl="item" />
                                </swiper-item>
                            </block>
                        </swiper>
                    </view>
                    <view class="swiper-bussiness-item">
                        <view class="swiper-bussiness-title">商家信息</view>
                        <view class="swiper-bussiness-info">
                            <view class="swiper-bussiness-row">
                                <text>{{ shopInfo.shop.briefIntroduction ? shopInfo.shop.briefIntroduction : '暂无简介' }}</text>
                            </view>
                        </view>
                    </view>
                    <view class="view-line"></view>
                    <view class="swiper-bussiness-row">
                        <text class="swiper-bussiness-row-left">品类</text>
                        <text>{{ shopInfo.shop.managePrimary }}</text>
                    </view>
                    <view class="view-line"></view>
                    <view class="swiper-bussiness-row" @tap="contactBussinessTap">
                        <text class="swiper-bussiness-row-left">商家电话</text>
                        <text class="contact-bussiness-text">联系商家</text>
                    </view>
                    <view class="view-line"></view>
                    <view class="swiper-bussiness-row">
                        <text class="swiper-bussiness-row-left">地址</text>
                        <view>
                            <view>
                                {{ shopInfo.shop.province ? shopInfo.shop.province : '' }}{{ shopInfo.shop.city ? shopInfo.shop.city : '' }}
                                {{ shopInfo.shop.area ? shopInfo.shop.area : '' }}
                            </view>
                            <view>{{ shopInfo.shop.street }}</view>
                        </view>
                    </view>
                    <view class="view-line"></view>
                    <view class="swiper-bussiness-row">
                        <text class="swiper-bussiness-row-left">营业时间</text>
                        <text>{{ shopInfo.shop.startTime }}-{{ shopInfo.shop.endTime }}</text>
                    </view>
                    <view class="view-line"></view>
                    <view class="swiper-bussiness-item">
                        <view class="swiper-bussiness-title">商家资质</view>
                    </view>
                </scroll-view>
            </view>
        </view>
        <view class="pinglun-bottom" v-if="currentTab == 1 && appraiseFocus">
            <textarea
                class="userinfo-text input-appraise"
                :focus="appraiseFocus"
                :show-confirm-bar="showConfirmBar"
                @blur="bindblur"
                :placeholder="'回复 ' + userName"
                @input="bindInputAppraise"
                cursor-spacing="152"
                @confirm="bindconfirm"
                confirm-type="send"
                :value="content"
                :fixed="!showConfirmBar"
            ></textarea>
            <view class="reply-button-view">
                <button size="mini" class="theme-bg" @tap="bindconfirm">回复</button>
            </view>
        </view>
        <mp-halfScreenDialog :show="isActivityDialog" extClass="extClassSpecifications">
            <view slot="title">优惠活动</view>
            <view slot="desc">
                <scroll-view style="height: 55vh" scroll-y>
                    <view v-if="shopInfo.promotionList.length > 0">
                        <text class="dialog-title">优惠：</text>
                        <view class="business-discount-info business-info-flex">
                            <view class="business-discount out_of_range one_row" @tap="isPromotionTap">
                                <view class="theme-color-border business-discount-list" v-if="index <= 5" v-for="(rule, index) in shopInfo.promotionList" :key="index">
                                    {{ rule.name }}
                                </view>
                            </view>
                        </view>
                    </view>
                    <view class="reduced-delivery-price" v-if="shopInfo.shop.reducedDeliveryPrice">
                        <text class="dialog-title">配送费：</text>
                        配送费立减{{ shopInfo.shop.reducedDeliveryPrice }}元
                    </view>
                    <view class="reduced-delivery-price" v-if="shopInfo.shop.announcement">{{ shopInfo.shop.announcement }}</view>
                </scroll-view>
            </view>
        </mp-halfScreenDialog>
        <mp-halfScreenDialog :show="isOutofDeliveryRangeDialog" extClass="extClassSpecifications">
            <view slot="title">本店超出配送范围</view>
            <view slot="desc">
                <view class="edit-address-class">建议您更换地址再下单</view>
            </view>
            <view slot="footer">
                <button size="mini" class="edit-address-btn theme-bg" @tap="editAddress" style="font-size: 24rpx">更换地址</button>
            </view>
        </mp-halfScreenDialog>
        <!-- <view class="pinglun-bottom" wx:if="{{currentTab==1&&appraiseFocus}}">
	<input class="userinfo-text input-appraise" focus="{{appraiseFocus}}" bindblur="bindblur" placeholder="评论" bindinput="bindInputAppraise" bindconfirm="bindconfirm" confirm-type="send" value="{{content}}"></input>
</view> -->

        <!-- <mp-dialog show="{{isShopDetail}}" buttons="{{buttons}}" mask-closable="{{maskClosable}}" catchtouchmove="true"
ext-class="{{extClassShopDetail}}" bind:tap="close">
	<view class="shop-detail-dialog">
		<view>
			<view></view>
		</view>
		<view>公告</view>
		<view>
			{{shopInfo.shop.announcement}}
		</view>
	</view>
</mp-dialog> -->
    </view>
</template>

<script>
import isShowTip from '../../../components/is-show-tip/is-show-tip';
import mpDialog from '../../../components/dialog/dialog';
import mpHalfScreenDialog from '../../../components/half-screen-dialog/half-screen-dialog';
import mpVtabs from '../../../components/vtabs/index';
import mpVtabsContent from '../../../components/vtabs-content/index';
import multipleRate from '../../../components/multiple-rate/multiple-rate';
import mpSelectText from '../../../components/select-text/index';
import GlobalConfig from '../../../utils/global-config';
import https from '../../../utils/http';
import toastService from '../../../utils/toast.service';
import authService from '../../../utils/auth';
import utilHelper from '../../../utils/util';
import CustomPage from '../../../base/CustomPage';
import dateHelper from '../../../utils/date-helper';
var app = getApp();
//声明全局变量
var proListToTop = [];
var isInitShow = true;
var totalPrice = 0;
var totalNum = 0;
var menuList = [];
export default {
    components: {
        isShowTip,
        mpDialog,
        mpHalfScreenDialog,
        mpVtabs,
        mpVtabsContent,
        multipleRate,
        mpSelectText
    },
    data() {
        return {
            staticImg: app.globalData.staticImg,
            currentActiveIndex: 0,
            menuList: [],
            imageTip: '../../assets/common/icon-internet-error.png',
            indicatorDots: true,
            autoplay: true,
            interval: 5000,
            duration: 1000,

            //beforeColor: "white",//指示点颜色,
            afterColor: 'white',

            //当前选中的指示点颜色
            extClass: 'weui-dialog-ext-index',

            businessModes: [
                {
                    modeName: '点餐',
                    modeId: 0
                },
                {
                    modeName: '评价',
                    modeId: 1
                },
                {
                    modeName: '商家',
                    modeId: 2
                }
            ],

            currentTab: 0,

            data: {
                dialog1: false,
                dialog2: false,
                name: ''
            },

            deliveryPrice: 0,
            shoppingCartDialog: false,
            specificationsDialog: false,
            activeTab: 0,
            rate: 5,
            disabled: true,
            priceDifference: 0,
            isStartDeliveryPrice: false,
            appraiseFocus: false,
            buttons: [],
            appraiseFocus: false,
            showConfirmBar: false,
            shoppingCartList: [],
            isShopDetail: false,
            isActivityDialog: false,
            isOutofDeliveryRangeDialog: false,
            netWorkType: '',
            dialogShow: false,
            maskClosable: false,
            proListToTop: [],
            businessRecommendHeight: '',
            ifScroll: false,
            shopId: '',
            screenHeight: '',
            winHeight: '',
            leftHeight: '',
            bussinessHeight: '',
            carouselUrls: '',
            toView: '',

            shopInfo: {
                shop: {
                    firstPoster: '',
                    shopLogoImg: '',
                    name: '',
                    serviceRating: '',
                    reducedDeliveryPrice: '',
                    announcement: '',
                    isOperating: '',
                    shopWithinImgs: [],
                    briefIntroduction: false,
                    managePrimary: '',
                    province: false,
                    city: false,
                    area: false,
                    street: '',
                    startTime: '',
                    endTime: ''
                },

                promotionList: [],
                isOutofDeliveryRange: '',
                isOperatingOfShop: ''
            },

            fullPriceReduction: '',
            fullReductionRuleName: '',
            fullPriceReductionIsHidden: '',
            appraiseList: '',
            merchantRecommendGoods: '',
            totalNum: '',
            packingCharges: '',
            totalPrice: '',
            appraise: '',
            appraiseId: '',
            userName: '',
            replyId: '',
            replyType: '',
            appraiseIndex: '',
            appraiseJson: '',
            content: '',
            appraiseHeight: '',
            goodsId: '',

            goodsInfo: {
                mainImage: '',
                name: ''
            },

            priceAfter: '',
            specListString: '',
            specList: '',
            pageTopView: '',
            dps: '',
            shoppingCartDetail: '',
            contentHeight: '',

            rule: {
                name: ''
            },

            goodsItem: {
                goodsStatus: 0,
                goodsId: '',
                mainImage: '',
                goodsName: '',
                latelyMonthlySales: '',
                goodsPrice: '',
                number: 0,
                cartId: ''
            },

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

            noDataTip: '',
            img: '',
            evt: ''
        };
    },
    /**
     * 生命周期函数--监听页面加载
     */
    onLoad: function (options) {
        this.onLoadClone3389(options);
    },
    /**
     * 生命周期函数--监听页面初次渲染完成
     */
    onReady: function () {
        console.log('onReady');
        var that = this;
        // toastService.showLoading();
        // let timeout = setTimeout(() => {
        //   that.getAllRects();
        //   clearTimeout(timeout);
        // }, 1000);
        this.getCarouselList();
        this.getSystemUsageRecord();
    },
    /**
     * 生命周期函数--监听页面显示
     */
    onShow: function () {
        // app.getMemberInfo().then(result => {
        //   if (result.isNewPeople && result.isRemindNewPeople) {
        //     this.openConfirm();
        //   }
        // });

        var that = this;
        this.getShopInfo();
        this.getAppraiseList();
        this.getMerchantRecommendGoods();
        //this.setData(app.deliveryAndSelfTaking);
        //this.getShoppingCartList();
        // let timeout = setTimeout(() => {
        //   that.autoHeight();
        //   clearTimeout(timeout);
        // }, 1000)
        this.getMenuList();
        uni.getNetworkType({
            success: function (res) {
                //console.log(res.networkType)
                let networkType = res.networkType;
                if (networkType != 'none') {
                }
                that.setData({
                    netWorkType: res.networkType == 'none' ? true : false
                });
            }
        });
    },
    /**
     * 生命周期函数--监听页面隐藏
     */
    onHide: function () {
        this.setData({
            dialogShow: false,
            maskClosable: true,
            proListToTop: []
        });
    },
    /**
     * 生命周期函数--监听页面卸载
     */
    onUnload: function () {},
    /**
     * 页面相关事件处理函数--监听用户下拉动作
     */
    onPullDownRefresh: function () {
        // 显示顶部刷新图标
        uni.showNavigationBarLoading();
        this.getMenuList();
        // 隐藏导航栏加载框
        uni.hideNavigationBarLoading();
        // 停止下拉动作
        uni.stopPullDownRefresh();
    },
    /**
     * 页面上拉触底事件的处理函数
     */
    onReachBottom: function () {},
    /**
     * 用户点击右上角分享
     */
    onShareAppMessage: function () {},
    // onPageScroll(e) {
    //   console.log(e);
    //   var height=(this.data.pageTopHeight);
    //   height=(height*2)-13;
    //   console.log(height)
    //   if (e.scrollTop >= (height)) {
    //     if (!this.data.scrollTop) {
    //       this.setData({
    //         scrollTop: true
    //       })
    //     }
    //   }else{
    //     if (this.data.scrollTop) {
    //       this.setData({
    //         scrollTop: false
    //       })
    //     }
    //   }
    //   // if (windowHeight < 600) {
    //   //   height = 173;
    //   // } else if (windowHeight > 700) {
    //   //   height = 49;
    //   // } else if (windowHeight < 620) {
    //   //   height = 51;
    //   // } else {
    //   //   height = 183;
    //   // }
    //   // if (e.scrollTop > height) {
    //   //   if (!this.data.scrollTop) {
    //   //     this.setData({
    //   //       scrollTop: true
    //   //     })
    //   //   }
    //   // } else {
    //   //   if (this.data.scrollTop) {
    //   //     this.setData({
    //   //       scrollTop: false
    //   //     })
    //   //   }
    //   // }
    // }
    onPageScroll(e) {
        console.log(e);
        var _this = this;
        var pageTopView = this.pageTopView;
        var dps = this.dps;
        uni.createSelectorQuery()
            .in(uni)
            .selectAll('#businessRecommend')
            .boundingClientRect(function (rect) {
                console.log(rect);
                var businessRecommendHeight = rect[0].height * 2;
                _this.setData({
                    businessRecommendHeight: rect[0].height * 2
                });
                if (e.scrollTop >= pageTopView / 2 + dps / 2 + businessRecommendHeight / 2) {
                    _this.setData({
                        ifScroll: true
                    });
                } else {
                    _this.setData({
                        ifScroll: false
                    });
                }
            })
            .exec();
    },
    methods: {
        /**
         * 生命周期函数--监听页面加载
         */
        onLoadClone3389: function (options) {
            // 确保页面数据已经刷新完毕~
            authService.checkIsLogin().then((result) => {
                console.log(result);
                if (result) {
                    return;
                }
            });
            this.setData({
                shopId: options.id,
                screenHeight: app.globalData.systemInfoSync.screenHeight
            });
            this.topViewHeight();
        },

        closeShoppingCart: function () {
            this.setData({
                shoppingCartDialog: false
            });
        },

        close: function () {
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

        // 滑动切换tab
        bindSlideChange: function (e) {
            var that = this;
            that.setData({
                currentTab: e.detail.current
            });
        },

        //点击切换
        clickTab: function (e) {
            var that = this;
            this.setData({
                shoppingCartDialog: false,
                specificationsDialog: false
            });
            if (that.currentTab === e.target.dataset.current) {
                return false;
            } else {
                that.setData({
                    currentTab: e.target.dataset.current
                });
            }
        },

        autoHeight() {
            var that = this;
            isInitShow = false;
            const query = uni.createSelectorQuery();
            let windowHeight = app.globalData.systemInfoSync.windowHeight;
            console.log(windowHeight);
            query
                .selectAll('.swiper-tabs-choice,.content-manjian')
                .boundingClientRect(function (rect) {
                    console.log(rect);
                    let height = windowHeight > 630 ? rect[0].height - 10 : rect[0].height + 25;
                    that.setData({
                        winHeight: windowHeight - height,
                        leftHeight: windowHeight - height,
                        bussinessHeight: windowHeight - height
                    });
                })
                .exec();
            // query.selectAll('.swiper-tabs-choice').boundingClientRect(function (rect) {
            //   console.log(rect)
            //   let height = (windowHeight > 600 ? windowHeight - 10 : windowHeight - 44)
            //   that.setData({
            //     winHeight: height,
            //     leftHeight: height
            //   })
            // }).exec();
        },

        commodityDetailTap(e) {
            console.log(e.currentTarget.dataset.id);
            uni.navigateTo({
                url: '../detail/detail?id=' + e.currentTarget.dataset.id + '&shopId=' + this.shopId
            });
        },

        getCarouselList() {
            https
                .request('/api-promotion/rest/advertisement/list', {
                    type: 2,
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

        radioChange(e) {
            //console.log(e)
            authService.checkIsLogin().then((result) => {
                if (result) {
                    let checkValue = e.currentTarget.dataset.radioindex;
                    let deliveryData = JSON.stringify(this.deliveryData);
                    uni.navigateTo({
                        url: '../../address/choose/choose?radioIndex=' + checkValue + '&deliveryData=' + deliveryData + '&pageType=menu'
                    });
                    return;
                }
                app.globalData.checkIsAuth('scope.userInfo');
            });
        },

        scroll(e) {
            //console.log(e)
            //获取当前滑动距离顶部的距离高度
            let scrollTop = e.detail.scrollTop;
            //console.log(scrollTop)
            //获取右侧菜单的高度
            let scrollArr = proListToTop;
            //当距离顶部的高度大于等于右侧菜单的item高度
            let index = 0;
            if (scrollTop >= scrollArr[scrollArr.length - 1]) {
                return;
            } else {
                for (let i = 0; i < scrollArr.length; i++) {
                    if (scrollTop >= 0 && scrollTop < scrollArr[0]) {
                        index = 0;
                    } else if (scrollTop >= scrollArr[i - 1] && scrollTop < scrollArr[i]) {
                        index = i;
                    } else if (scrollTop <= 0) {
                    }
                }
            }
            this.setData({
                currentActiveIndex: index
            });
        },

        selectMenuTap(e) {
            var index = e.currentTarget.dataset.index;
            console.log(index);
            this.setData({
                toView: 'NAV' + index.toString(),
                currentActiveIndex: index
            });
        },

        getAllRects() {
            // 获取商品数组的位置信息
            let menuHeight = 0;
            uni.createSelectorQuery()
                .in(uni)
                .selectAll('.commodity-item-view')
                .boundingClientRect(function (rects) {
                    rects.forEach(function (rect) {
                        menuHeight += rect.height;
                        proListToTop.push(menuHeight);
                    });
                    toastService.hideLoading();
                })
                .exec();
        },

        getShopInfo() {
            https
                .request('/api-merchant/rest/shop/detail', {
                    id: this.shopId,
                    position: app.globalData.deliveryAndSelfTaking.location
                })
                .then((result) => {
                    if (result.success && result.data) {
                        if (result.data.isOutofDeliveryRange) {
                            this.setData({
                                isOutofDeliveryRangeDialog: true
                            });
                        }
                        this.getFullReductionRule(result.data);
                    }
                });
        },

        //获取满减规则
        getFullReductionRule(shopInfo) {
            //console.log(shopInfo)
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
            shopInfo.shop.firstPoster = GlobalConfig.ossUrl + shopInfo.shop.firstPoster;
            shopInfo.shop.shopWithinImgs = shopWithinImgs;
            //shopInfo.shop.isOperating = false;
            app.globalData.getIsItNear(shopInfo.shop.startTime, shopInfo.shop.endTime);
            this.setData({
                shopInfo: shopInfo,
                fullPriceReduction: utilHelper.toFixed(fullPriceReduction, 2),
                fullReductionRuleName: fullReductionRuleName,
                fullPriceReductionIsHidden: fullPriceReductionIsHidden
            });
            authService.checkIsLogin().then((result) => {
                if (result) {
                    this.getShoppingCartList();
                }
            });
        },

        getMenuList() {
            toastService.showLoading();
            var self = this;
            https
                .request('/api-goods/rest/menu/listWithGoods', {
                    shopId: this.shopId
                })
                .then((result) => {
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

        getAppraiseList() {
            https
                .request('/api-order/rest/member/appraise/list', {
                    shopId: this.shopId,
                    pageNo: -1,
                    pageSize: 20
                })
                .then((result) => {
                    if (result.success && result.data) {
                        result.data.records.forEach((aitem, index) => {
                            //console.log(aitem.imagesUrl)
                            aitem.imagesUrls = [];
                            if (aitem.imagesUrl) {
                                aitem.imagesUrl.split(',').forEach((item) => {
                                    item = GlobalConfig.ossUrl + item;
                                    aitem.imagesUrls.push(item);
                                });
                            }
                            if (aitem.replyPageInfo) {
                                aitem.replyPageInfo.list.forEach((bitem, bIndex) => {
                                    let goodt = bitem.isGiveLike == 0 ? '点赞' : '取消点赞';
                                    let type = bitem.isGiveLike == 0 ? 'good' : 'deletegood';
                                    let good = goodt + (bitem.giveLikeCount > 0 ? ' ' + bitem.giveLikeCount : '') + '';
                                    //console.log("获取是否点赞============" + goodt)
                                    bitem.selectTexts = [
                                        {
                                            name: '复制',
                                            type: 'copy',
                                            bind: 'handleCopy'
                                        },
                                        {
                                            name: good,
                                            type: type,
                                            bind: ''
                                        }
                                    ];
                                    //console.log(bitem)
                                    if (bitem.replierType == 2) {
                                        bitem.username = '商家回复';
                                    }
                                    // if(app.globalData.loginUserInfo.id==bitem.memberId){
                                    //   bitem.selectTexts.push({
                                    //     "name": "删除",
                                    //     "type": "delete",
                                    //     "bind": ""
                                    //   })
                                    // }
                                    bitem.placement = 'top';
                                });
                            }
                            aitem.createTime = dateHelper.fmtDate(aitem.createTime);
                        });
                        //console.log(result.data.records)
                        this.setData({
                            appraiseList: result.data.records
                        });
                    }
                });
        },

        getShoppingCartList() {
            var that = this;
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
                        menuList.forEach((menu, menuIndex) => {
                            menu.goodsList.forEach((goods, goodsIndex) => {
                                let number = 0;
                                menuList[menuIndex].goodsList[goodsIndex].number = number;
                                result.data.records.forEach((result, carIndex) => {
                                    if (goods.goodsId == result.goodsId) {
                                        number = number + result.number;
                                        menuList[menuIndex].goodsList[goodsIndex].isShopCart = true;
                                        result.menuIndex = menuIndex;
                                        result.goodsIndex = goodsIndex;
                                        menuList[menuIndex].goodsList[goodsIndex].number = number;
                                        menuList[menuIndex].goodsList[goodsIndex].cartId = result.id;
                                    }
                                });
                            });
                        });
                        this.merchantRecommendGoods.forEach((merchants, merchanIndex) => {
                            let merchant = 0;
                            merchants.number = merchant;
                            result.data.records.forEach((result, carIndex) => {
                                if (merchants.goodsId == result.goodsId) {
                                    merchant = merchant + result.number;
                                    merchants.isShopCart = true;
                                    merchants.number = merchant;
                                    merchants.cartId = result.id;
                                }
                            });
                        });
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
                            //console.log(result)
                        });
                        menuList = menuList;
                        totalPrice = utilHelper.toFixed(totalPrice, 2);
                        var isStartDeliveryPrice = false;
                        var priceDifference = 0;
                        this.shopInfo.shop.startDeliveryPrice = this.shopInfo.shop.startDeliveryPrice ? this.shopInfo.shop.startDeliveryPrice : 0;
                        if (totalPrice + packingCharges >= this.shopInfo.shop.startDeliveryPrice) {
                            isStartDeliveryPrice = true;
                        }
                        priceDifference = this.shopInfo.shop.startDeliveryPrice - (totalPrice + packingCharges);
                        this.setData({
                            menuList: menuList,
                            merchantRecommendGoods: this.merchantRecommendGoods,
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
                            fullPriceReductionIsHidden: fullPriceReductionIsHidden
                        });
                    }
                });
        },

        getMerchantRecommendGoods() {
            https
                .request('/api-goods/rest/merchantRecommendGoods/list', {
                    pageNo: -1,
                    pageSize: 1,
                    shopId: this.shopId
                })
                .then((result) => {
                    if (result.success) {
                        result.data.records.forEach(function (item, index) {
                            item.mainImage = GlobalConfig.ossUrl + item.mainImage;
                            item.isShopCart = false;
                        });
                        this.setData({
                            merchantRecommendGoods: result.data.records
                        });
                    }
                });
        },

        bindInputAppraise(e) {
            this.setData({
                appraise: e.detail.value
            });
        },

        viewImage: function (event) {
            var imgs = event.currentTarget.dataset.imgs; //获取data-src
            var index = event.currentTarget.dataset.index; //获取data-currentimg
            var src = imgs[index];

            //图片预览
            uni.previewImage({
                current: src,
                // 当前显示图片的http链接
                urls: imgs // 需要预览的图片http链接列表
            });
        },

        appraiseTap(e) {
            let appraiseId = e.currentTarget.dataset.appraiseid;
            let userName = e.currentTarget.dataset.username;
            let replyType = 1;
            let replyId = '';
            let appraiseIndex = e.currentTarget.dataset.appraiseindex;
            let appraiseJson = {
                appraiseId: this.appraiseId,
                userName: this.userName,
                replyType: this.replyType,
                appraiseIndex: this.appraiseIndex,
                replyId: this.replyId
            };
            this.setData({
                appraiseId: appraiseId,
                userName: userName,
                replyId: replyId,
                replyType: replyType,
                appraiseIndex: appraiseIndex,
                appraiseJson: appraiseJson,
                content: appraiseId == appraiseJson.appraiseId ? this.content : '',
                appraiseFocus: true
            });
            this.getAppraiseTextareaHeight();
        },

        getAppraiseTextareaHeight() {
            let that = this;
            const query = uni.createSelectorQuery();
            query.select('.pinglun-bottom').boundingClientRect();
            query.selectViewport().scrollOffset();
            query.exec(function (res) {
                console.log();
                that.setData({
                    appraiseHeight: res[0].height
                });
            });
            console.log(this.appraiseHeight);
        },

        handleTap(e, _dataset) {
            /* ---处理dataset begin--- */
            this.handleDataset(e, _dataset);
            /* ---处理dataset end--- */
            console.log(e);
            //this.setData({ evt: e })
        },

        handleCurrencyTap(e, _dataset) {
            /* ---处理dataset begin--- */
            this.handleDataset(e, _dataset);
            /* ---处理dataset end--- */
            console.log(e);
            this.setData({
                appraiseId: e.currentTarget.dataset.appraiseid,
                appraiseIndex: e.currentTarget.dataset.appraiseindex
            });
            if (e.detail.type == 'deletegood') {
                this.deleteLikeCountTap(e);
            }
            if (e.detail.type == 'good') {
                this.giveLikeCountTap(e);
            }
            if (e.detail.type == 'delete') {
                let url = '';
                if (e.currentTarget.dataset.type == '1') {
                    url = '/api-order/rest/member/appraise/delete';
                }
                if (e.currentTarget.dataset.type == '2') {
                    url = '/api-user/rest/member/reply/delete';
                }
                this.deleteAppraise(e, url);
            }
        },

        deleteAppraise(e, url) {
            toastService.showLoading();
            let replyId = e.currentTarget.dataset.replyid;
            let type = e.currentTarget.dataset.type;
            https
                .request(url, {
                    id: replyId
                })
                .then((result) => {
                    toastService.hideLoading();
                    if (result.success) {
                        if (type == '1') {
                            this.appraiseList[appraiseIndex].giveLikeCount -= 1;
                            this.appraiseList[appraiseIndex].isGiveLike = true;
                        }
                        this.setData({
                            appraiseList: this.appraiseList
                        });
                    }
                });
        },

        giveLikeCountTap(e) {
            toastService.showLoading();
            this.setData({
                appraiseId: e.currentTarget.dataset.appraiseid,
                appraiseIndex: e.currentTarget.dataset.appraiseindex
            });
            let appraiseId = e.currentTarget.dataset.appraiseid;
            let appraiseIndex = e.currentTarget.dataset.appraiseindex;
            let replyId = e.currentTarget.dataset.replyid;
            let type = e.currentTarget.dataset.type;
            let data = {
                appraiseId: appraiseId,
                type: type
            };
            if (replyId) {
                data.replyId = replyId;
            }
            https.request('/api-order/rest/member/giveLike/insert', data).then((result) => {
                toastService.hideLoading();
                if (result.success) {
                    if (type == '1') {
                        this.appraiseList[appraiseIndex].giveLikeCount += 1;
                        this.appraiseList[appraiseIndex].isGiveLike = true;
                    }
                    this.getReplyByAppraiseId();
                    // this.setData({
                    //   appraiseList: this.data.appraiseList
                    // });
                }
            });
        },

        deleteLikeCountTap(e) {
            this.setData({
                appraiseId: e.currentTarget.dataset.appraiseid,
                appraiseIndex: e.currentTarget.dataset.appraiseindex
            });
            toastService.showLoading();
            let appraiseId = e.currentTarget.dataset.appraiseid;
            let appraiseIndex = e.currentTarget.dataset.appraiseindex;
            let replyId = e.currentTarget.dataset.replyid;
            let type = e.currentTarget.dataset.type;
            let data = {
                appraiseId: appraiseId,
                type: type
            };
            if (replyId) {
                data.replyId = replyId;
            }
            https.request('/api-user/rest/member/giveLike/delete', data).then((result) => {
                if (result.success) {
                    toastService.hideLoading();
                    if (type == '1') {
                        this.appraiseList[appraiseIndex].giveLikeCount -= 1;
                        this.appraiseList[appraiseIndex].isGiveLike = false;
                    }
                    this.getReplyByAppraiseId();
                    // this.setData({
                    //   appraiseList: this.data.appraiseList
                    // });
                }
            });
        },

        bindblur(e) {
            let content = e.detail.value;
            if (this.appraiseId == this.appraiseJson.appraiseId) {
                content = this.content;
            }
            this.setData({
                appraiseFocus: false,
                content: content
            });
        },

        replyTap(e, _dataset) {
            /* ---处理dataset begin--- */
            this.handleDataset(e, _dataset);
            /* ---处理dataset end--- */
            let replyType = 2;
            let appraiseId = e.currentTarget.dataset.appraiseid;
            let replyId = e.currentTarget.dataset.replyid;
            let userName = e.currentTarget.dataset.username;
            let appraiseIndex = e.currentTarget.dataset.appraiseindex;
            console.log('我点击了回复，我回复====' + userName);
            let appraiseJson = {
                appraiseId: this.appraiseId,
                replyType: this.replyType,
                replyId: this.replyId,
                userName: this.userName,
                appraiseIndex: this.appraiseIndex
            };
            this.setData({
                appraiseFocus: true,
                appraiseId: appraiseId,
                replyId: replyId,
                userName: userName,
                replyType: replyType,
                appraiseJson: appraiseJson,
                appraiseIndex: appraiseIndex,
                content: appraiseId == appraiseJson.appraiseId && replyId == appraiseJson.replyId ? this.content : ''
            });
        },

        bindconfirm(e) {
            let appraise = this.appraise;
            if (!appraise) {
                toastService.showToast('请输入您的回复内容');
                return;
            }
            toastService.showLoading();
            https
                .request('/api-order/rest/member/reply/insert', {
                    appraiseId: this.appraiseId,
                    replyId: this.replyId,
                    replyType: this.replyType,
                    content: appraise
                })
                .then((result) => {
                    toastService.hideLoading();
                    if (result.success) {
                        this.setData({
                            appraise: ''
                        });
                        this.getReplyByAppraiseId();
                    }
                });
        },

        getReplyByAppraiseId(e) {
            https
                .request('/api-order/rest/member/reply/list', {
                    appraiseId: this.appraiseId,
                    pageNo: -1,
                    pageSize: 20
                })
                .then((result) => {
                    toastService.hideLoading();
                    if (result.success && result.data) {
                        result.data.records.forEach((bitem, bIndex) => {
                            let goodt = bitem.isGiveLike == 0 ? '点赞' : '取消点赞';
                            let type = bitem.isGiveLike == 0 ? 'good' : 'deletegood';
                            let good = goodt + (bitem.giveLikeCount > 0 ? ' ' + bitem.giveLikeCount : '') + '';
                            bitem.selectTexts = [
                                {
                                    name: '复制',
                                    type: 'copy',
                                    bind: 'handleCopy'
                                },
                                {
                                    name: good,
                                    type: type,
                                    bind: ''
                                }
                            ];
                            if (bitem.replierType == 2) {
                                bitem.username = '商家回复';
                            }
                            // if(app.globalData.loginUserInfo.id==bitem.memberId){
                            //   bitem.selectTexts.push({
                            //     "name": "删除",
                            //     "type": "delete",
                            //     "bind": ""
                            //   })
                            // }
                            bitem.placement = 'top';
                        });
                        this.appraiseList[this.appraiseIndex].replyPageInfo.list = result.data.records;
                        this.setData({
                            appraiseList: this.appraiseList
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
            toastService.hideLoading();
            this.getCommodityDetails(e.currentTarget.dataset.goodsid);
        },

        closeSpecifications: function () {
            this.getShoppingCartList();
            this.setData({
                specificationsDialog: false
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

        selectByGoodsId(goodsId) {
            console.log(this);
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
                    console.log(specList[key][keyof].price);
                    if (specList[key][keyof].checked) {
                        price = price + specList[key][keyof].price;
                        specListString = (specListString ? specListString + '/' : specListString) + specList[key][keyof].name;
                    }
                }
            }
            let totalPrice = price * this.commodityNum;
            let fullPriceReduction = 0;
            let fullReductionRuleName = '';
            let fullPriceReductionIsHidden = false;
            let limitedPrice = 0; //console.log(this.data.fullReductionRuleList)
            for (let i = 0; i < this.shopInfo.fullReductionRuleList.length; i++) {
                //console.log(this.data.fullReductionRuleList[i].limitedPrice)
                if (totalPrice >= this.shopInfo.fullReductionRuleList[i].limitedPrice) {
                    if (this.shopInfo.fullReductionRuleList[i].limitedPrice > limitedPrice) {
                        limitedPrice = this.shopInfo.fullReductionRuleList[i].limitedPrice;
                        fullPriceReduction = totalPrice + this.data.packingCharges - this.shopInfo.fullReductionRuleList[i].reducedPrice;
                        fullReductionRuleName = this.shopInfo.fullReductionRuleList[i].name;
                        fullPriceReductionIsHidden = true;
                    }
                }
            }

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
                        toastService.hideLoading();
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
                toastService.hideLoading();
                //重新计算被选中的商品的总金额
                totalNum++;
                that.getShoppingCartList();
            });
        },

        updateNumber(id, number, type, callbak) {
            authService.checkIsLogin().then((result) => {
                console.log(result);
                if (!result) {
                    app.globalData.checkIsAuth('scope.userInfo');
                    return;
                }
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
            });
        },

        goToPay() {
            authService.checkIsLogin().then((result) => {
                if (!result) {
                    app.globalData.checkIsAuth('scope.userInfo');
                    return;
                }
                //判断店铺是否打烊
                let startTime = this.shopInfo.shop.startTime;
                let endTime = this.shopInfo.shop.endTime;
                let isOperating = this.shopInfo.shop.isOperating;
                app.globalData.getIsBusiness(startTime, endTime, isOperating).then((result) => {
                    if (!result) {
                        return;
                    }
                    this.toPay();
                });
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

        connectedTap() {
            this.onLoadClone3389({});
        },

        gotoShop(e) {
            app.globalData.isRemindNewPeople().then((result) => {
                if (result) {
                    this.setData({
                        dialogShow: false,
                        maskClosable: true
                    });
                }
            });
        },

        openConfirm() {
            this.setData({
                dialogShow: true,
                maskClosable: false
            });
        },

        close(e) {
            console.log(e);
            this.setData({
                specificationsDialog: false,
                shoppingCartDialog: false
            });
        },

        onTabCLick(e) {
            var _this = this;
            const index = e.detail.index;
            console.log('tabClick', index);
            this.setData({
                activeTab: index,
                ifScroll: true
            });
            setTimeout(() => {
                _this.setData({
                    ifScroll: false
                });
            }, 1000);
        },

        isShopDetailTap() {
            this.setData({
                isShopDetail: true
            });
        },

        isPromotionTap() {
            this.setData({
                isActivityDialog: true
            });
        },

        onChange(e) {
            const index = e.detail.index;
            //console.log('change', index)
        },

        contactBussinessTap() {
            uni.makePhoneCall({
                phoneNumber: this.shopInfo.shop.contactPhone
            });
        },

        getSystemUsageRecord() {
            https
                .request('/api-util/rest/systemUsageRecord/insert', {
                    shopId: this.shopId,
                    type: 'intoShop'
                })
                .then((result) => {
                    if (result.success) {
                    }
                });
        },

        editAddress() {
            this.setData({
                isOutofDeliveryRangeDialog: false
            });
            uni.navigateTo({
                url: '../../address/replace/replace?jump_page=menu_index&addressType=true&shopId=' + this.shopInfo.shop.id
            });
        },

        topViewHeight() {
            var _this = this;
            isInitShow = false;
            const query = uni.createSelectorQuery();
            let windowHeight = app.globalData.systemInfoSync.windowHeight;
            let screenHeight = app.globalData.systemInfoSync.screenHeight;
            let statusBarHeight = app.globalData.systemInfoSync.statusBarHeight * 2;
            console.log(windowHeight);
            setTimeout(() => {}, 1000);
            query
                .selectAll('#page-top-view,#d_p_s,#shopping-cart-detail')
                .boundingClientRect(function (rect) {
                    console.log(rect);
                    _this.setData({
                        pageTopView: rect[0].height * 2,
                        dps: rect[1].height * 2,
                        shoppingCartDetail: rect[2].height * 2,
                        contentHeight: screenHeight - rect[1].height - rect[2].height - 48
                    });
                })
                .exec();
            console.log(app.globalData.systemInfoSync);
        }
    }
};
</script>
<style>
@import './index.css';
</style>
