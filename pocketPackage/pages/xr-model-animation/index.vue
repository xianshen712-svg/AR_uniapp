<template>
  <view class="xr-page">
    <yn-viewer :showBackBtn="true" @back="handleBack">
      <yn-3d
        id="model-animation"
        :width="renderWidth"
        :height="renderHeight"
        :style="'width:' + width + 'px;height:' + height + 'px'"
        yn_asset="https://mmbizwxaminiprogram-1258344707.cos.ap-guangzhou.myqcloud.com/xr-frame/demo/xr-frame-team/2dmarker/hikari-v.mp4"
        yn_asset_position="0 0 0"
        yn_asset_rotation="0 0 0"
        yn_asset_scale="1 1 1"
        yn_camera_position="0 1 4"
        yn_camera_clear_color="0.1 0.1 0.2 1"
        @ready="handleReady"
      />
    </yn-viewer>

    <view class="animation-panel" v-if="showControls">
      <view class="panel-title">🎬 模型动画控制</view>
      
      <view class="animation-info">
        <view class="info-item">
          <text class="info-label">当前动画:</text>
          <text class="info-value">{{ currentAnimation }}</text>
        </view>
        <view class="info-item">
          <text class="info-label">播放状态:</text>
          <text class="info-value" :class="{ playing: isPlaying }">{{ isPlaying ? '播放中' : '已暂停' }}</text>
        </view>
        <view class="info-item">
          <text class="info-label">播放进度:</text>
          <view class="progress-bar">
            <view class="progress-fill" :style="{ width: progress + '%' }"></view>
          </view>
          <text class="progress-text">{{ progress }}%</text>
        </view>
      </view>

      <view class="control-buttons">
        <view class="control-btn" @tap="togglePlay">
          <text class="btn-icon">{{ isPlaying ? '⏸️' : '▶️' }}</text>
          <text class="btn-text">{{ isPlaying ? '暂停' : '播放' }}</text>
        </view>
        <view class="control-btn" @tap="stopAnimation">
          <text class="btn-icon">⏹️</text>
          <text class="btn-text">停止</text>
        </view>
        <view class="control-btn" @tap="resetAnimation">
          <text class="btn-icon">🔄</text>
          <text class="btn-text">重置</text>
        </view>
      </view>

      <view class="animation-list">
        <view class="list-title">动画列表:</view>
        <view class="animation-items">
          <view 
            class="animation-item" 
            v-for="(anim, index) in animationList" 
            :key="index"
            :class="{ active: currentAnimation === anim.name }"
            @tap="selectAnimation(anim)"
          >
            <text class="anim-icon">{{ anim.icon }}</text>
            <text class="anim-name">{{ anim.name }}</text>
          </view>
        </view>
      </view>
    </view>

    <view class="loading-overlay" v-if="isLoading">
      <view class="loading-spinner"></view>
      <text class="loading-text">正在加载模型...</text>
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
      isLoading: true,
      showControls: false,
      isPlaying: true,
      currentAnimation: '旋转展示',
      progress: 0,
      animationTimer: null,
      animationList: [
        { name: '旋转展示', icon: '🔄' },
        { name: '缩放脉冲', icon: '💓' },
        { name: '上下浮动', icon: '↕️' },
        { name: '左右摇摆', icon: '↔️' }
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
  onUnload() {
    if (this.animationTimer) {
      clearInterval(this.animationTimer);
    }
  },
  methods: {
    handleBack() {
      uni.navigateBack({ delta: 1 });
    },
    handleReady() {
      this.isLoading = false;
      this.showControls = true;
      this.startProgressSimulation();
      uni.showToast({
        title: '模型加载完成',
        icon: 'success',
        duration: 2000
      });
    },
    startProgressSimulation() {
      this.animationTimer = setInterval(() => {
        if (this.isPlaying) {
          this.progress = (this.progress + 1) % 100;
        }
      }, 100);
    },
    togglePlay() {
      this.isPlaying = !this.isPlaying;
      uni.showToast({
        title: this.isPlaying ? '开始播放' : '已暂停',
        icon: 'none',
        duration: 1000
      });
    },
    stopAnimation() {
      this.isPlaying = false;
      this.progress = 0;
      uni.showToast({
        title: '动画已停止',
        icon: 'none',
        duration: 1000
      });
    },
    resetAnimation() {
      this.progress = 0;
      this.isPlaying = true;
      uni.showToast({
        title: '动画已重置',
        icon: 'none',
        duration: 1000
      });
    },
    selectAnimation(anim) {
      this.currentAnimation = anim.name;
      this.progress = 0;
      this.isPlaying = true;
      uni.showToast({
        title: `切换到: ${anim.name}`,
        icon: 'none',
        duration: 1500
      });
    }
  }
};
</script>

<style scoped>
.xr-page {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: #000;
}

.animation-panel {
  position: absolute;
  bottom: 40rpx;
  left: 30rpx;
  right: 30rpx;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 24rpx;
  padding: 30rpx;
  z-index: 100;
}

.panel-title {
  font-size: 32rpx;
  font-weight: bold;
  color: #333;
  margin-bottom: 24rpx;
  text-align: center;
}

.animation-info {
  margin-bottom: 24rpx;
}

.info-item {
  display: flex;
  align-items: center;
  padding: 12rpx 0;
  border-bottom: 1rpx solid rgba(0, 0, 0, 0.05);
}

.info-item:last-child {
  border-bottom: none;
}

.info-label {
  font-size: 26rpx;
  color: #666;
  width: 140rpx;
}

.info-value {
  font-size: 26rpx;
  color: #333;
  font-weight: 500;
}

.info-value.playing {
  color: #52c41a;
}

.progress-bar {
  flex: 1;
  height: 12rpx;
  background: rgba(0, 0, 0, 0.1);
  border-radius: 6rpx;
  margin: 0 16rpx;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #667eea, #764ba2);
  border-radius: 6rpx;
  transition: width 0.1s linear;
}

.progress-text {
  font-size: 24rpx;
  color: #666;
  width: 60rpx;
  text-align: right;
}

.control-buttons {
  display: flex;
  justify-content: space-around;
  margin-bottom: 24rpx;
  padding: 20rpx 0;
  border-top: 1rpx solid rgba(0, 0, 0, 0.1);
  border-bottom: 1rpx solid rgba(0, 0, 0, 0.1);
}

.control-btn {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 16rpx 30rpx;
  background: rgba(102, 126, 234, 0.1);
  border-radius: 16rpx;
  transition: all 0.3s;
}

.control-btn:active {
  background: rgba(102, 126, 234, 0.2);
  transform: scale(0.95);
}

.btn-icon {
  font-size: 40rpx;
  margin-bottom: 8rpx;
}

.btn-text {
  font-size: 24rpx;
  color: #667eea;
}

.animation-list {
  margin-top: 20rpx;
}

.list-title {
  font-size: 28rpx;
  font-weight: bold;
  color: #333;
  margin-bottom: 16rpx;
  display: block;
}

.animation-items {
  display: flex;
  flex-wrap: wrap;
  gap: 16rpx;
}

.animation-item {
  display: flex;
  align-items: center;
  padding: 16rpx 24rpx;
  background: rgba(0, 0, 0, 0.05);
  border-radius: 30rpx;
  border: 2rpx solid transparent;
  transition: all 0.3s;
}

.animation-item.active {
  background: rgba(102, 126, 234, 0.15);
  border-color: #667eea;
}

.animation-item:active {
  transform: scale(0.95);
}

.anim-icon {
  font-size: 32rpx;
  margin-right: 12rpx;
}

.anim-name {
  font-size: 26rpx;
  color: #333;
}

.animation-item.active .anim-name {
  font-weight: bold;
  color: #667eea;
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