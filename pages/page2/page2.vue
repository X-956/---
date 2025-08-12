<template>
  <view class="wrap">
    <view class="title">你可以询问我任何问题</view>
    <!-- DeepSeek交互区域 -->
    <view class="chat-container">
		
      <view class="chat-history">
        <view class="message" v-for="(msg, i) in chatHistory" :key="i">
          <text class="user">用户：{{ msg.content }}</text>
          <text class="bot" v-if="msg.answer">DeepSeek：{{ msg.answer }}</text>
        </view>
      </view>

	  
      <view class="input-area">
        <input type="text" v-model="question" placeholder="输入问题..." class="question-input"/>
        <button class="send-btn" @click="sendToDeepSeek">发送</button>
      </view>
    </view>
    <view class="page-switch">
      <button class="page-btn" @click="goToPageOne">页面一</button>
      <button class="page-btn" @click="goToPageTwo">页面二</button>
      <button class="page-btn" @click="goToPageThree">页面三</button>
    </view>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        question: '',
        chatHistory: [],
        apiKey: 'sk-d2fbc00d25814133baf7bed8280b826e' 
      }
    },
    methods: {
      async sendToDeepSeek() {
		const trimmed = this.question.trim();
		if (!trimmed) return;
		
		// 保留用户问题
	    const message = { content: trimmed, answer: '' };
	    this.chatHistory.push(message);
	    this.question = '';

        try {
          const res = await uni.request({
            url: 'https://api.deepseek.com/v1/chat/completions', // DeepSeek API地址
            method: 'POST',
            header: {
              'Authorization': `Bearer ${this.apiKey}`,
              'Content-Type': 'application/json'
            },
            data: {
              model: 'deepseek-chat',
              messages: [
				  { role: 'system', content: '你是一个百科全书小助手。' },
				  { role: 'user', content: trimmed }
				  ]
            }
          });

          if (res.data?.choices?.[0]?.message?.content) {
            this.chatHistory[this.chatHistory.length - 1].answer = res.data.choices[0].message.content;
          }else{
			  this.chatHistory[this.chatHistory.length - 1].answer = '未收到有效回复，请稍后再试。';
		  }
        } catch (err) {
          console.error('DeepSeek API请求失败:', err);
          this.chatHistory[this.chatHistory.length - 1].answer = '抱歉，请求失败，请重试';
        }
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
    min-height: 100vh;
  }
  .chat-container {
    background-color: #f8f8f8;
    border-radius: 15rpx;
    padding: 30rpx;
    margin-bottom: 40rpx;
    box-shadow: 0 2rpx 10rpx rgba(0,0,0,0.05);
  }
  .chat-history {
    height: 600rpx;
    overflow-y: auto;
    margin-bottom: 30rpx;
  }
  .message {
    margin: 20rpx 0;
    padding: 20rpx;
    border-radius: 10rpx;
    max-width: 80%;
  }
  .message.user {
    background-color: #e3f2fd;
    margin-left: auto;
  }
  .message.bot {
    background-color: #f5f5f5;
    margin-right: auto;
  }
  .input-area {
    display: flex;
    gap: 20rpx;
  }
  .question-input {
    flex: 1;
    height: 60rpx;
    padding: 0 20rpx;
    border: 1rpx solid #ddd;
    border-radius: 30rpx;
    font-size: 26rpx;
  }
  .send-btn {
    width: 150rpx;
    height: 60rpx;
    background-color: #2b9939;
    color: white;
    border-radius: 30rpx;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 26rpx;
    border: none;
  }
  .title {
    font-size: 40rpx;
    text-align: center;
    margin: 30rpx 0;
    color: #333;
    font-weight: bold;
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
    color: #8f2222;
    border: none; /* 移除默认边框 */
  }
  .page-btn.active {
    background-color: #2b9939;
    color: white;
  }
</style>