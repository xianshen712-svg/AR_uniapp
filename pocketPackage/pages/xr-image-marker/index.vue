<template>
    <view class="image-marker-container">
        <view class="marker-header">
            <view class="back-btn" @tap="handleBack">
                <text class="back-icon">←</text>
            </view>
            <view class="header-title">AR图片Marker</view>
        </view>

        <view class="marker-viewer">
            <view class="camera-preview">
                <view class="marker-scan-area">
                    <view class="marker-frame">
                        <view class="frame-corner top-left"></view>
                        <view class="frame-corner top-right"></view>
                        <view class="frame-corner bottom-left"></view>
                        <view class="frame-corner bottom-right"></view>
                        <view class="frame-crosshair"></view>
                    </view>
                    <view class="detected-marker" v-if="detectedMarker">
                        <view class="marker-content">
                            <view class="marker-icon">{{ detectedMarker.icon }}</view>
                            <view class="marker-name">{{ detectedMarker.name }}</view>
                        </view>
                        <view class="marker-glow"></view>
                    </view>
                </view>
                <view class="scan-progress" v-if="isScanning">
                    <view class="progress-bar">
                        <view class="progress-fill" :style="{ width: scanProgress + '%' }"></view>
                    </view>
                    <view class="progress-text">扫描中 {{ scanProgress }}%</view>
                </view>
            </view>
        </view>

        <view class="marker-controls">
            <view class="control-btn" :class="{ active: isScanning }" @tap="toggleScan">
                <text class="control-icon">{{ isScanning ? '⏹️' : '🔍' }}</text>
                <text class="control-text">{{ isScanning ? '停止扫描' : '开始扫描' }}</text>
            </view>
            <view class="control-btn" @tap="showMarkerGuide">
                <text class="control-icon">📖</text>
                <text class="control-text">识别指南</text>
            </view>
        </view>

        <view class="marker-library">
            <view class="library-title">📋 可用Marker列表</view>
            <scroll-view scroll-x class="marker-scroll">
                <view class="marker-items">
                    <view 
                        v-for="(marker, index) in availableMarkers" 
                        :key="index"
                        class="marker-item"
                        :class="{ active: detectedMarker?.id === marker.id }"
                        @tap="selectMarker(marker)"
                    >
                        <view class="marker-preview">{{ marker.icon }}</view>
                        <view class="marker-label">{{ marker.name }}</view>
                    </view>
                </view>
            </scroll-view>
        </view>

        <view class="marker-info-panel">
            <view class="info-title">🖼️ AR图片Marker演示</view>
            <view class="info-desc">识别特定图片并触发AR内容展示</view>
            <view class="info-tips">
                <text class="tip-icon">💡</text>
                <text class="tip-text">将Marker图片对准扫描框进行识别</text>
            </view>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            isScanning: false,
            scanProgress: 0,
            scanTimer: null,
            detectedMarker: null,
            availableMarkers: [
                { id: 1, name: '二维码', icon: '📱', description: '扫描二维码获取信息' },
                { id: 2, name: '海报', icon: '🖼️', description: '识别海报展示详情' },
                { id: 3, name: '名片', icon: '💳', description: '识别名片保存联系人' },
                { id: 4, name: '书籍', icon: '📚', description: '识别书籍获取详情' },
                { id: 5, name: '地标', icon: '🗼', description: '识别地标展示介绍' },
                { id: 6, name: '产品', icon: '📦', description: '识别产品展示信息' }
            ]
        };
    },
    onUnload() {
        if (this.scanTimer) {
            clearInterval(this.scanTimer);
        }
    },
    methods: {
        handleBack() {
            uni.navigateBack({
                delta: 1
            });
        },
        toggleScan() {
            this.isScanning = !this.isScanning;
            
            if (this.isScanning) {
                this.startScanning();
            } else {
                this.stopScanning();
            }
        },
        startScanning() {
            this.scanProgress = 0;
            this.detectedMarker = null;
            
            this.scanTimer = setInterval(() => {
                this.scanProgress += 5;
                
                if (this.scanProgress >= 100) {
                    this.stopScanning();
                    this.detectRandomMarker();
                }
            }, 100);
        },
        stopScanning() {
            this.isScanning = false;
            if (this.scanTimer) {
                clearInterval(this.scanTimer);
                this.scanTimer = null;
            }
        },
        detectRandomMarker() {
            const randomMarker = this.availableMarkers[Math.floor(Math.random() * this.availableMarkers.length)];
            this.detectedMarker = randomMarker;
            
            uni.showToast({
                title: `识别成功: ${randomMarker.name}`,
                icon: 'success',
                duration: 2000
            });
        },
        selectMarker(marker) {
            this.detectedMarker = marker;
            uni.showToast({
                title: `已选择: ${marker.name}`,
                icon: 'none',
                duration: 1500
            });
        },
        showMarkerGuide() {
            uni.showModal({
                title: '📖 Marker识别指南',
                content: '1. 确保Marker图片清晰可见\n2. 将Marker放入扫描框内\n3. 保持手机稳定\n4. 等待识别完成',
                showCancel: false,
                confirmText: '知道了'
            });
        }
    }
};
</script>

<style scoped>
.image-marker-container {
    position: relative;
    width: 100%;
    height: 100vh;
    background: linear-gradient(135deg, #2d3436 0%, #1a1a2e 100%);
    overflow: hidden;
}

.marker-header {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    display: flex;
    align-items: center;
    padding: 60rpx 30rpx 30rpx;
    z-index: 100;
}

.back-btn {
    width: 80rpx;
    height: 80rpx;
    display: flex;
    align-items: center;
    justify-content: center;
    background: rgba(255, 255, 255, 0.1);
    border-radius: 50%;
    backdrop-filter: blur(10px);
}

.back-icon {
    font-size: 40rpx;
    color: white;
    font-weight: bold;
}

.header-title {
    flex: 1;
    text-align: center;
    font-size: 34rpx;
    font-weight: bold;
    color: white;
    margin-right: 80rpx;
}

.marker-viewer {
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding-top: 140rpx;
    padding-bottom: 360rpx;
}

.camera-preview {
    width: 90%;
    height: 60%;
    background: linear-gradient(180deg, #333 0%, #111 100%);
    border-radius: 30rpx;
    overflow: hidden;
    position: relative;
}

.marker-scan-area {
    width: 100%;
    height: 100%;
    position: relative;
    display: flex;
    align-items: center;
    justify-content: center;
}

.marker-frame {
    width: 60%;
    height: 60%;
    position: relative;
    border: 3rpx dashed rgba(255, 179, 71, 0.6);
    border-radius: 20rpx;
}

.frame-corner {
    position: absolute;
    width: 60rpx;
    height: 60rpx;
    border: 4rpx solid #ffb347;
}

.frame-corner.top-left {
    top: -3rpx;
    left: -3rpx;
    border-right: none;
    border-bottom: none;
    border-radius: 15rpx 0 0 0;
}

.frame-corner.top-right {
    top: -3rpx;
    right: -3rpx;
    border-left: none;
    border-bottom: none;
    border-radius: 0 15rpx 0 0;
}

.frame-corner.bottom-left {
    bottom: -3rpx;
    left: -3rpx;
    border-right: none;
    border-top: none;
    border-radius: 0 0 0 15rpx;
}

.frame-corner.bottom-right {
    bottom: -3rpx;
    right: -3rpx;
    border-left: none;
    border-top: none;
    border-radius: 0 0 15rpx 0;
}

.frame-crosshair {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 80rpx;
    height: 80rpx;
}

.frame-crosshair::before,
.frame-crosshair::after {
    content: '';
    position: absolute;
    background: #ffb347;
}

.frame-crosshair::before {
    width: 4rpx;
    height: 100%;
    left: 50%;
    transform: translateX(-50%);
}

.frame-crosshair::after {
    width: 100%;
    height: 4rpx;
    top: 50%;
    transform: translateY(-50%);
}

.detected-marker {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    animation: markerAppear 0.5s ease-out;
}

@keyframes markerAppear {
    0% { transform: translate(-50%, -50%) scale(0); opacity: 0; }
    50% { transform: translate(-50%, -50%) scale(1.2); }
    100% { transform: translate(-50%, -50%) scale(1); opacity: 1; }
}

.marker-content {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 40rpx 60rpx;
    background: rgba(255, 179, 71, 0.9);
    border-radius: 20rpx;
    position: relative;
    z-index: 1;
}

.marker-icon {
    font-size: 60rpx;
    margin-bottom: 10rpx;
}

.marker-name {
    font-size: 28rpx;
    font-weight: bold;
    color: #333;
}

.marker-glow {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 100%;
    height: 100%;
    background: #ffb347;
    border-radius: 20rpx;
    filter: blur(30rpx);
    opacity: 0.5;
    animation: glowPulse 1.5s infinite;
}

@keyframes glowPulse {
    0%, 100% { opacity: 0.5; transform: translate(-50%, -50%) scale(1); }
    50% { opacity: 0.8; transform: translate(-50%, -50%) scale(1.1); }
}

.scan-progress {
    position: absolute;
    bottom: 30rpx;
    left: 30rpx;
    right: 30rpx;
}

.progress-bar {
    height: 8rpx;
    background: rgba(255, 255, 255, 0.2);
    border-radius: 4rpx;
    overflow: hidden;
}

.progress-fill {
    height: 100%;
    background: linear-gradient(90deg, #ffb347, #ff6b6b);
    border-radius: 4rpx;
    transition: width 0.1s linear;
}

.progress-text {
    font-size: 22rpx;
    color: #ffb347;
    margin-top: 10rpx;
    text-align: center;
}

.marker-controls {
    position: absolute;
    bottom: 260rpx;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    gap: 40rpx;
}

.control-btn {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 30rpx 60rpx;
    background: rgba(255, 255, 255, 0.1);
    backdrop-filter: blur(10px);
    border-radius: 20rpx;
    border: 2rpx solid rgba(255, 255, 255, 0.2);
    transition: all 0.3s;
}

.control-btn.active {
    background: rgba(255, 179, 71, 0.2);
    border-color: #ffb347;
}

.control-btn:active {
    transform: scale(0.95);
}

.control-icon {
    font-size: 48rpx;
    margin-bottom: 10rpx;
}

.control-text {
    font-size: 26rpx;
    color: white;
}

.marker-library {
    position: absolute;
    bottom: 160rpx;
    left: 30rpx;
    right: 30rpx;
    background: rgba(255, 255, 255, 0.1);
    backdrop-filter: blur(10px);
    border-radius: 20rpx;
    padding: 25rpx;
}

.library-title {
    font-size: 26rpx;
    color: rgba(255, 255, 255, 0.9);
    margin-bottom: 15rpx;
    font-weight: bold;
}

.marker-scroll {
    white-space: nowrap;
}

.marker-items {
    display: inline-flex;
    gap: 20rpx;
}

.marker-item {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 20rpx 30rpx;
    background: rgba(255, 255, 255, 0.05);
    border-radius: 15rpx;
    border: 2rpx solid transparent;
    transition: all 0.3s;
}

.marker-item.active {
    background: rgba(255, 179, 71, 0.2);
    border-color: #ffb347;
}

.marker-preview {
    font-size: 48rpx;
    margin-bottom: 10rpx;
}

.marker-label {
    font-size: 22rpx;
    color: rgba(255, 255, 255, 0.8);
}

.marker-info-panel {
    position: absolute;
    bottom: 30rpx;
    left: 30rpx;
    right: 30rpx;
    background: rgba(255, 255, 255, 0.1);
    backdrop-filter: blur(10px);
    border-radius: 20rpx;
    padding: 30rpx;
}

.info-title {
    font-size: 32rpx;
    font-weight: bold;
    color: white;
    margin-bottom: 10rpx;
}

.info-desc {
    font-size: 26rpx;
    color: rgba(255, 255, 255, 0.7);
    margin-bottom: 20rpx;
}

.info-tips {
    display: flex;
    align-items: center;
}

.tip-icon {
    font-size: 28rpx;
    margin-right: 10rpx;
}

.tip-text {
    font-size: 24rpx;
    color: rgba(255, 255, 255, 0.8);
}
</style>