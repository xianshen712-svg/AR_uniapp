<template>
  <view class="ar-page">
    <yn-viewer :showBackBtn="true" @back="handleBack">
      <yn-ar
        id="hand-tracking"
        :width="renderWidth"
        :height="renderHeight"
        :style="'width:' + width + 'px;height:' + height + 'px'"
        yn_ar_mode="Hand"
        yn_camera_clear_color="0.1 0.1 0.1 1"
        @arReady="handleARReady"
        @handDetected="handleHandDetected"
        @handLost="handleHandLost"
      />
    </yn-viewer>

    <!-- 手部识别引导 -->
    <view class="hand-guide" v-if="!isHandDetected && arReady">
      <view class="hand-icon">✋</view>
      <view class="hand-text">请将手掌对准摄像头</view>
      <view class="hand-tips">支持识别：张开手掌、握拳、比耶等手势</view>
    </view>

    <!-- 识别成功 - 手势信息 -->
    <view class="gesture-info" v-if="isHandDetected">
      <view class="gesture-card">
        <view class="gesture-icon">{{ currentGesture.icon }}</view>
        <view class="gesture-name">{{ currentGesture.name }}</view>
        <view class="gesture-confidence">置信度: {{ currentGesture.confidence }}%</view>
      </view>
    </view>

    <!-- 底部信息面板 -->
    <view class="info-panel">
      <view class="info-title">👋 手部动作识别</view>
      <view class="info-desc">识别手部手势和动作</view>

      <view class="status-section">
        <view class="status-item" v-if="!arReady">
          <view class="status-dot loading"></view>
          <text class="status-label">正在启动AR...</text>
        </view>
        <view class="status-item" v-else-if="!isHandDetected">
          <view class="status-dot scanning"></view>
          <text class="status-label">正在识别手部...</text>
        </view>
        <view class="status-item success" v-else>
          <view class="status-dot detected"></view>
          <text class="status-label">已识别到手部</text>
        </view>
      </view>

      <view class="gesture-list">
        <view class="gesture-title">支持的手势:</view>
        <view class="gesture-items">
          <view class="gesture-tag" v-for="(gesture, index) in supportedGestures" :key="index" :class="{ active: currentGesture.name === gesture.name }">
            {{ gesture.icon }} {{ gesture.name }}
          </view>
        </view>
      </view>

      <view class="info-tips">
        <text class="tip-icon">💡</text>
        <text class="tip-text">{{ isHandDetected ? '移动手部查看实时追踪' : '将手放在摄像头前' }}</text>
      </view>
    </view>

    <!-- 加载状态 -->
    <view class="loading-overlay" v-if="!arReady">
      <view class="loading-spinner"></view>
      <text class="loading-text">正在启动AR...</text>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      width: 300,
      height: 300,
      renderWidth: 300,
      renderHeight: 300,
      arReady: false,
      isHandDetected: false,
      currentGesture: {
        name: '未知',
        icon: '✋',
        confidence: 0
      },
      supportedGestures: [
        { name: '张开手掌', icon: '✋' },
        { name: '握拳', icon: '✊' },
        { name: '比耶', icon: '✌️' },
        { name: '点赞', icon: '👍' },
        { name: 'OK', icon: '👌' }
      ]
    };
  },
  onLoad() {
    const info = uni.getSystemInfoSync();
    this.width = info.windowWidth;
    this.height = info.windowHeight;
    this.renderWidth = this.width * info.pixelRatio;
    this.renderHeight = this.height * info.pixelRatio;
  },
  methods: {
    handleBack() {
      uni.navigateBack({ delta: 1 });
    },
    handleARReady() {
      this.arReady = true;
      uni.showToast({
        title: 'AR已就绪',
        icon: 'success',
        duration: 1500
      });
    },
    handleHandDetected(e) {
      this.isHandDetected = true;
      // 解析手势信息
      if (e.detail && e.detail.gesture) {
        this.currentGesture = {
          name: e.detail.gesture.name || '张开手掌',
          icon: this.getGestureIcon(e.detail.gesture.name),
          confidence: Math.round((e.detail.gesture.confidence || 0.95) * 100)
        };
      }
      console.log('手部识别成功:', e.detail);
    },
    handleHandLost(e) {
      this.isHandDetected = false;
      this.currentGesture = {
        name: '未知',
        icon: '✋',
        confidence: 0
      };
      console.log('手部丢失:', e.detail);
    },
    getGestureIcon(name) {
      const iconMap = {
        '张开手掌': '✋',
        '握拳': '✊',
        '比耶': '✌️',
        '点赞': '👍',
        'OK': '👌'
      };
      return iconMap[name] || '✋';
    }
  }
};
</script>

<style scoped>
.ar-page {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: #000;
}

/* 手部识别引导 */
.hand-guide {
  position: absolute;
  top: 40%;
  left: 50%;
  transform: translate(-50%, -50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  z-index: 50;
}

.hand-icon {
  font-size: 120rpx;
  margin-bottom: 30rpx;
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0%, 100% { transform: scale(1); opacity: 1; }
  50% { transform: scale(1.1); opacity: 0.8; }
}

.hand-text {
  font-size: 32rpx;
  color: white;
  background: rgba(0, 0, 0, 0.6);
  padding: 20rpx 40rpx;
  border-radius: 40rpx;
  margin-bottom: 20rpx;
}

.hand-tips {
  font-size: 24rpx;
  color: rgba(255, 255, 255, 0.7);
  text-align: center;
}

/* 手势信息 */
.gesture-info {
  position: absolute;
  top: 180rpx;
  left: 50%;
  transform: translateX(-50%);
  z-index: 60;
}

.gesture-card {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 24rpx;
  padding: 40rpx 60rpx;
  display: flex;
  flex-direction: column;
  align-items: center;
  box-shadow: 0 8rpx 32rpx rgba(0, 0, 0, 0.2);
}

.gesture-icon {
  font-size: 80rpx;
  margin-bottom: 16rpx;
}

.gesture-name {
  font-size: 36rpx;
  font-weight: bold;
  color: #333;
  margin-bottom: 12rpx;
}

.gesture-confidence {
  font-size: 24rpx;
  color: #52c41a;
  background: rgba(82, 196, 26, 0.1);
  padding: 8rpx 20rpx;
  border-radius: 20rpx;
}

/* 信息面板 */
.info-panel {
  position: absolute;
  bottom: 40rpx;
  left: 30rpx;
  right: 30rpx;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 20rpx;
  padding: 30rpx;
  z-index: 100;
}

.info-title {
  font-size: 32rpx;
  font-weight: bold;
  color: #333;
  margin-bottom: 10rpx;
}

.info-desc {
  font-size: 26rpx;
  color: #666;
  margin-bottom: 24rpx;
}

.status-section {
  margin-bottom: 24rpx;
}

.status-item {
  display: flex;
  align-items: center;
  padding: 16rpx 20rpx;
  background: rgba(0, 0, 0, 0.05);
  border-radius: 12rpx;
}

.status-item.success {
  background: rgba(82, 196, 26, 0.1);
}

.status-dot {
  width: 16rpx;
  height: 16rpx;
  border-radius: 50%;
  margin-right: 16rpx;
}

.status-dot.loading {
  background: #faad14;
  animation: blink 1s infinite;
}

.status-dot.scanning {
  background: #1890ff;
  animation: blink 1s infinite;
}

.status-dot.detected {
  background: #52c41a;
}

@keyframes blink {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

.status-label {
  font-size: 28rpx;
  color: #333;
}

.status-item.success .status-label {
  color: #52c41a;
  font-weight: bold;
}

.gesture-list {
  margin-bottom: 24rpx;
}

.gesture-title {
  font-size: 28rpx;
  font-weight: bold;
  color: #333;
  margin-bottom: 16rpx;
  display: block;
}

.gesture-items {
  display: flex;
  flex-wrap: wrap;
  gap: 12rpx;
}

.gesture-tag {
  padding: 12rpx 24rpx;
  background: rgba(221, 160, 221, 0.1);
  border-radius: 30rpx;
  font-size: 24rpx;
  color: #666;
  border: 2rpx solid transparent;
}

.gesture-tag.active {
  background: rgba(221, 160, 221, 0.3);
  border-color: #DDA0DD;
  color: #333;
  font-weight: bold;
}

.info-tips {
  display: flex;
  align-items: center;
  padding-top: 20rpx;
  border-top: 1rpx solid rgba(0, 0, 0, 0.1);
}

.tip-icon {
  font-size: 28rpx;
  margin-right: 10rpx;
}

.tip-text {
  font-size: 24rpx;
  color: #666;
}

/* 加载状态 */
.loading-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  z-index: 200;
}

.loading-spinner {
  width: 80rpx;
  height: 80rpx;
  border: 6rpx solid rgba(255, 255, 255, 0.3);
  border-top-color: #DDA0DD;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.loading-text {
  margin-top: 30rpx;
  font-size: 28rpx;
  color: white;
}
</style>
