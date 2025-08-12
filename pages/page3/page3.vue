<template>
  <view class="wrap">
    <view class="main-title">作者还活着吗？</view>
    <view class="status-container">
      <view class="status-title">lky's Status:</view>
      <view class="status-value">School</view>
      <view class="status-desc">正在学校，不可能在线。也可能是失联而无法更新。</view>
    </view>
    <view class="device-status">
      <view class="section-title">Device Status</view>
      <view class="device-info">lky's STM32: {{deviceStatus}}</view>
      <view class="update-time">最后更新: {{lastUpdate}}</view>
    </view>
    <view class="light-stats">
      <view class="stats-item">今日已开关灯 {{switchCount}} 次</view>
      <view class="control-tip">你可以通过 主页面 控制lky的台灯开关</view>
    </view>
    <view class="page-switch">
      <button class="page-btn" @click="goToPageOne">页面一</button>
      <button class="page-btn" @click="goToPageTwo">页面二</button>
      <button class="page-btn active" @click="goToPageThree">页面三</button>
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
        deviceStatus: '电源供电中，可以小程序遥控',
        lastUpdate: '2025-07-04 14:52:54',
        switchCount: getApp().globalData.switchCount || 5,
        token: ''
      }
    },
    onShow() {
      this.switchCount = getApp().globalData.switchCount || 0;
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
      this.fetchDeviceData();
      this.interval = setInterval(() => {
        this.fetchDeviceData();
      }, 3000);
    },
    onUnload() {
      clearInterval(this.interval);
    },
    methods: {
      fetchDeviceData() {
        // 已改为本地计数，移除云平台请求
        return; // 占位保持结构完整
      },
      updateDeviceStatus() {
        const lastTime = new Date(this.lastUpdate);
        const now = new Date();
        const diffMinutes = Math.floor((now - lastTime) / (1000 * 60));
        this.deviceStatus = diffMinutes > 30 ? '没在使用' : '电源供电中，可以小程序遥控';
      },
      goToPageOne() {
        uni.navigateTo({
          url: '/pages/index/index'
        })
      },
      goToPageTwo() {
        uni.navigateTo({
          url: '/pages/page2/page2'
        })
      },
      goToPageThree() {
        uni.navigateTo({
          url: '/pages/page3/page3'
        })
      }
    }
  }
</script>

<style>
  .wrap {
    padding: 30rpx;
    font-family: PingFang SC, sans-serif;
    background-color: #f9f9f9;
    min-height: 100vh;
  }
  .main-title {
    font-size: 36rpx;
    text-align: center;
    margin: 40rpx 0;
    color: #333;
    font-weight: bold;
  }
  .status-container {
    background-color: #fff;
    border-radius: 15rpx;
    padding: 30rpx;
    margin-bottom: 30rpx;
    box-shadow: 0 2rpx 10rpx rgba(0,0,0,0.05);
  }
  .status-title {
    font-size: 32rpx;
    color: #333;
    margin-bottom: 20rpx;
  }
  .status-value {
    font-size: 28rpx;
    color: #666;
    text-align: center;
    margin: 20rpx 0;
  }
  .status-desc {
    font-size: 24rpx;
    color: #999;
    text-align: center;
    padding-top: 20rpx;
    border-top: 1rpx solid #eee;
  }
  .device-status {
    background-color: #fff;
    border-radius: 15rpx;
    padding: 30rpx;
    margin-bottom: 30rpx;
    box-shadow: 0 2rpx 10rpx rgba(0,0,0,0.05);
  }
  .section-title {
    font-size: 28rpx;
    color: #666;
    margin-bottom: 20rpx;
    padding-bottom: 10rpx;
    border-bottom: 1rpx solid #eee;
  }
  .device-info {
    font-size: 26rpx;
    color: #333;
    margin: 20rpx 0;
  }
  .update-time {
    font-size: 22rpx;
    color: #999;
  }
  .light-stats {
    background-color: #fff;
    border-radius: 15rpx;
    padding: 30rpx;
    margin-bottom: 40rpx;
    box-shadow: 0 2rpx 10rpx rgba(0,0,0,0.05);
  }
  .stats-item {
    font-size: 26rpx;
    color: #333;
    text-align: center;
    margin: 30rpx 0;
  }
  .control-tip {
    font-size: 24rpx;
    color: #666;
    text-align: center;
    padding: 20rpx;
    background-color: #f5f5f5;
    border-radius: 10rpx;
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

    border: none; /* 移除默认边框 */
  }
  .page-btn.active {
    background-color: #2b9939;
    color: white;
  }
</style>