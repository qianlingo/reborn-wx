<template>
	<view>
		<view class="home-top-nav-content "></view>
		<!-- 静态banner -->
		<image class="banner" src="~@/static/images/index/banner.png" mode="aspectFit"></image>
		<!-- 轮播banner -->
		<swiper class="swiper-box" indicator-dots autoplay :interval="5000">
			<swiper-item>
				<image class="swiper-images" src="~@/static/images/index/banner2.png" mode="aspectFit"></image>
			</swiper-item><swiper-item>
				<image class="swiper-images" src="~@/static/images/index/banner2.png" mode="aspectFit"></image>
			</swiper-item><swiper-item>
				<image class="swiper-images" src="~@/static/images/index/banner2.png" mode="aspectFit"></image>
			</swiper-item>
		</swiper>
		<!-- 热词 -->
		<view class="hot-word">
			<view class="hot-word-item" style="--bg:#f0e7e0">/ NEW /</view>
			<view class="hot-word-item" style="--bg:#f5dfe1">/ HOT /</view>
			<view class="hot-word-item" style="--bg:#fee5e1">/ ALL /</view>
		</view>

		<!-- 人气热卖 -->
		<view style="margin-bottom: 32rpx;">
			<view class="module-title">
				<view class="line"></view>
				<view class="text">人气热卖</view>
				<view class="line"></view>
			</view>
			<view class="waterfall">
				<view class="main block"></view>
				<view class="aside block"></view>
			</view>
			<view class="waterfall">
				<view class="aside block" style="height: 210rpx;"></view>
				<view class="aside block" style="height: 210rpx;"></view>
				<view class="aside block" style="height: 210rpx;"></view>
			</view>
			<view class="waterfall">
				<view class="aside block"></view>
				<view class="main block"></view>
			</view>
		</view>

		<!-- 优惠卷 -->

		<!-- 精品推荐 -->
		<view style="margin-bottom: 32rpx;">
			<view class="module-title">
				<view class="line"></view>
				<view class="text">精品推荐</view>
				<view class="line"></view>
			</view>

			<component-goods-list :propData="{ style_type: 1, goods_list: goods_list, random: random_value }"
				:propLabel="plugins_label_data" :propCurrencySymbol="currency_symbol" :propIsCartParaCurve="true"
				propSource="index"></component-goods-list>

		</view>



	</view>
</template>

<script>
	const app = getApp();
	import componentGoodsList from '@/components/goods-list/goods-list';
	export default {
		components: {
			componentGoodsList,
		},
		data() {
			return {
				goods_list: [],
				goods_bottom_line_status: false,
				goods_is_loading: 0,
				goods_total: 0,
				goods_page_total: 0,
				goods_page: 1,
				currency_symbol: app.globalData.data.currency_symbol,
				plugins_label_data: null,
				// 增加随机数，避免无法监听数据列表内部数据更新
				random_value: 0,
			}
		},
		methods: {
			get_data_list(is_mandatory) {
				// 分页是否还有数据
				if ((is_mandatory || 0) == 0) {
					if (this.goods_bottom_line_status == true) {
						return false;
					}
				}

				// 是否加载中
				if (this.goods_is_loading == 1) {
					return false;
				}
				this.setData({
					goods_is_loading: 1,
				});

				// 获取数据
				uni.request({
					url: app.globalData.get_request_url('datalist', 'search'),
					method: 'POST',
					data: {
						page: this.goods_page,
					},
					dataType: 'json',
					success: (res) => {
						if (res.data.code == 0) {
							var data = res.data.data;
							if (data.data.length > 0) {
								if (this.goods_page <= 1) {
									var temp_data_list = data.data;
								} else {
									var temp_data_list = this.goods_list || [];
									var temp_data = data.data;
									for (var i in temp_data) {
										temp_data_list.push(temp_data[i]);
									}
								}
								this.setData({
									goods_list: temp_data_list,
									random_value: Math.random(),
									goods_total: data.total,
									goods_page_total: data.page_total,
									goods_page: this.goods_page + 1,
									goods_is_loading: 0,
								});

								// 是否还有数据
								this.setData({
									goods_bottom_line_status: this.goods_page > 1 && this.goods_page >
										this
										.goods_page_total,
								});
							} else {
								this.setData({
									goods_total: 0,
									goods_is_loading: 0,
								});
								if (this.goods_page <= 1) {
									this.setData({
										goods_list: [],
										goods_bottom_line_status: false,
									});
								}
							}
						} else {
							this.setData({
								goods_is_loading: 0,
							});
							app.globalData.showToast(res.data.msg);
						}
					},
					fail: () => {
						this.setData({
							goods_is_loading: 0,
						});
						app.globalData.showToast('网络开小差了哦~');
					},
				});
			},
		},
		onPageScroll() {
			this.get_data_list();
		},
		created: function() {
			// 猜你喜欢
			this.get_data_list(1);
		},

	}
</script>

<style>
	@import './index.css';
</style>