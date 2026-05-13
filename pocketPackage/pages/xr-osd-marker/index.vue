<template>
    <view class="ar-page">
        <yn-viewer :showBackBtn="true" @back="handleBack">
            <yn-ar
                id="osd-marker"
                :width="renderWidth"
                :height="renderHeight"
                :style="'width:' + width + 'px;height:' + height + 'px'"
                yn_ar_mode="OSD"
                yn_marker_url="https://mmbizwxaminiprogram-1258344707.cos.ap-guangzhou.myqcloud.com/xr-frame/demo/marker/osdmarker-test.jpg"
                yn_camera_clear_color="0.925 0.925 0.925 1"
                @arReady="handleARReady"
                @markerDetected="handleMarkerDetected"
                @markerLost="handleMarkerLost"
            />
        </yn-viewer>

        <view class="scan-guide" v-if="!isDetected && arReady">
            <view class="scan-frame">
                <view class="scan-corner top-left"></view>
                <view class="scan-corner top-right"></view>
                <view class="scan-corner bottom-left"></view>
                <view class="scan-corner bottom-right"></view>
                <view class="scan-line"></view>
            </view>
            <view class="scan-text">请将物体对准扫描框</view>
        </view>

        <view class="info-card" v-if="isDetected">
            <view class="card-header">
                <view class="card-icon">🎯</view>
                <view class="card-title">物体识别成功</view>
            </view>
            <view class="card-body">
                <view class="info-item">
                    <text class="info-label">识别模式:</text>
                    <text class="info-value">OSD (One-shot Detection)</text>
                </view>
                <view class="info-item">
                    <text class="info-label">识别结果:</text>
                    <text class="info-value success">测试物体</text>
                </view>
                <view class="info-item">
                    <text class="info-label">置信度:</text>
                    <text class="info-value">95%</text>
                </view>
            </view>
            <view class="card-footer">
                <text class="footer-text">移开物体可重新识别</text>
            </view>
        </view>

        <view class="info-panel">
            <view class="info-title">🎯 OSD物体识别</view>
            <view class="info-desc">识别特定角度的物体并展示信息</view>
            
            <view class="status-section">
                <view class="status-item" v-if="!arReady">
                    <view class="status-dot loading"></view>
                    <text class="status-label">正在启动AR...</text>
                </view>
                <view class="status-item" v-else-if="!isDetected">
                    <view class="status-dot scanning"></view>
                    <text class="status-label">正在扫描物体...</text>
                </view>
                <view class="status-item success" v-else>
                    <view class="status-dot detected"></view>
                    <text class="status-label">物体已识别</text>
                </view>
            </view>

            <view class="osd-features">
                <view class="feature-title">OSD模式特点:</view>
                <view class="feature-list">
                    <view class="feature-item">
                        <text class="feature-dot">•</text>
                        <text class="feature-text">纯屏幕空间算法</text>
                    </view>
                    <view class="feature-item">
                        <text class="feature-dot">•</text>
                        <text class="feature-text">适合识别二维、特征清晰的物体</text>
                    </view>
                    <view class="feature-item">
                        <text class="feature-dot">•</text>
                        <text class="feature-text">识别速度快，如广告牌等</text>
                    </view>
                </view>
            </view>

            <view class="info-tips">
                <text class="tip-icon">💡</text>
                <text class="tip-text">{{ isDetected ? '移开物体可重新扫描' : '将物体对准摄像头进行识别' }}</text>
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
            uni.showToast({
                title: '物体识别成功',
                icon: 'success',
                duration: 2000
            });
            console.log('OSD识别成功:', e.detail);
        },
        handleMarkerLost(e) {
            this.isDetected = false;
            console.log('物体丢失:', e.detail);
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
    width: 450rpx;
    height: 450rpx;
    position: relative;
    border: 2rpx solid rgba(255, 179, 71, 0.3);
}

.scan-corner {
    position: absolute;
    width: 50rpx;
    height: 50rpx;
    border-color: #FFB347;
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
    background: linear-gradient(90deg, transparent, #FFB347, transparent);
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

.info-card {
    position: absolute;
    top: 180rpx;
    left: 50%;
    transform: translateX(-50%);
    width: 600rpx;
    background: rgba(255, 255, 255, 0.95);
    border-radius: 24rpx;
    padding: 30rpx;
    z-index: 60;
    box-shadow: 0 8rpx 32rpx rgba(0, 0, 0, 0.2);
}

.card-header {
    display: flex;
    align-items: center;
    margin-bottom: 24rpx;
    padding-bottom: 20rpx;
    border-bottom: 2rpx solid rgba(0, 0, 0, 0.1);
}

.card-icon {
    font-size: 48rpx;
    margin-right: 16rpx;
}

.card-title {
    font-size: 32rpx;
    font-weight: bold;
    color: #333;
}

.card-body {
    margin-bottom: 20rpx;
}

.info-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 16rpx 0;
    border-bottom: 1rpx solid rgba(0, 0, 0, 0.05);
}

.info-item:last-child {
    border-bottom: none;
}

.info-label {
    font-size: 28rpx;
    color: #666;
}

.info-value {
    font-size: 28rpx;
    color: #333;
    font-weight: 500;
}

.info-value.success {
    color: #52c41a;
}

.card-footer {
    padding-top: 20rpx;
    border-top: 2rpx solid rgba(0, 0, 0, 0.1);
    text-align: center;
}

.footer-text {
    font-size: 24rpx;
    color: #999;
}

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

.osd-features {
    margin-bottom: 24rpx;
    padding: 20rpx;
    background: rgba(255, 179, 71, 0.1);
    border-radius: 16rpx;
}

.feature-title {
    font-size: 28rpx;
    font-weight: bold;
    color: #333;
    margin-bottom: 16rpx;
    display: block;
}

.feature-list {
    display: flex;
    flex-direction: column;
    gap: 12rpx;
}

.feature-item {
    display: flex;
    align-items: flex-start;
}

.feature-dot {
    font-size: 32rpx;
    color: #FFB347;
    margin-right: 12rpx;
    line-height: 1;
}

.feature-text {
    font-size: 26rpx;
    color: #666;
    flex: 1;
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
    border-top-color: #FFB347;
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