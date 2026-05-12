<template>
    <view class="ar-scene-container">
        <view class="ar-header">
            <view class="back-btn" @tap="handleBack">
                <text class="back-icon">←</text>
            </view>
            <view class="header-title">AR平面检测</view>
        </view>

        <view class="ar-viewer">
            <view class="camera-preview">
                <view class="scan-lines"></view>
                <view class="scan-frame">
                    <view class="corner top-left"></view>
                    <view class="corner top-right"></view>
                    <view class="corner bottom-left"></view>
                    <view class="corner bottom-right"></view>
                </view>
                <view class="plane-indicator" v-if="detectedPlanes.length > 0">
                    <view 
                        v-for="(plane, index) in detectedPlanes" 
                        :key="index"
                        class="plane-marker"
                        :style="{ 
                            left: plane.x + '%', 
                            top: plane.y + '%',
                            width: plane.size + 'rpx',
                            height: plane.size + 'rpx'
                        }"
                    >
                        <view class="plane-dot"></view>
                        <view class="plane-label">平面 {{ index + 1 }}</view>
                    </view>
                </view>
            </view>
        </view>

        <view class="ar-controls">
            <view :class="getScanBtnClass()" @tap="toggleScan">
                <text class="control-icon">{{ isScanning ? '⏹️' : '🔍' }}</text>
                <text class="control-text">{{ isScanning ? '停止扫描' : '开始扫描' }}</text>
            </view>
            <view class="control-btn" @tap="placeObject">
                <text class="control-icon">📦</text>
                <text class="control-text">放置物体</text>
            </view>
        </view>

        <view class="ar-info-panel">
            <view class="info-title">📱 AR平面检测演示</view>
            <view class="info-desc">识别水平面并放置3D物体</view>
            <view class="info-status">
                <text class="status-icon">{{ isScanning ? '🟢' : '🔴' }}</text>
                <text class="status-text">{{ isScanning ? '正在扫描平面...' : '点击开始扫描' }}</text>
            </view>
            <view class="info-tips">
                <text class="tip-icon">💡</text>
                <text class="tip-text">将手机对准水平面进行扫描</text>
            </view>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            isScanning: false,
            detectedPlanes: [],
            scanTimer: null,
            placedObjects: []
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
        getScanBtnClass() {
            return this.isScanning ? 'control-btn active' : 'control-btn';
        },
        startScanning() {
            this.detectedPlanes = [];
            let count = 0;
            
            this.scanTimer = setInterval(() => {
                if (count < 3) {
                    const plane = {
                        x: 30 + Math.random() * 40,
                        y: 40 + Math.random() * 30,
                        size: 80 + Math.random() * 40
                    };
                    this.detectedPlanes.push(plane);
                    count++;
                    
                    uni.showToast({
                        title: `发现平面 ${count}`,
                        icon: 'success',
                        duration: 1000
                    });
                } else {
                    this.stopScanning();
                    uni.showToast({
                        title: '扫描完成',
                        icon: 'success',
                        duration: 2000
                    });
                }
            }, 1500);
        },
        stopScanning() {
            this.isScanning = false;
            if (this.scanTimer) {
                clearInterval(this.scanTimer);
                this.scanTimer = null;
            }
        },
        placeObject() {
            if (this.detectedPlanes.length === 0) {
                uni.showToast({
                    title: '请先扫描平面',
                    icon: 'none',
                    duration: 2000
                });
                return;
            }
            
            const randomPlane = this.detectedPlanes[Math.floor(Math.random() * this.detectedPlanes.length)];
            const object = {
                id: Date.now(),
                x: randomPlane.x,
                y: randomPlane.y,
                type: ['🚀', '🏠', '🚗', '🎁', '⭐'][Math.floor(Math.random() * 5)]
            };
            
            this.placedObjects.push(object);
            
            uni.showToast({
                title: '物体已放置',
                icon: 'success',
                duration: 1500
            });
        }
    }
};
</script>

<style scoped>
.ar-scene-container {
    position: relative;
    width: 100%;
    height: 100vh;
    background: linear-gradient(135deg, #0f3460 0%, #16213e 100%);
    overflow: hidden;
}

.ar-header {
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

.ar-viewer {
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    padding-top: 140rpx;
    padding-bottom: 280rpx;
}

.camera-preview {
    width: 100%;
    height: 100%;
    background: linear-gradient(180deg, #2d3436 0%, #000 100%);
    position: relative;
    border-radius: 30rpx;
    overflow: hidden;
}

.scan-lines {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: repeating-linear-gradient(
        0deg,
        transparent,
        transparent 2px,
        rgba(0, 255, 255, 0.03) 2px,
        rgba(0, 255, 255, 0.03) 4px
    );
    pointer-events: none;
}

.scan-frame {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 60%;
    height: 60%;
    border: 2rpx solid rgba(0, 255, 255, 0.5);
    border-radius: 20rpx;
}

.corner {
    position: absolute;
    width: 40rpx;
    height: 40rpx;
    border: 6rpx solid #00ffff;
}

.corner.top-left {
    top: -3rpx;
    left: -3rpx;
    border-right: none;
    border-bottom: none;
    border-radius: 10rpx 0 0 0;
}

.corner.top-right {
    top: -3rpx;
    right: -3rpx;
    border-left: none;
    border-bottom: none;
    border-radius: 0 10rpx 0 0;
}

.corner.bottom-left {
    bottom: -3rpx;
    left: -3rpx;
    border-right: none;
    border-top: none;
    border-radius: 0 0 0 10rpx;
}

.corner.bottom-right {
    bottom: -3rpx;
    right: -3rpx;
    border-left: none;
    border-top: none;
    border-radius: 0 0 10rpx 0;
}

.plane-indicator {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
}

.plane-marker {
    position: absolute;
    display: flex;
    flex-direction: column;
    align-items: center;
    transform: translate(-50%, -50%);
}

.plane-dot {
    width: 30rpx;
    height: 30rpx;
    background: #00ff00;
    border-radius: 50%;
    box-shadow: 0 0 20rpx #00ff00;
    animation: pulse 1s infinite;
}

@keyframes pulse {
    0%, 100% {
        transform: scale(1);
        opacity: 1;
    }
    50% {
        transform: scale(1.5);
        opacity: 0.5;
    }
}

.plane-label {
    font-size: 22rpx;
    color: #00ff00;
    margin-top: 10rpx;
    text-shadow: 0 0 10rpx #00ff00;
}

.ar-controls {
    position: absolute;
    bottom: 200rpx;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    gap: 40rpx;
}

.control-btn {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 30rpx 50rpx;
    background: rgba(255, 255, 255, 0.1);
    backdrop-filter: blur(10px);
    border-radius: 20rpx;
    border: 2rpx solid rgba(255, 255, 255, 0.2);
    transition: all 0.3s;
}

.control-btn.active {
    background: rgba(0, 255, 255, 0.2);
    border-color: #00ffff;
}

.control-btn:active {
    transform: scale(0.95);
}

.control-icon {
    font-size: 48rpx;
    margin-bottom: 10rpx;
}

.control-text {
    font-size: 24rpx;
    color: white;
}

.ar-info-panel {
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
    margin-bottom: 15rpx;
}

.info-status {
    display: flex;
    align-items: center;
    margin-bottom: 15rpx;
}

.status-icon {
    font-size: 28rpx;
    margin-right: 10rpx;
}

.status-text {
    font-size: 24rpx;
    color: rgba(255, 255, 255, 0.8);
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