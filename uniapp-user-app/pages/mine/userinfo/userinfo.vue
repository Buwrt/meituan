<template>
    <view>
        <view>
            <view class="userinfo-item">
                <text class="title-text">头像</text>
                <!-- <image class="userinfo-avatar" src="{{userInfo.avatarUrl}}" mode="cover"></image> -->
                <image class="userinfo-avatar" :src="data.headImg ? data.headImg : '../../../assets/images/user-head.png'" mode="cover"></image>
            </view>
            <view class="userinfo-item">
                <text class="title-text">用户名</text>
                <text class="userinfo-text">{{ data.username }}</text>
            </view>
            <view class="userinfo-item">
                <text class="title-text">绑定手机</text>
                <text class="userinfo-text">{{ data.mobile }}</text>
            </view>
            <view class="userinfo-item" @tap="openRealName">
                <text class="title-text">真实姓名</text>
                <view class="input-image">
                    <text :class="data.realName ? 'userinfo-text' : 'placeholder-text'">{{ data.realName ? data.realName : '请输入真实姓名' }}</text>
                    <text class="iconfont iconhtbArrowright02"></text>
                </view>
            </view>
            <view class="userinfo-item" @tap="openConfirm">
                <text class="title-text">性别</text>
                <view class="input-image">
                    <text class="userinfo-text">{{ data.sex == 1 ? '男' : data.sex == 2 ? '女' : '保密' }}</text>
                    <text class="iconfont iconhtbArrowright02"></text>
                </view>
            </view>
            <view class="userinfo-item" @tap="openEmail">
                <text class="title-text">邮箱</text>

                <view class="input-image">
                    <!-- <input class="userinfo-text input-email" placeholder="请输入邮箱地址" value="{{data.email}}" bindblur="blurEmail"></input> -->
                    <!-- <input placeholder-class="placeholder-email" class="userinfo-text" placeholder="请输入邮箱地址" value="{{data.email}}" disabled="disabled"></input> -->
                    <text :class="data.email ? 'userinfo-text' : 'placeholder-text'">{{ data.email ? data.email : '请输入邮箱地址' }}</text>
                    <text class="iconfont iconhtbArrowright02"></text>
                </view>
            </view>

            <view class="userinfo-item" @tap="parseEventDynamicCode($event, data.isBindWx ? 'bindWx' : '')" :data-mobile="data.mobile" :data-isBindWx="data.isBindWx">
                <text class="title-text">绑定微信</text>
                <view class="bind-wx">
                    <text class="userinfo-text">{{ data.isBindWx ? '已绑定' : '未绑定' }}</text>
                    <text class="iconfont iconhtbArrowright02"></text>
                </view>
            </view>
        </view>
        <mp-dialog title="选择性别" :show="dialogShow" :buttons="buttons" :mask-closable="maskClosable" @touchmove.native.stop.prevent="trueFun" @buttontap="tapdialogButton">
            <radio-group @change="radioChange" class="sex-radio-group">
                <label :data-radioIndex="index" class="radio-group-label" v-for="(item, index) in modeList" :key="index">
                    {{ item.name }}

                    <radio :value="item.value" class="radio-group-label-radio" :checked="item.checked" />
                </label>
            </radio-group>
        </mp-dialog>
        <mp-dialog
            title="编辑邮箱"
            :show="dialogShowEmail"
            :buttons="buttons"
            :mask-closable="maskClosable"
            @touchmove.native.stop.prevent="trueFun"
            @buttontap="tapdialogEmailButton"
        >
            <input class="userinfo-text input-email" placeholder="请输入邮箱地址" :value="data.email" @input="bindInputEmail" />
        </mp-dialog>
        <mp-dialog
            title="确认真实姓名"
            :show="dialogShowRealName"
            :buttons="buttons"
            :mask-closable="maskClosable"
            @touchmove.native.stop.prevent="trueFun"
            @buttontap="tapdialogRealNameButton"
        >
            <view class="title-tip">请于身份证上保持一致，信息有误影响奖励提现</view>
            <input class="userinfo-text input-email" placeholder="请输入真实姓名" :value="data.realName" @input="bindInputRealName" />
        </mp-dialog>
        <view class="out-login-btn theme-bg" hover-class="hover-class-public" @tap="bindOutLogTap">退出登录</view>
    </view>
</template>

<script>
import mpDialog from '../../../components/dialog/dialog';
import GlobalConfig from '../../../utils/global-config';
import https from '../../../utils/http';
import authService from '../../../utils/auth';
import toastService from '../../../utils/toast.service';
import utils from '../../../utils/util';
//获取应用实例
const app = getApp();
export default {
    components: {
        mpDialog
    },
    data() {
        return {
            buttons: [
                {
                    text: '取消'
                },
                {
                    text: '确定'
                }
            ],

            userInfo: '',
            hasUserInfo: false,

            data: {
                isBindWx: false,
                sex: '',
                email: '',
                realName: '',
                headImg: false,
                username: '',
                mobile: ''
            },

            dialogShowEmail: '',
            email: '',
            dialogShowRealName: '',
            realName: '',
            dialogShow: '',
            maskClosable: '',
            modeList: [],
            sex: ''
        };
    },
    /**
     * 生命周期函数--监听页面加载
     */
    onLoad: function (options) {
        if (app.globalData.userInfo) {
            this.setData({
                userInfo: app.globalData.userInfo,
                hasUserInfo: true
            });
        } else if (this.canIUse) {
            // 由于 getUserInfo 是网络请求，可能会在 Page.onLoad 之后才返回
            // 所以此处加入 callback 以防止这种情况
            app.globalData.userInfoReadyCallback = (res) => {
                this.setData({
                    userInfo: res.userInfo,
                    hasUserInfo: true
                });
            };
        } else {
            // 在没有 open-type=getUserInfo 版本的兼容处理
            // wx.getUserInfo({
            //   success: res => {
            //     app.globalData.userInfo = res.userInfo
            //     this.setData({
            //       userInfo: res.userInfo,
            //       hasUserInfo: true
            //     })
            //   }
            // })
        }
    },
    /**
     * 生命周期函数--监听页面初次渲染完成
     */
    onReady: function () {},
    /**
     * 生命周期函数--监听页面显示
     */
    onShow: function () {
        this.getUserInfo();
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
        getUserInfo() {
            https.request('/api-user/rest/member/getLoginMemberInfo', {}).then((result) => {
                if (result.success) {
                    result.data.email = result.data.email ? result.data.email : '';
                    result.data.realName = result.data.realName ? result.data.realName : '';
                    this.setData({
                        data: result.data
                    });
                }
            });
        },

        bindWx(e) {
            var that = this;
            toastService.showModal(null, '确定解除与微信账号的关联关系吗？', function confirm() {
                https
                    .request('/api-user/rest/member/removeBindingWx', {
                        mobile: e.currentTarget.dataset.mobile,
                        isbindwx: e.currentTarget.dataset.isbindwx
                    })
                    .then((result) => {
                        if (result.success) {
                            toastService.showToast(result.message);
                            that.setData({
                                'data.isBindWx': false
                            });
                        }
                    });
            });
        },

        openEmail() {
            this.setData({
                dialogShowEmail: true,
                email: this.data.email
            });
        },

        openRealName() {
            this.setData({
                dialogShowRealName: true,
                realName: this.data.email
            });
        },

        openConfirm() {
            this.setData({
                dialogShow: true,
                maskClosable: false,
                modeList: [
                    {
                        name: '男',
                        value: 1,
                        checked: this.data.sex == 1 ? true : false
                    },
                    {
                        name: '女',
                        value: 2,
                        checked: this.data.sex == 2 ? true : false
                    }
                ]
            });
        },

        radioChange(e) {
            this.setData({
                sex: e.detail.value
            });
        },

        bindOutLogTap() {
            toastService.showModal(null, '确定退出登录吗？', function confirm() {
                https.request('/api-user/rest/member/logout', {}).then((result) => {
                    if (result.success) {
                        authService.deleteToken();
                        authService.deleteOpenId();
                        uni.navigateBack(1);
                    }
                });
            });
        },

        tapdialogButton: function (e) {
            if (e.detail.item.text == '确定') {
                https
                    .request('/api-user/rest/member/update', {
                        sex: this.sex
                    })
                    .then((result) => {
                        if (result.success) {
                            toastService.showToast(result.message);
                            this.setData({
                                'data.sex': this.sex
                            });
                        }
                    });
            }
            this.setData({
                dialogShow: false
            });
        },

        tapdialogEmailButton(e) {
            if (e.detail.item.text == '确定') {
                let email = this.email;
                if (email == this.data.email) {
                    toastService.showToast('请输入不同的邮箱号码');
                    return;
                }
                let isEmail = utils.verifyEmail(email);
                if (!isEmail) {
                    toastService.showToast('邮箱号码格式错误');
                    return;
                }
                https
                    .request('/api-user/rest/member/update', {
                        email: this.email
                    })
                    .then((result) => {
                        if (result.success) {
                            toastService.showToast(result.message);
                            this.setData({
                                'data.email': email
                            });
                        }
                    });
            }
            this.setData({
                dialogShowEmail: false
            });
        },

        tapdialogRealNameButton(e) {
            if (e.detail.item.text == '确定') {
                let realName = this.realName;
                if (!realName) {
                    toastService.showToast('请输入真实姓名');
                    return;
                }
                https
                    .request('/api-user/rest/member/update', {
                        realName: realName
                    })
                    .then((result) => {
                        if (result.success) {
                            toastService.showToast(result.message);
                            this.setData({
                                'data.realName': realName
                            });
                        }
                    });
            }
            this.setData({
                dialogShowRealName: false
            });
        },

        bindInputEmail(e) {
            this.setData({
                email: e.detail.value
            });
        },

        bindInputRealName(e) {
            this.setData({
                realName: e.detail.value
            });
        },

        trueFun() {
            console.log('占位：函数 true 未声明');
        }
    }
};
</script>
<style>
@import './userinfo.css';
</style>
