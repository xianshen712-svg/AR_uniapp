<template>
  <view class="ar-page">
    <yn-viewer :showBackBtn="true" @back="handleBack">
      <yn-ar
        id="tracker-2d"
        :width="renderWidth"
        :height="renderHeight"
        :style="'width:' + width + 'px;height:' + height + 'px'"
        yn_ar_mode="Marker"
        yn_camera_clear_color="0.1 0.1 0.1 1"
        @arReady="handleARReady"
        @markerDetected="handleMarkerDetected"
        @markerLost="handleMarkerLost"
      />
    </yn-viewer>

    <!-- 扫描引导 -->
    <view class="scan-guide" v-if="!currentTarget && arReady">
      <view class="scan-frame">
        <view class="scan-corner top-left"></view>
        <view class="scan-corner top-right"></view>
        <view class="scan-corner bottom-left"></view>
        <view class="scan-corner bottom-right"></view>
        <view class="scan-line"></view>
      </view>
      <view class="scan-text">请将图片对准扫描框</view>
      <view class="scan-hint">支持识别 {{ targetList.length }} 种不同的图片</view>
    </view>

    <!-- 识别成功 - 视频播放 -->
    <view class="video-overlay" v-if="currentTarget && isVideoPlaying">
      <view class="video-container">
        <view class="video-placeholder">
          <text class="video-icon">▶️</text>
          <text class="video-text">{{ currentTarget.videoName }}</text>
        </view>
        <view class="video-progress">
          <view class="progress-bar">
            <view class="progress-fill" :style="{ width: videoProgress + '%' }"></view>
          </view>
          <text class="progress-time">{{ currentTime }} / {{ totalTime }}</text>
        </view>
        <view class="video-controls">
          <view class="control-btn" @tap="togglePlay">
            <text class="btn-icon">{{ isPlaying ? '⏸️' : '▶️' }}</text>
          </view>
          <view class="control-btn" @tap="stopVideo">
            <text class="btn-icon">⏹️</text>
          </view>
          <view class="control-btn" @tap="switchTarget">
            <text class="btn-icon">🔄</text>
          </view>
        </view>
      </view>
    </view>

    <!-- 识别成功 - 信息展示 -->
    <view class="target-info" v-if="currentTarget">
      <view class="info-card">
        <view class="card-header">
          <view class="target-icon">{{ currentTarget.icon }}</view>
          <view class="target-title">{{ currentTarget.name }}</view>
        </view>
        <view class="card-body">
          <view class="info-row">
            <text class="info-label">识别ID:</text>
            <text class="info-value">{{ currentTarget.id }}</text>
          </view>
          <view class="info-row">
            <text class="info-label">视频时长:</text>
            <text class="info-value">{{ currentTarget.duration }}</text>
          </view>
          <view class="info-row">
            <text class="info-label">识别次数:</text>
            <text class="info-value">{{ currentTarget.detectCount }} 次</text>
          </view>
        </view>
      </view>
    </view>

    <!-- 底部信息面板 -->
    <view class="info-panel">
      <view class="info-title">🎯 综合图片视频识别</view>
      <view class="info-desc">支持多目标图片识别和视频播放</view>

      <view class="status-section">
        <view class="status-item" v-if="!arReady">
          <view class="status-dot loading"></view>
          <text class="status-label">正在启动AR...</text>
        </view>
        <view class="status-item" v-else-if="!currentTarget">
          <view class="status-dot scanning"></view>
          <text class="status-label">正在扫描图片...</text>
        </view>
        <view class="status-item success" v-else>
          <view class="status-dot detected"></view>
          <text class="status-label">已识别: {{ currentTarget.name }}</text>
        </view>
      </view>

      <view class="target-list">
        <view class="list-title">可识别目标 ({{ detectedCount }}/{{ targetList.length }}):</view>
        <view class="target-items">
          <view 
            class="target-item" 
            v-for="(target, index) in targetList" 
            :key="index"
            :class="{ detected: target.detected, active: currentTarget && currentTarget.id === target.id }"
          >
            <view class="item-icon">{{ target.icon }}</view>
            <view class="item-info">
              <text class="item-name">{{ target.name }}</text>
              <text class="item-status">{{ target.detected ? '已识别' : '未识别' }}</text>
            </view>
            <view class="item-check" v-if="target.detected">✓</view>
          </view>
        </view>
      </view>

      <view class="stats-section">
        <view class="stat-item">
          <text class="stat-value">{{ totalDetectCount }}</text>
          <text class="stat-label">总识别次数</text>
        </view>
        <view class="stat-item">
          <text class="stat-value">{{ detectedCount }}</text>
          <text class="stat-label">已识别目标</text>
        </view>
        <view class="stat-item">
          <text class="stat-value">{{ targetList.length - detectedCount }}</text>
          <text class="stat-label">待识别目标</text>
        </view>
      </view>

      <view class="info-tips">
        <text class="tip-icon">💡</text>
        <text class="tip-text">{{ currentTarget ? '移开图片可识别其他目标' : '扫描不同的图片查看效果' }}</text>
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
      currentTarget: null,
      isVideoPlaying: false,
      isPlaying: true,
      videoProgress: 0,
      currentTime: '00:00',
      totalTime: '00:15',
      progressTimer: null,
      totalDetectCount: 0,
      targetList: [
        {
          id: 'target-001',
          name: 'Hikari',
          icon: '🌸',
          markerUrl: 'https://mmbizwxaminiprogram-1258344707.cos.ap-guangzhou.myqcloud.com/xr-frame/demo/xr-frame-team/2dmarker/hikari.jpg',
          videoUrl: 'https://mmbizwxaminiprogram-1258344707.cos.ap-guangzhou.myqcloud.com/xr-frame/demo/xr-frame-team/2dmarker/hikari-v.mp4',
          videoName: 'Hikari视频',
          duration: '00:15',
          detected: false,
          detectCount: 0
        },
        {
          id: 'target-002',
          name: 'Miku',
          icon: '🎤',
          markerUrl: 'https://mmbizwxaminiprogram-1258344707.cos.ap-guangzhou.myqcloud.com/xr-frame/demo/xr-frame-team/2dmarker/miku.jpg',
          videoUrl: 'https://mmbizwxaminiprogram-1258344707.cos.ap-guangzhou.myqcloud.com/xr-frame/demo/xr-frame-team/2dmarker/miku-v.mp4',
          videoName: 'Miku视频',
          duration: '00:12',
          detected: false,
          detectCount: 0
        },
        {
          id: 'target-003',
          name: 'Rin',
          icon: '🎸',
          markerUrl: 'https://mmbizwxaminiprogram-1258344707.cos.ap-guangzhou.myqcloud.com/xr-frame/demo/xr-frame-team/2dmarker/rin.jpg',
          videoUrl: 'https://mmbizwxaminiprogram-1258344707.cos.ap-guangzhou.myqcloud.com/xr-frame/demo/xr-frame-team/2dmarker/rin-v.mp4',
          videoName: 'Rin视频',
          duration: '00:18',
          detected: false,
          detectCount: 0
        }
      ]
    };
  },
  computed: {
    detectedCount() {
      return this.targetList.filter(t => t.detected).length;
    }
  },
  onLoad() {
    const info = uni.getSystemInfoSync();
    this.width = info.windowWidth;
    this.height = info.windowHeight;
    this.renderWidth = this.width * info.pixelRatio;
    this.renderHeight = this.height * info.pixelRatio;
  },
  onUnload() {
    if (this.progressTimer) {
      clearInterval(this.progressTimer);
    }
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
    handleMarkerDetected(e) {
      // 根据识别结果匹配目标
      const targetId = e.detail && e.detail.targetId ? e.detail.targetId : 'target-001';
      const target = this.targetList.find(t => t.id === targetId);
      
      if (target) {
        this.currentTarget = target;
        target.detected = true;
        target.detectCount++;
        this.totalDetectCount++;
        this.isVideoPlaying = true;
        this.startVideoProgress();
        
        uni.showToast({
          title: `识别成功: ${target.name}`,
          icon: 'success',
          duration: 2000
        });
      }
    },
    handleMarkerLost(e) {
      this.currentTarget = null;
      this.isVideoPlaying = false;
      this.stopVideoProgress();
    },
    startVideoProgress() {
      this.videoProgress = 0;
      this.currentTime = '00:00';
      this.progressTimer = setInterval(() => {
        if (this.isPlaying && this.videoProgress < 100) {
          this.videoProgress += 1;
          this.updateCurrentTime();
        } else if (this.videoProgress >= 100) {
          this.videoProgress = 0;
        }
      }, 150);
    },
    stopVideoProgress() {
      if (this.progressTimer) {
        clearInterval(this.progressTimer);
        this.progressTimer = null;
      }
    },
    updateCurrentTime() {
      const totalSeconds = 15;
      const currentSeconds = Math.floor((this.videoProgress / 100) * totalSeconds);
      const minutes = Math.floor(currentSeconds / 60).toString().padStart(2, '0');
      const seconds = (currentSeconds % 60).toString().padStart(2, '0');
      this.currentTime = `${minutes}:${seconds}`;
    },
    togglePlay() {
      this.isPlaying = !this.isPlaying;
      uni.showToast({
        title: this.isPlaying ? '继续播放' : '已暂停',
        icon: 'none',
        duration: 1000
      });
    },
    stopVideo() {
      this.isPlaying = false;
      this.videoProgress = 0;
      this.currentTime = '00:00';
      uni.showToast({
        title: '视频已停止',
        icon: 'none',
        duration: 1000
      });
    },
    switchTarget() {
      // 切换到下一个未识别的目标
      const undetected = this.targetList.filter(t => !t.detected);
      if (undetected.length > 0) {
        uni.showToast({
          title: `还有 ${undetected.length} 个目标待识别`,
          icon: 'none',
          duration: 1500
        });
      } else {
        uni.showToast({
          title: '所有目标已识别完成！',
          icon: 'success',
          duration: 2000
        });
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

/* 扫描引导 */
.scan-guide {
  position: absolute;
  top: 35%;
  left: 50%;
  transform: translate(-50%, -50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  z-index: 50;
}

.scan-frame {
  width: 400rpx;
  height: 400rpx;
  position: relative;
  border: 2rpx solid rgba(78, 205, 196, 0.3);
}

.scan-corner {
  position: absolute;
  width: 40rpx;
  height: 40rpx;
  border-color: #4ECDC4;
  border-style: solid;
}

.scan-corner.top-left {
  top: -4rpx;
  left: -4rpx;
  border-width: 6rpx 0 0 6rpx;
}

.scan-corner.top-right {
  top: -4rpx;
  right: -4rpx;
  border-width: 6rpx 6rpx 0 0;
}

.scan-corner.bottom-left {
  bottom: -4rpx;
  left: -4rpx;
  border-width: 0 0 6rpx 6rpx;
}

.scan-corner.bottom-right {
  bottom: -4rpx;
  right: -4rpx;
  border-width: 0 6rpx 6rpx 0;
}

.scan-line {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 4rpx;
  background: linear-gradient(90deg, transparent, #4ECDC4, transparent);
  animation: scan 2s linear infinite;
}

@keyframes scan {
  0% { top: 0; }
  100% { top: 100%; }
}

.scan-text {
  margin-top: 30rpx;
  font-size: 32rpx;
  color: white;
  background: rgba(0, 0, 0, 0.6);
  padding: 16rpx 32rpx;
  border-radius: 30rpx;
}

.scan-hint {
  margin-top: 16rpx;
  font-size: 26rpx;
  color: rgba(255, 255, 255, 0.7);
}

/* 视频覆盖层 */
.video-overlay {
  position: absolute;
  top: 180rpx;
  left: 50%;
  transform: translateX(-50%);
  z-index: 60;
}

.video-container {
  width: 560rpx;
  background: rgba(0, 0, 0, 0.9);
  border-radius: 24rpx;
  overflow: hidden;
  box-shadow: 0 8rpx 32rpx rgba(0, 0, 0, 0.4);
}

.video-placeholder {
  height: 300rpx;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.video-icon {
  font-size: 80rpx;
  margin-bottom: 16rpx;
}

.video-text {
  font-size: 28rpx;
  color: white;
  font-weight: bold;
}

.video-progress {
  padding: 20rpx 24rpx;
  background: rgba(255, 255, 255, 0.1);
}

.progress-bar {
  height: 8rpx;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 4rpx;
  overflow: hidden;
  margin-bottom: 12rpx;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #4ECDC4, #44A08D);
  border-radius: 4rpx;
  transition: width 0.1s linear;
}

.progress-time {
  font-size: 22rpx;
  color: rgba(255, 255, 255, 0.8);
  text-align: center;
  display: block;
}

.video-controls {
  display: flex;
  justify-content: center;
  gap: 40rpx;
  padding: 20rpx;
  background: rgba(0, 0, 0, 0.5);
}

.control-btn {
  width: 80rpx;
  height: 80rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 50%;
  transition: all 0.3s;
}

.control-btn:active {
  background: rgba(255, 255, 255, 0.3);
  transform: scale(0.95);
}

.btn-icon {
  font-size: 40rpx;
}

/* 目标信息 */
.target-info {
  position: absolute;
  top: 180rpx;
  right: 30rpx;
  z-index: 55;
}

.info-card {
  width: 280rpx;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 20rpx;
  padding: 24rpx;
  box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.2);
}

.card-header {
  display: flex;
  align-items: center;
  margin-bottom: 20rpx;
  padding-bottom: 16rpx;
  border-bottom: 2rpx solid rgba(0, 0, 0, 0.1);
}

.target-icon {
  font-size: 48rpx;
  margin-right: 16rpx;
}

.target-title {
  font-size: 30rpx;
  font-weight: bold;
  color: #333;
}

.card-body {
  display: flex;
  flex-direction: column;
  gap: 12rpx;
}

.info-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.info-label {
  font-size: 24rpx;
  color: #666;
}

.info-value {
  font-size: 24rpx;
  color: #333;
  font-weight: 500;
}

/* 信息面板 */
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

/* 目标列表 */
.target-list {
  margin-bottom: 24rpx;
}

.list-title {
  font-size: 28rpx;
  font-weight: bold;
  color: #333;
  margin-bottom: 16rpx;
  display: block;
}

.target-items {
  display: flex;
  flex-direction: column;
  gap: 12rpx;
}

.target-item {
  display: flex;
  align-items: center;
  padding: 16rpx 20rpx;
  background: rgba(0, 0, 0, 0.05);
  border-radius: 16rpx;
  border: 2rpx solid transparent;
  transition: all 0.3s;
}

.target-item.detected {
  background: rgba(82, 196, 26, 0.1);
  border-color: #52c41a;
}

.target-item.active {
  background: rgba(78, 205, 196, 0.15);
  border-color: #4ECDC4;
}

.item-icon {
  font-size: 40rpx;
  margin-right: 16rpx;
}

.item-info {
  flex: 1;
}

.item-name {
  font-size: 28rpx;
  color: #333;
  font-weight: 500;
  display: block;
}

.item-status {
  font-size: 22rpx;
  color: #999;
}

.target-item.detected .item-status {
  color: #52c41a;
}

.item-check {
  width: 40rpx;
  height: 40rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #52c41a;
  border-radius: 50%;
  color: white;
  font-size: 24rpx;
  font-weight: bold;
}

/* 统计区域 */
.stats-section {
  display: flex;
  justify-content: space-around;
  padding: 20rpx 0;
  margin-bottom: 20rpx;
  border-top: 1rpx solid rgba(0, 0, 0, 0.1);
  border-bottom: 1rpx solid rgba(0, 0, 0, 0.1);
}

.stat-item {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.stat-value {
  font-size: 36rpx;
  font-weight: bold;
  color: #667eea;
}

.stat-label {
  font-size: 22rpx;
  color: #666;
  margin-top: 4rpx;
}

.info-tips {
  display: flex;
  align-items: center;
  padding-top: 20rpx;
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
  border-top-color: #4ECDC4;
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
