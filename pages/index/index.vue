<template>
	<view class="wrap">
		<view class="title">物联网智能家居</view>

		<!-- 顶部信息卡片区域 -->
		<view class="top-cards">
			<view class="info-card">
				<view class="info-item">温度 {{temp}} ℃</view>
				<view class="info-item">天气 {{weather}}</view>
			</view>
			<view class="info-card">
				<view class="info-item">江苏 无锡</view>
				<view class="info-item">云量 {{cloudCover}}%</view>
				<view class="info-item">风速 {{windSpeed}} m/s</view>
			</view>
		</view>

		<!-- 传感器数据区域 -->
		<view class="sensor-area">
			<view class="sensor-title">蠡湖大道1800号江南大学DHT11传感器数据</view>
			<view class="sensor-cards">
				<view class="dev-cart">
					<view class="">
						<view class="dev-name">温度</view>
						<image class="dev-logo" src="../../static/temp.png" mode=""></image>
					</view>
					<view class="dev-data">{{temp}} ℃</view>
				</view>
				<view class="dev-cart">
					<view class="">
						<view class="dev-name">湿度</view>
						<image class="dev-logo" src="../../static/humi.png" mode=""></image>
					</view>
					<view class="dev-data">{{humi}} %</view>
				</view>
				<view class="dev-cart">
					<view class="">
						<view class="dev-name">台灯</view>
						<image class="dev-logo" src="../../static/lamp.png" mode=""></image>
					</view>
					<switch :checked="led" @change="onLedSwitch" color="#2b9939"/>
				</view>
			</view>
		</view>

		<!-- 折线图区域 -->
		<view class="chart-area">
			<view class="chart-title">传感器湿度变化折线图</view>
			<view class="chart-container">
				<svg width="100%" height="100%" viewBox="0 0 500 200">
					<!-- 修复points属性语法 -->
					<polyline 
						:points="getPolylinePoints()"
						fill="none" 
						stroke="#2b9939" 
						stroke-width="2"
					/>
					<template v-for="(val, i) in tempData">
						<circle 
						:cx="50 + i * pointSpacing" 
						:cy="getYPosition(val)" 
						r="4" 
						fill="#2b9939" 
						:key="i"
						/>
					</template>
					<!-- 添加坐标轴 -->
					<line x1="40" y1="50" x2="40" y2="200" stroke="#ccc" stroke-width="1" />
      				<line x1="40" y1="200" x2="490" y2="200" stroke="#ccc" stroke-width="1" />
					</svg>
			</view>
		</view>

		<!-- 页面切换 -->
		<view class="page-switch">
			<view class="page-btn active">页面一</view>
			<view class="page-btn" @click="goToPageTwo">页面二</view>
			<view class="page-btn" @click="goToPageThree">页面三</view>
		</view>
	</view>
</template>

<script>
	const {
		createCommonToken
	} = require('@/key.js')
	export default {
		data() {
			return {
				temp: '',
				humi: '',
				led: true,
				token: '',
				weather: '晴',
				cloudCover: '30',
				windSpeed: '2.5',
				pointSpacing: 70,
				tempData: [25, 26, 25.5, 27, 26.5, 27.5, 28] // 添加默认数据
			}
		},
		onLoad() {
			const params = {
				author_key: 'xV/Be37vvPZ10fadeflzH4j6hnWEbkfhIxAjeXGN8D1vBBNF3bYIHKdyvUDjel42',
				version: '2022-05-01',
				user_id: '456966',
			}
			this.token = createCommonToken(params);
		},
		onShow() {
			this.fetchDevData();
			setInterval(()=>{
				this.fetchDevData();
			},3000)
		},
		methods: {
			// 切换到页面二
			goToPageTwo() {
				uni.navigateTo({
					url: '/pages/page2/page2'
				});
			},
			// 切换到页面三
			goToPageThree() {
				uni.navigateTo({
					url: '/pages/page3/page3'
				});
			},
			getPolylinePoints() {
				return this.tempData.map((val, i) => {
				return `${50 + i * this.pointSpacing},${this.getYPosition(val)}`
				}).join(' ')
			},
			getYPosition(value) {
				if (this.tempData.length === 0) return 100;
				// 动态计算Y坐标，确保在视图范围内
				const maxVal = Math.max(...this.tempData, 70) 
				const minVal = Math.min(...this.tempData, 30) // 确保最小值
				const range = maxVal - minVal || 10 // 防止除零
				
				// 映射到150-50的范围内（顶部留50px，底部留50px）
				return 50 + 150 * (1 - (value - minVal) / range)
			},
			fetchDevData() {
				uni.request({
					url: 'https://iot-api.heclouds.com/thingmodel/query-device-property',
					method: 'GET',
					data: {
						product_id: 'CRU6Hr7I1v',
						device_name: 'iot1'
					},
					header: {
						'authorization': this.token
					},
					success: (res) => {
						console.log('API response:', res.data);
						// 验证数据结构
						if (res.data && res.data.data && res.data.data.length >= 3) {
							this.temp = res.data.data[2].value;
							this.humi = res.data.data[0].value;
							this.led = res.data.data[1].value === 'true';

							// 确保温度是有效数字
							const tempValue = Number(this.humi);
							if (!isNaN(tempValue)) {
								this.tempData.push(tempValue);
								if(this.tempData.length > 7) {
									this.tempData.shift();
								}
							} else {
								console.error('Invalid temperature value:', this.temp);
							}
						} else {
							console.error('Invalid API response structure:', res.data);
						}
					},
					fail: (err) => {
						console.error('API request failed:', err);
					}
				});
			},
			onLedSwitch(event) {
				console.log(event.detail.value);
				let value = event.detail.value;
				uni.request({
					url: 'https://iot-api.heclouds.com/thingmodel/set-device-property', //仅为示例，并非真实接口地址。
					method: 'POST',
					data: {
						product_id: 'CRU6Hr7I1v',
						device_name: 'iot1',
						params: {
							"led": value
						}
					},
					header: {
						'authorization': this.token //自定义请求头信息
					},
					success: (res) => {
          if (res.data.errno === 0) {
            this.led = !this.led;
            // 增加开关次数计数
            if (!getApp().globalData.switchCount) getApp().globalData.switchCount = 0;
            getApp().globalData.switchCount++;
            uni.showToast({ title: '操作成功' });
          }
        }
				});
			}

		}
	}
</script>

<style>
.wrap {
	 padding: 30rpx;
	 font-family: PingFang SC, sans-serif;
	}

	.title {
	 font-size: 40rpx;
	 text-align: center;
	 margin: 30rpx 0;
	 color: #333;
	 font-weight: bold;
	}

	.top-cards {
	 display: flex;
	 justify-content: space-between;
	 margin-bottom: 40rpx;
	}

	.info-card {
	 width: 48%;
	 background-color: #f5f7fa;
	 border-radius: 20rpx;
	 padding: 30rpx;
	 box-shadow: 0 5rpx 15rpx rgba(223, 99, 99, 0.1);
	}

	.info-item {
	 font-size: 28rpx;
	 margin: 15rpx 0;
	 color: #333;
	}

	.sensor-area {
	 background-color: #fff;
	 border-radius: 20rpx;
	 padding: 20rpx;
	 margin-bottom: 40rpx;
	 box-shadow: 0 5rpx 15rpx rgba(0,0,0,0.1);
	}

	.sensor-title {
	 font-size: 28rpx;
	 margin-bottom: 20rpx;
	 color: #666;
	}

	.sensor-cards {
	 display: flex;
	 justify-content: space-between;
	 flex-wrap: wrap;
	}

	.dev-area {
		display: flex;
		justify-content: space-between;
		flex-wrap: wrap;
	}

	.dev-cart {
		height: 180rpx;
		width: 30%;
		border-radius: 15rpx;
		margin-top: 20rpx;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		background-color: #f9f9f9;
		box-shadow: 0 3rpx 10rpx rgba(0,0,0,0.05);
	}

	.dev-name {
		font-size: 24rpx;
		text-align: center;
		color: #666;
		margin-bottom: 10rpx;
	}

	.dev-logo {
		width: 60rpx;
		height: 60rpx;
	}

	.dev-data {
		font-size: 36rpx;
		color: #333;
		margin-top: 10rpx;
	}

	.chart-area {
		background-color: #fff;
		border-radius: 20rpx;
		padding: 20rpx;
		margin-bottom: 40rpx;
		box-shadow: 0 5rpx 15rpx rgba(0,0,0,0.1);
	}

	.chart-title {
		font-size: 28rpx;
		margin-bottom: 20rpx;
		color: #666;
	}

	.chart-container {
		width: 100%;
		height: 300rpx;
		background-color: #f9f9f9;
		border-radius: 10rpx;
		display: flex;
		justify-content: center;
		align-items: center;
		color: #999;
	}

	.page-switch {
		display: flex;
		justify-content: center;
		margin-top: 50rpx;
	}

	.page-btn {
		width: 180rpx; /* 增加宽度确保文字显示完整 */
		height: 60rpx;
		background-color: #f5f5f5;
		border-radius: 30rpx;
		display: flex;
		justify-content: center;
		align-items: center;
		margin: 0 20rpx;
		font-size: 26rpx; /* 微调字体大小 */
		color: #666;
		box-shadow: 0 2rpx 8rpx rgba(0,0,0,0.1); /* 添加阴影优化边框质感 */
		border: none; /* 移除默认边框 */
	}

	.page-btn.active {
		background-color: #2b9939;
		color: white;
	}

</style>