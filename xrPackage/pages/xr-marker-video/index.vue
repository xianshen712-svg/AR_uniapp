<template>
    <view class="ar-page">
        <yn-viewer :showBackBtn="true" @back="handleBack">
            <yn-ar
                id="marker-video"
                :width="renderWidth"
                :height="renderHeight"
                :style="'width:' + width + 'px;height:' + height + 'px'"
                yn_ar_mode="Marker"
                yn_marker_url="https://mmbizwxaminiprogram-1258344707.cos.ap-guangzhou.myqcloud.com/xr-frame/demo/xr-frame-team/2dmarker/hikari.jpg"
                yn_video_url="https://mmbizwxaminiprogram-1258344707.cos.ap-guangzhou.myqcloud.com/xr-frame/demo/xr-frame-team/2dmarker/hikari-v.mp4"
                yn_camera_clear_color="0.1 0.1 0.1 1"
                @arReady="handleARReady"
                @markerDetected="handleMarkerDetected"
                @markerLost="handleMarkerLost"
            />
        </yn-viewer>

        <!-- 扫描引导 -->
        <view class="scan-guide" v-if="!isDetected && arReady">
            <view class="scan-frame">
                <view class="scan-corner top-left"></view>
                <view class="scan-corner top-right"></view>
                <view class="scan-corner bottom-left"></view>
                <view class="scan-corner bottom-right"></view>
                <view class="scan-line"></view>
            </view>
            <view class="scan-text">请将Marker图片对准扫描框</view>
        </view>

        <!-- 识别成功提示 -->
        <view class="detected-toast" v-if="isDetected">
            <view class="toast-content">
                <text class="toast-icon">✓</text>
                <text class="toast-text">识别成功，正在播放视频</text>
            </view>
        </view>

        <!-- 底部信息面板 -->
        <view class="info-panel">
            <view class="info-title">📱 AR扫描播放视频</view>
            <view class="info-desc">扫描Marker图片播放视频内容</view>
            
            <view class="status-section">
                <view class="status-item" v-if="!arReady">
                    <view class="status-dot loading"></view>
                    <text class="status-label">正在启动AR...</text>
                </view>
                <view class="status-item" v-else-if="!isDetected">
                    <view class="status-dot scanning"></view>
                    <text class="status-label">正在扫描Marker...</text>
                </view>
                <view class="status-item success" v-else>
                    <view class="status-dot detected"></view>
                    <text class="status-label">Marker已识别</text>
                </view>
            </view>

            <view class="marker-preview">
                <view class="preview-title">Marker图片示例:</view>
                <view class="preview-image">
                    <text class="preview-icon">🖼️</text>
                    <text class="preview-text">hikari.jpg</text>
                </view>
            </view>

            <view class="info-tips">
                <text class="tip-icon">💡</text>
                <text class="tip-text">{{ isDetected ? '移开图片可暂停视频' : '将Marker图片对准摄像头' }}</text>
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
            isDetected: false
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
        handleMarkerDetected(e) {
            this.isDetected = true;
            console.log('Marker识别成功:', e.detail);
        },
        handleMarkerLost(e) {
            this.isDetected = false;
            console.log('Marker丢失:', e.detail);
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
    top: 50%;
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
    font-size: 28rpx;
    color: white;
    background: rgba(0, 0, 0, 0.6);
    padding: 16rpx 32rpx;
    border-radius: 30rpx;
}

/* 识别成功提示 */
.detected-toast {
    position: absolute;
    top: 200rpx;
    left: 50%;
    transform: translateX(-50%);
    z-index: 60;
}

.toast-content {
    display: flex;
    align-items: center;
    background: rgba(82, 196, 26, 0.9);
    padding: 20rpx 40rpx;
    border-radius: 40rpx;
}

.toast-icon {
    font-size: 32rpx;
    color: white;
    margin-right: 16rpx;
}

.toast-text {
    font-size: 28rpx;
    color: white;
    font-weight: bold;
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
    animation: pulse 1s infinite;
}

.status-dot.scanning {
    background: #1890ff;
    animation: pulse 1s infinite;
}

.status-dot.detected {
    background: #52c41a;
}

@keyframes pulse {
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

.marker-preview {
    margin-bottom: 24rpx;
}

.preview-title {
    font-size: 26rpx;
    color: #666;
    margin-bottom: 16rpx;
    display: block;
}

.preview-image {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    background: rgba(78, 205, 196, 0.1);
    border-radius: 16rpx;
    padding: 30rpx;
    border: 2rpx dashed #4ECDC4;
}

.preview-icon {
    font-size: 64rpx;
    margin-bottom: 12rpx;
}

.preview-text {
    font-size: 24rpx;
    color: #4ECDC4;
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
