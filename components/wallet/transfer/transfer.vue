<!-- 转账 -->
<template>
    <view :class="theme_view">
        <!-- 列表 -->
        <view v-if="data_list.length > 0" class="data-list">
            <view v-for="(item, index) in data_list" :key="index" class="item padding-main border-radius-main oh bg-white spacing-mb">
                <view class="base oh br-b-dashed padding-bottom-main flex-row jc-sb align-c">
                    <text>转账时间</text>
                    <text class="cr-grey-9">{{ item.add_time }}</text>
                </view>
                <view class="content margin-top">
                    <navigator :url="'/pages/plugins/wallet/user-transfer-detail/user-transfer-detail?id=' + item.id" hover-class="none">
                        <block v-for="(fv, fi) in content_list" :key="fi">
                            <view class="single-text margin-top-sm transfer-item">
                                <text class="name cr-grey-9 margin-right-main dis-inline-block">{{ fv.name }}:</text>
                                <text class="fw-b">{{ item[fv.field] }}</text>
                                <text v-if="(fv.unit || null) != null" class="fw-b">{{ fv.unit }}</text>
                            </view>
                        </block>
                    </navigator>
                </view>
            </view>
        </view>
        <view v-else>
            <!-- 提示信息 -->
            <component-no-data :propStatus="data_list_loding_status"></component-no-data>
        </view>

        <!-- 结尾 -->
        <component-bottom-line :propStatus="data_bottom_line_status"></component-bottom-line>
    </view>
</template>
<script>
    const app = getApp();
    import componentNoData from '@/components/no-data/no-data';
    import componentBottomLine from '@/components/bottom-line/bottom-line';

    export default {
        props: {
            propPullDownRefresh: {
                type: Boolean,
                default: false,
            },
            propScrollLower: {
                type: Boolean,
                default: false,
            },
        },
        data() {
            return {
                theme_view: app.globalData.get_theme_value_view(),
                data_list: [],
                data_total: 0,
                data_page_total: 0,
                data_page: 1,
                data_list_loding_status: 1,
                data_bottom_line_status: false,
                data_is_loading: 0,
                params: null,
                content_list: [
                    { name: '收款人', field: 'transfer_no' },
                    { name: '转账金额', field: 'money', unit: '元' },
                    { name: '转账备注', field: 'note' },
                ],
            };
        },

        components: {
            componentNoData,
            componentBottomLine,
        },
        created() {
            this.init();
        },
        mounted() {
            // 分享菜单处理
            app.globalData.page_share_handle();
        },
        watch: {
            propPullDownRefresh(newVal, oldVal) {
                if (newVal !== oldVal) {
                    this.setData({
                        data_page: 1,
                    });
                    this.get_data_list(1);
                }
            },
            propScrollLower(newVal, oldVal) {
                if (newVal !== oldVal) {
                    this.get_data_list();
                }
            },
        },

        methods: {
            init() {
                var user = app.globalData.get_user_info(this, 'init');
                if (user != false) {
                    // 用户未绑定手机则转到登录页面
                    if (app.globalData.user_is_need_login(user)) {
                        uni.redirectTo({
                            url: '/pages/login/login?event_callback=init',
                        });
                        return false;
                    } else {
                        // 获取数据
                        this.get_data_list();
                    }
                } else {
                    this.setData({
                        data_list_loding_status: 0,
                        data_bottom_line_status: false,
                    });
                }
            },

            // 获取数据
            get_data_list(is_mandatory) {
                // 分页是否还有数据
                if ((is_mandatory || 0) == 0) {
                    if (this.data_bottom_line_status == true) {
                        uni.stopPullDownRefresh();
                        return false;
                    }
                }

                // 是否加载中
                if (this.data_is_loading == 1) {
                    return false;
                }
                this.setData({
                    data_is_loading: 1,
                    data_list_loding_status: 1,
                });

                // 加载loding
                uni.showLoading({
                    title: '加载中...',
                });

                // 获取数据
                uni.request({
                    url: app.globalData.get_request_url('index', 'transfer', 'wallet'),
                    method: 'POST',
                    data: {
                        page: this.data_page,
                    },
                    dataType: 'json',
                    success: (res) => {
                        uni.hideLoading();
                        uni.stopPullDownRefresh();
                        if (res.data.code == 0) {
                            if (res.data.data.data.length > 0) {
                                if (this.data_page <= 1) {
                                    var temp_data_list = res.data.data.data;
                                } else {
                                    var temp_data_list = this.data_list || [];
                                    var temp_data = res.data.data.data;
                                    for (var i in temp_data) {
                                        temp_data_list.push(temp_data[i]);
                                    }
                                }
                                this.setData({
                                    data_list: temp_data_list,
                                    data_total: res.data.data.total,
                                    data_page_total: res.data.data.page_total,
                                    data_list_loding_status: 3,
                                    data_page: this.data_page + 1,
                                    data_is_loading: 0,
                                });

                                // 是否还有数据
                                this.setData({
                                    data_bottom_line_status: this.data_page > 1 && this.data_page > this.data_page_total,
                                });
                            } else {
                                this.setData({
                                    data_list_loding_status: 0,
                                    data_list: [],
                                    data_bottom_line_status: false,
                                    data_is_loading: 0,
                                });
                            }
                        } else {
                            this.setData({
                                data_list_loding_status: 0,
                                data_is_loading: 0,
                            });
                            if (app.globalData.is_login_check(res.data, this, 'get_data_list')) {
                                app.globalData.showToast(res.data.msg);
                            }
                        }
                    },
                    fail: () => {
                        uni.hideLoading();
                        uni.stopPullDownRefresh();
                        this.setData({
                            data_list_loding_status: 2,
                            data_is_loading: 0,
                        });
                        app.globalData.showToast('网络开小差了哦~');
                    },
                });
            },
        },
    };
</script>
<style scoped>
    .nav-child .item {
        height: 60rpx;
        line-height: 60rpx;
        padding: 0 30rpx;
        min-width: 84rpx;
    }
    .transfer-item .name {
        min-width: 112rpx;
    }
</style>
