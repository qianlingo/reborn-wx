<template>
    <view :class="theme_view">
        <component-popup :propShow="popup_status" propPosition="bottom" @onclose="popup_close_event">
            <view class="share-popup bg-white">
                <view class="close fr oh">
                    <view class="fr" @tap.stop="popup_close_event">
                        <iconfont name="icon-huiyuan-guanbi" size="28rpx" color="#999"></iconfont>
                    </view>
                </view>
                <view class="share-popup-content">
                    <!-- #ifdef H5 -->
                    <view class="share-items oh cp" @tap="share_h5_event">
                        <image :src="common_static_url + 'share-user-icon.png'" mode="scaleToFill"></image>
                        <text class="cr-grey text-size-xs single-text">点击复制地址分享给好友、群聊</text>
                    </view>
                    <!-- #endif -->
                    <!-- #ifdef MP-ALIPAY -->
                    <view class="share-items oh cp" @tap="share_base_event">
                        <image :src="common_static_url + 'share-user-icon.png'" mode="scaleToFill"></image>
                        <text class="cr-grey text-size-xs single-text">一键分享给好友、群聊</text>
                    </view>
                    <!-- #endif -->
                    <!-- #ifdef MP-WEIXIN || MP-BAIDU || MP-QQ || MP-TOUTIAO || MP-KUAISHOU -->
                    <view class="share-items oh cp">
                        <button class="dis-block br-0 ht-auto" type="default" size="mini" open-type="share" hover-class="none" @tap="popup_close_event">
                            <image :src="common_static_url + 'share-user-icon.png'" mode="scaleToFill"></image>
                            <text class="cr-grey text-size-xs single-text">一键分享给好友、群聊</text>
                        </button>
                    </view>
                    <!-- #endif -->
                    <view v-if="is_goods_poster == 1 && (goods_id || 0) != 0" class="share-items oh cp" @tap="poster_event">
                        <image :src="common_static_url + 'share-friend-icon.png'" mode="scaleToFill"></image>
                        <text class="cr-grey text-size-xs single-text">生成海报，分享到朋友圈、好友及群聊</text>
                    </view>
                </view>
            </view>
        </component-popup>

        <!-- 用户基础 -->
        <component-user-base ref="user_base"></component-user-base>
    </view>
</template>
<script>
    const app = getApp();
    var common_static_url = app.globalData.get_static_url('common');
    import componentPopup from '../../components/popup/popup';
    import componentUserBase from '../../components/user-base/user-base';
    export default {
        data() {
            return {
                theme_view: app.globalData.get_theme_value_view(),
                common_static_url: common_static_url,
                popup_status: false,
                is_goods_poster: 0,
                goods_id: 0,
            };
        },

        components: {
            componentPopup,
            componentUserBase,
        },

        created: function () {},

        methods: {
            // 初始配置
            init(config = {}) {
                if (!app.globalData.is_single_page_check()) {
                    return false;
                }
                this.setData({
                    popup_status: config.status == undefined ? true : config.status,
                    is_goods_poster: config.is_goods_poster || 0,
                    goods_id: config.goods_id || 0,
                });

                // 用户头像和昵称设置提示
                if ((this.$refs.user_base || null) != null) {
                    this.$refs.user_base.init('share');
                }
            },

            // 弹层关闭
            popup_close_event(e) {
                this.setData({
                    popup_status: false,
                });
            },

            // h5分享
            share_h5_event() {
                app.globalData.text_copy_event(app.globalData.get_page_url());
            },

            // 基础分享事件
            share_base_event() {
                this.setData({
                    popup_status: false,
                });
                uni.pageScrollTo({
                    scrollTop: 0,
                    duration: 300,
                    complete: (res) => {
                        setTimeout(function () {
                            uni.showShareMenu();
                        }, 500);
                    },
                });
            },

            // 商品海报分享
            poster_event() {
                var user = app.globalData.get_user_info(this, 'poster_event');
                if (user != false) {
                    // 用户未绑定手机则转到登录页面
                    if (app.globalData.user_is_need_login(user)) {
                        uni.navigateTo({
                            url: '/pages/login/login?event_callback=poster_event',
                        });
                        return false;
                    } else {
                        uni.showLoading({
                            title: '生成中...',
                        });
                        uni.request({
                            url: app.globalData.get_request_url('goodsposter', 'distribution', 'distribution'),
                            method: 'POST',
                            data: { goods_id: this.goods_id },
                            dataType: 'json',
                            success: (res) => {
                                uni.hideLoading();

                                if (res.data.code == 0) {
                                    uni.previewImage({
                                        current: res.data.data,
                                        urls: [res.data.data],
                                    });
                                } else {
                                    if (app.globalData.is_login_check(res.data, this, 'poster_event')) {
                                        app.globalData.showToast(res.data.msg);
                                    }
                                }
                            },
                            fail: () => {
                                uni.hideLoading();
                                app.globalData.showToast('网络开小差了哦~');
                            },
                        });
                    }
                }
            },
        },
    };
</script>
<style>
    .share-popup {
        padding: 20rpx 10rpx 0 10rpx;
        position: relative;
    }
    .share-popup .close {
        position: absolute;
        top: 20rpx;
        right: 20rpx;
        z-index: 2;
    }
    .share-popup-content {
        padding: 0 20rpx;
        text-align: left;
    }
    .share-popup-content .share-items {
        padding: 30rpx 0;
        min-height: 85rpx;
    }
    .share-popup-content .share-items:not(:first-child) {
        border-top: 1px solid #f0f0f0;
    }
    .share-popup-content .share-items button {
        background: transparent;
        padding: 0;
        width: 100%;
        text-align: left;
        margin: 0;
    }
    .share-popup-content .share-items image {
        width: 80rpx;
        height: 80rpx;
        vertical-align: middle;
        margin-right: 20rpx;
    }
    .share-popup-content .share-items .single-text {
        width: calc(100% - 100rpx);
        line-height: 85rpx;
    }
</style>
