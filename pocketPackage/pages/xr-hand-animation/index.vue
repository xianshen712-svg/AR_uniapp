<template>
  <view class="ar-page">
    <yn-viewer :showBackBtn="true" @back="handleBack">
      <yn-ar
        id="hand-animation"
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

    <view class="hand-guide" v-if="!isHandDetected && arReady">
      <view class="hand-icon">✋</view>
      <view class="hand-text">请做出手势控制模型动画</view>
      <view class="hand-tips">✊ 旋转 | ✌️ 缩放 | 👍 浮动 | 👌 摇摆</view>
    </view>

    <view class="status-display" v-if="isHandDetected">
      <view class="gesture-badge">
        <text class="gesture-icon">{{ currentGesture.icon }}</text>
        <text class="gesture-name">{{ currentGesture.name }}</text>
      </view>
      <view class="animation-badge" :style="{ backgroundColor: currentAnimation.color }">
        <text class="animation-icon">{{ currentAnimation.icon }}</text>
        <text class="animation-name">{{ currentAnimation.name }}</text>
      </view>
    </view>

    <view class="model-display" v-if="isHandDetected">
      <view class="model-container">
        <view class="model-3d" :class="currentAnimation.class">
          <view class="model-core">🎯</view>
          <view class="model-ring ring-1"></view>
          <view class="model-ring ring-2"></view>
          <view class="model-ring ring-3"></view>
        </view>
      </view>
    </view>

    <view class="info-panel">
      <view class="info-title">🎮 手部动画交互</view>
      <view class="info-desc">手势控制3D模型动画</view>

      <view class="gesture-map">
        <view class="map-title">手势映射:</view>
        <view class="map-items">
          <view class="map-item" v-for="(map, index) in gestureMap" :key="index" :class="{ active: currentGesture.name === map.gesture }">
            <view class="map-gesture">{{ map.gestureIcon }} {{ map.gesture }}</view>
            <view class="map-arrow">→</view>
            <view class="map-animation" :style="{ color: map.color }">{{ map.animationIcon }} {{ map.animation }}</view>
          </view>
        </view>
      </view>

      <view class="status-section">
        <view class="status-item" v-if="!arReady">
          <view class="status-dot loading"></view>
          <text class="status-label">正在启动AR...</text>
        </view>
        <view class="status-item" v-else-if="!isHandDetected">
          <view class="status-dot scanning"></view>
          <text class="status-label">等待手势...</text>
        </view>
        <view class="status-item success" v-else>
          <view class="status-dot detected"></view>
          <text class="status-label">手势识别中 - {{ currentAnimation.name }}</text>
        </view>
      </view>

      <view class="info-tips">
        <text class="tip-icon">💡</text>
        <text class="tip-text">改变手势可切换不同的模型动画效果</text>
      </view>
    </view>

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
        icon: '✋'
      },
      currentAnimation: {
        name: '待机',
        icon: '⏸️',
        class: 'anim-idle',
        color: '#999'
      },
      gestureMap: [
        { gesture: '握拳', gestureIcon: '✊', animation: '旋转', animationIcon: '🔄', color: '#667eea', class: 'anim-rotate' },
        { gesture: '比耶', gestureIcon: '✌️', animation: '缩放', animationIcon: '💓', color: '#ff6b6b', class: 'anim-scale' },
        { gesture: '点赞', gestureIcon: '👍', animation: '浮动', animationIcon: '↕️', color: '#4ecdc4', class: 'anim-float' },
        { gesture: 'OK', gestureIcon: '👌', animation: '摇摆', animationIcon: '↔️', color: '#f7dc6f', class: 'anim-swing' }
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
      if (e.detail && e.detail.gesture) {
        const gestureName = e.detail.gesture.name || '张开手掌';
        this.updateGestureAndAnimation(gestureName);
      }
    },
    handleHandLost(e) {
      this.isHandDetected = false;
      this.currentGesture = { name: '未知', icon: '✋' };
      this.currentAnimation = { name: '待机', icon: '⏸️', class: 'anim-idle', color: '#999' };
    },
    updateGestureAndAnimation(gestureName) {
      const gestureIcons = {
        '握拳': '✊',
        '比耶': '✌️',
        '点赞': '👍',
        'OK': '👌',
        '张开手掌': '✋'
      };
      this.currentGesture = {
        name: gestureName,
        icon: gestureIcons[gestureName] || '✋'
      };

      const mapping = this.gestureMap.find(m => m.gesture === gestureName);
      if (mapping) {
        this.currentAnimation = {
          name: mapping.animation,
          icon: mapping.animationIcon,
          class: mapping.class,
          color: mapping.color
        };
      }
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

.hand-guide {
  position: absolute;
  top: 35%;
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
  font-size: 26rpx;
  color: rgba(255, 255, 255, 0.8);
  text-align: center;
  background: rgba(102, 126, 234, 0.3);
  padding: 16rpx 32rpx;
  border-radius: 30rpx;
}

.status-display {
  position: absolute;
  top: 160rpx;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 20rpx;
  z-index: 60;
}

.gesture-badge, .animation-badge {
  display: flex;
  align-items: center;
  background: rgba(255, 255, 255, 0.95);
  padding: 16rpx 32rpx;
  border-radius: 40rpx;
  box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.2);
}

.gesture-icon, .animation-icon {
  font-size: 40rpx;
  margin-right: 12rpx;
}

.gesture-name, .animation-name {
  font-size: 28rpx;
  font-weight: bold;
  color: #333;
}

.model-display {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 40;
}

.model-container {
  width: 300rpx;
  height: 300rpx;
  display: flex;
  align-items: center;
  justify-content: center;
}

.model-3d {
  position: relative;
  width: 200rpx;
  height: 200rpx;
  display: flex;
  align-items: center;
  justify-content: center;
}

.model-core {
  font-size: 100rpx;
  z-index: 10;
  filter: drop-shadow(0 0 20rpx rgba(102, 126, 234, 0.8));
}

.model-ring {
  position: absolute;
  border: 4rpx solid rgba(102, 126, 234, 0.5);
  border-radius: 50%;
}

.ring-1 {
  width: 250rpx;
  height: 250rpx;
  animation: ring-pulse 2s infinite;
}

.ring-2 {
  width: 300rpx;
  height: 300rpx;
  animation: ring-pulse 2s infinite 0.5s;
}

.ring-3 {
  width: 350rpx;
  height: 350rpx;
  animation: ring-pulse 2s infinite 1s;
}

@keyframes ring-pulse {
  0%, 100% { transform: scale(1); opacity: 0.8; }
  50% { transform: scale(1.1); opacity: 0.3; }
}

.anim-rotate {
  animation: rotate-3d 2s linear infinite;
}

@keyframes rotate-3d {
  from { transform: rotateY(0deg); }
  to { transform: rotateY(360deg); }
}

.anim-scale {
  animation: scale-pulse 1.5s ease-in-out infinite;
}

@keyframes scale-pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.3); }
}

.anim-float {
  animation: float-updown 2s ease-in-out infinite;
}

@keyframes float-updown {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-50rpx); }
}

.anim-swing {
  animation: swing-sway 2s ease-in-out infinite;
}

@keyframes swing-sway {
  0%, 100% { transform: rotateZ(-15deg); }
  50% { transform: rotateZ(15deg); }
}

.anim-idle {
  animation: idle-breathe 3s ease-in-out infinite;
}

@keyframes idle-breathe {
  0%, 100% { transform: scale(1); opacity: 1; }
  50% { transform: scale(1.05); opacity: 0.8; }
}

.info-panel {
  position: absolute;
  bottom: 40rpx;
  left: 30rpx;
  right: 30rpx;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 24rpx;
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

.gesture-map {
  margin-bottom: 24rpx;
}

.map-title {
  font-size: 28rpx;
  font-weight: bold;
  color: #333;
  margin-bottom: 16rpx;
  display: block;
}

.map-items {
  display: flex;
  flex-direction: column;
  gap: 12rpx;
}

.map-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16rpx 20rpx;
  background: rgba(0, 0, 0, 0.05);
  border-radius: 16rpx;
  border: 2rpx solid transparent;
  transition: all 0.3s;
}

.map-item.active {
  background: rgba(102, 126, 234, 0.1);
  border-color: #667eea;
}

.map-gesture {
  font-size: 26rpx;
  color: #333;
  font-weight: 500;
}

.map-arrow {
  font-size: 28rpx;
  color: #999;
}

.map-animation {
  font-size: 26rpx;
  font-weight: bold;
}

.status-section {
  margin-bottom: 20rpx;
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
  border-top-color: #667eea;
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