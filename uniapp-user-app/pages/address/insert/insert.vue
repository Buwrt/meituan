<template>
    <view>
        <form @submit="formSubmit" @reset="formReset">
            <view class="address-info-list">
                <view class="address-info-item">
                    <text>联系人</text>
                    <input placeholder="请填写您的姓名" name="realname" />
                </view>
                <view class="address-info-item">
                    <text>性别</text>
                    <radio-group class="sex-radio-group" @change="sexRadioChange" name="sex">
                        <label class="sex-radio-group-label" v-for="(item, index) in sexRadio" :key="index">
                            <radio :value="item.id" :checked="item.checked" class="block-radio"></radio>

                            {{ item.name }}
                        </label>
                    </radio-group>
                </view>
                <view class="address-info-item">
                    <text>手机号</text>
                    <input placeholder="请输入您的手机号" type="number" name="phone" @blur="bindPhoneBlur" />
                </view>
                <view class="address-info-item">
                    <text>地址</text>
                    <view class="section" @tap="getRegeoAddress">
                        <input placeholder="选择收货地址" disabled="disabled" name="street" :value="street" />
                        <text class="iconfont iconhtbArrowright02" style="width: auto"></text>
                    </view>
                </view>
                <view class="address-info-item">
                    <text>门牌号</text>
                    <input placeholder="例：5号楼203室" name="houseNumber" :value="houseNumber" />
                </view>
                <!-- <view class="address-info-item">
      <text>标签</text>
      <radio-group class="tag-radio-group" bindchange="tagRadioChange">
        <label wx:for="{{tagRadio}}" wx:key="index" class="tag-radio-group-label {{item.checked?'tag-active theme-bg':'tag-unchecked'}}">
          <radio class="none-radio" value="{{item.id}}" checked="{{item.checked}}"/>{{item.name}}
        </label>
      </radio-group>
    </view> -->
            </view>
            <view>
                <checkbox-group name="isDefault" @change="isDefaultChange">
                    <label class="setting-default">
                        <checkbox :checked="checkbox" :value="isDefault"></checkbox>
                        <text class="default-words">设为默认地址</text>
                    </label>
                </checkbox-group>
            </view>
            <view class="operation-view">
                <button class="save-view theme-bg" hover-class="hover-class-public" form-type="submit" style="margin: 0; width: 100%; padding: initial">保存</button>
            </view>
        </form>
    </view>
</template>

<script>
import https from '../../../utils/http';
import toastService from '../../../utils/toast.service';
import utils from '../../../utils/util';
import stringService from '../../../utils/string-service';
//获取应用实例
const app = getApp();
export default {
    data() {
        return {
            sexRadio: [
                {
                    id: 0,
                    name: '先生',
                    checked: true
                },
                {
                    id: 1,
                    name: '女士',
                    checked: false
                }
            ],

            tagRadio: [
                {
                    id: 0,
                    name: '家',
                    checked: true
                },
                {
                    id: 1,
                    name: '公司',
                    checked: false
                },
                {
                    id: 2,
                    name: '学校',
                    checked: false
                }
            ],

            sexKey: 0,
            region: [],
            isDefault: 0,
            tagKey: '',
            street: '',
            houseNumber: 0,
            checkbox: false
        };
    }
    /**
     * 生命周期函数--监听页面加载
     */,
    onLoad: function (options) {},
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
        bindPhoneBlur(e) {
            let phone = e.detail.value;
            let isMobile = utils.verifyPhone(phone);
            if (!isMobile) {
                toastService.showToast('手机号不正确');
                return;
            }
        },

        sexRadioChange(e) {
            let checkedValue = e.detail.value;
            for (let key in this.sexRadio) {
                this.sexRadio[key].checked = false;
            }
            this.sexRadio[checkedValue].checked = true;
            this.setData({
                sexKey: checkedValue,
                sexRadio: this.sexRadio
            });
        },

        tagRadioChange(e) {
            let checkedValue = e.detail.value;
            for (let key in this.tagRadio) {
                this.tagRadio[key].checked = false;
            }
            this.tagRadio[checkedValue].checked = true;
            this.setData({
                tagKey: checkedValue,
                tagRadio: this.tagRadio
            });
        },

        bindRegionChange: function (e) {
            //console.log('picker发送选择改变，携带值为', e.detail.value)
            this.setData({
                region: e.detail.value
            });
        },

        isDefaultChange(e) {
            this.setData({
                isDefault: e.detail.value.length == 0 ? 0 : 1
            });
        },

        formSubmit(e) {
            //console.log(e)
            var res = e.detail.value;
            if (stringService.isEmpty(res.realname)) {
                toastService.showToast('联系人姓名为必填');
                return;
            }
            if (stringService.isEmpty(res.phone)) {
                toastService.showToast('手机号为必填');
                return;
            }
            let isMobile = utils.verifyPhone(res.phone);
            if (!isMobile) {
                toastService.showToast('手机号不正确');
                return;
            }
            if (stringService.isEmpty(res.street)) {
                toastService.showToast('门牌号为必填');
                return;
            }
            console.log(this);
            res.province = this.region[0];
            res.city = this.region[1];
            res.area = this.region[2];
            res.isDefault = this.isDefault;
            res.longitude = this.longitude;
            res.latitude = this.latitude;
            https.request('/api-user/rest/member/deliveryAddress/insert', res).then((result) => {
                if (result.success) {
                    toastService.showSuccess(result.message, true, 1000);
                    var pages = getCurrentPages();
                    //var currPage = pages[pages.length - 1];   //当前页面
                    var prevPage = pages[pages.length - 2]; //上一个页面
                    //直接调用上一个页面的 setData() 方法，把数据存到上一个页面中去
                    /* var prevData = prevPage.data.addressList;
        res.id=result.data;
        prevData.push(res);
        prevPage.setData({
          addressList: prevData,
        }) */
                    prevPage.data.addressList = [];
                    prevPage.getDeliveryAddressList();
                    uni.navigateBack(1);
                } else {
                    showToast.showToast(result.message);
                }
            });
        },

        getRegeoAddress(e) {
            uni.navigateTo({
                url: '../../address/search/search'
            });
        },

        formReset() {
            console.log('占位：函数 formReset 未声明');
        }
    }
};
</script>
<style>
@import './insert.css';
</style>
