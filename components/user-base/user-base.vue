<template>
    <view :class="theme_view">
        <component-popup :propShow="popup_status" propPosition="bottom" @onclose="popup_close_event">
            <view :class="'user-base-popup bg-white ' + (propIsGrayscale ? 'grayscale' : '')">
                <view class="close fr oh">
                    <view class="fr" @tap.stop="popup_close_event">
                        <iconfont name="icon-huiyuan-guanbi" size="24rpx" color="#999"></iconfont>
                    </view>
                </view>
                <form @submit="form_submit" class="form-container">
                    <view class="padding-top-sm">
                        <view class="text-size-md spacing-mb">获取您的昵称、头像</view>
                        <view class="padding-bottom-main cr-grey-9 br-b">获取用户头像、昵称完善个人资料，主要用于向用户提供 具有辨识度的用户中心界面</view>
                        <view v-if="(user.avatar || null) == null" class="form-gorup oh flex-row align-c br-b">
                            <view class="form-gorup-title text-size-md">头像<text class="form-group-tips-must">*</text></view>
                            <button class="bg-white br-0 padding-0 margin-left-xxl flex-row jc-sb align-c flex-1" hover-class="none" open-type="chooseAvatar" @chooseavatar="choose_avatar_event" @tap="choose_avatar_event">
                                <image :src="user_avatar || default_avatar" mode="widthFix" class="circle br user-base-avatar"></image>
                                <iconfont name="icon-qiandao-jiantou2" size="24rpx" color="#ccc"></iconfont>
                            </button>
                        </view>
                        <view v-if="(user.nickname || null) == null" class="form-gorup oh flex-row align-c br-b">
                            <view class="form-gorup-title text-size-md">昵称<text class="form-group-tips-must">*</text></view>
                            <view class="user-nickname-container padding-left-xxl">
                                <input :type="application_client_type == 'weixin' ? 'nickname' : 'text'" name="nickname" maxlength="16" placeholder-class="cr-grey-c" class="cr-base" placeholder="输入昵称1~16个字符" @input="on_input_name" />
                            </view>
                        </view>

                        <view class="tc padding-top-xxl">
                            <button class="sub-btn cr-white text-size" :class="form_submit_disabled_status ? 'bg-grey-d br-grey-d' : 'bg-main br-main'" type="default" form-type="submit" hover-class="none">保存</button>
                        </view>
                    </view>
                </form>
            </view>
        </component-popup>
    </view>
</template>
<script>
    const app = getApp();
    import componentPopup from '../../components/popup/popup';
    export default {
        data() {
            return {
                theme_view: app.globalData.get_theme_value_view(),
                application_client_type: app.globalData.application_client_type(),
                default_avatar: app.globalData.data.default_user_head_src,
                cache_key: app.globalData.data.cache_user_base_personal_interval_time_key,
                popup_status: false,
                user: {},
                user_avatar: '',
                nickname: '',
                pages: [],
                client: [],
                interval_time: 0,
                form_submit_disabled_status: true,
                testValue: '',
            };
        },

        components: {
            componentPopup,
        },
        props: {
            propIsGrayscale: {
                type: Boolean,
                default: false,
            },
        },

        created: function () {
            // 初始化配置
            this.init_config();
        },

        methods: {
            // 初始化配置
            init_config(status) {
                if ((status || false) == true) {
                    this.setData({
                        pages: app.globalData.get_config('config.common_app_user_base_popup_pages', []),
                        client: app.globalData.get_config('config.common_app_user_base_popup_client', []),
                        interval_time: parseInt(app.globalData.get_config('config.common_app_user_base_popup_interval_time', 1800)),
                    });
                } else {
                    app.globalData.is_config(this, 'init_config');
                }
            },

            // 初始配置
            init(type = '') {
                // 初始化配置
                this.init_config(true);

                // 是否需要展示弹窗提示
                if (!this.popup_status && this.pages.indexOf(type) != -1 && this.client.indexOf(this.application_client_type) != -1) {
                    // 当前缓存用户
                    var user = app.globalData.get_user_cache_info() || null;
                    // 默认昵称则赋空值
                    var arr = ['支付宝用户', '百度用户', '头条用户', 'QQ用户', '快手用户', '微信用户', 'WeChat User', 'Usuarios de Wechat'];
                    if (user != null && (user.nickname || null) != null && arr.indexOf(user.nickname) != -1) {
                        user.nickname = '';
                    }
                    // 头像是默认则置为空
                    if (user != null && (user.avatar || null) != null && user.avatar.indexOf('default-user-avatar') != -1) {
                        user.avatar = '';
                    }
                    // 状态
                    var status = user == null ? false : ((user.avatar || null) == null || (user.nickname || null) == null ? true : false);
                    // 间隔时间
                    var cache_time = parseInt(uni.getStorageSync(this.cache_key) || 0);
                    var current_time = Date.parse(new Date()) / 1000;
                    if (status && !this.popup_status && cache_time > 0 && current_time < cache_time + parseInt(this.interval_time)) {
                        status = false;
                    }

                    // 1秒后再提示用户填写信息
                    var self = this;
                    clearTimeout(this.timer);
                    this.timer = setTimeout(function () {
                        self.setData({
                            popup_status: status,
                            user: user,
                        });
                    }, 500);
                }
            },

            // 弹层关闭
            popup_close_event(e) {
                this.setData({
                    popup_status: false,
                });
                uni.setStorageSync(this.cache_key, Date.parse(new Date()) / 1000);
            },

            // 头像事件
            choose_avatar_event(e) {
                var self = this;
                if (this.application_client_type == 'weixin') {
                    self.upload_handle(e.detail.avatarUrl);
                } else {
                    uni.chooseImage({
                        count: 1,
                        success(res) {
                            if (res.tempFilePaths.length > 0) {
                                self.upload_handle(res.tempFilePaths[0]);
                            }
                        },
                    });
                }
            },

            // 上传处理
            upload_handle(image) {
                var self = this;
                uni.uploadFile({
                    url: app.globalData.get_request_url('useravatarupload', 'personal'),
                    filePath: image,
                    name: 'file',
                    formData: {},
                    success: function (res) {
                        if (res.statusCode == 200) {
                            var data = typeof res.data == 'object' ? res.data : JSON.parse(res.data);
                            if (data.code == 0) {
                                self.setData({
                                    user_avatar: data.data,
                                    form_submit_disabled_status: ((self.user.nickname || null) == null && (self.nickname || null) == null)
                                });
                            } else {
                                app.globalData.showToast(data.msg);
                            }
                        }
                    },
                });
            },

            // 数据提交
            form_submit(e) {
                // 表单数据
                var form_data = e.detail.value;
                // 头像
                form_data['avatar'] = this.user_avatar || '';

                // 验证数据项
                var validation = [];
                if ((this.user.avatar || null) == null) {
                    validation.push({
                        fields: 'avatar',
                        msg: '请上传头像',
                    });
                } else {
                    delete form_data['avatar'];
                }
                if ((this.user.nickname || null) == null) {
                    validation.push({
                        fields: 'nickname',
                        msg: '请填写昵称',
                    });
                } else {
                    delete form_data['nickname'];
                }
                if (app.globalData.fields_check(e.detail.value, validation)) {
                    // 数据保存
                    uni.showLoading({
                        title: '处理中...',
                    });
                    uni.request({
                        url: app.globalData.get_request_url('save', 'personal'),
                        method: 'POST',
                        data: form_data,
                        dataType: 'json',
                        success: (res) => {
                            uni.hideLoading();
                            if (res.data.code == 0) {
                                uni.setStorageSync(app.globalData.data.cache_user_info_key, res.data.data);
                                app.globalData.showToast(res.data.msg, 'success');
                                this.setData({
                                    popup_status: false,
                                });
                            } else {
                                if (app.globalData.is_login_check(res.data)) {
                                    app.globalData.showToast(res.data.msg);
                                } else {
                                    app.globalData.showToast('提交失败，请重试！');
                                }
                            }
                        },
                        fail: () => {
                            uni.hideLoading();
                            app.globalData.showToast('网络开小差了哦~');
                        },
                    });
                }
            },
            // 输入框值监听
            on_input_name(event) {
                if (event.detail.value != '') {
                    this.setData({
                        nickname: event.detail.value,
                        form_submit_disabled_status: ((this.user.avatar || null) == null && (this.user_avatar || null) == null)
                    });
                } else {
                    this.setData({
                        form_submit_disabled_status: true,
                    });
                }
            },
        },
    };
</script>
<style scoped>
    .user-base-popup {
        padding: 36rpx;
        position: relative;
    }
    .user-base-popup .close {
        position: absolute;
        top: 36rpx;
        right: 36rpx;
        z-index: 2;
    }
    .user-base-avatar {
        width: 80rpx;
        height: 80rpx !important;
    }
    .user-base-popup .form-gorup-title {
        line-height: 70rpx;
        margin-bottom: 0;
    }
    .user-base-popup .user-nickname-container {
        width: calc(100% - 120rpx);
    }
    .form-container .form-gorup {
        padding: 24rpx 0;
        border-radius: 0;
    }
    .form-container .form-gorup-title {
        font-weight: 400;
    }
    .sub-btn {
        width: 336rpx;
        height: 84rpx;
        line-height: 84rpx;
        padding: 0;
        display: inline-flex;
        align-items: center;
        justify-content: center;
        border-radius: 12rpx;
    }
</style>
