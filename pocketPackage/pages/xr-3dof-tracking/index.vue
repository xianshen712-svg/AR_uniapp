<template>
    <view class="dof-container">
        <view class="dof-header">
            <view class="back-btn" @tap="handleBack">
                <text class="back-icon">←</text>
            </view>
            <view class="header-title">3DOF追踪</view>
        </view>

        <view class="dof-viewer">
            <view class="scene-container">
                <view 
                    class="target-object"
                    :style="{ 
                        transform: `translate(${objectX}px, ${objectY}px) scale(${objectScale})` 
                    }"
                >
                    <view class="target-ring"></view>
                    <view class="target-core">
                        <text class="target-icon">🎯</text>
                    </view>
                    <view class="target-trail"></view>
                </view>
                <view class="reference-grid">
                    <view v-for="i in 9" :key="i" class="grid-cell"></view>
                </view>
                <view class="motion-indicator">
                    <view class="motion-line x" :style="{ transform: `rotate(${(rotationX % 360)}deg)` }"></view>
                    <view class="motion-line y" :style="{ transform: `rotate(${(rotationY % 360)}deg)` }"></view>
                    <view class="motion-center"></view>
                </view>
            </view>
        </view>

        <view class="dof-controls">
            <view class="control-btn" :class="{ active: isTracking }" @tap="toggleTracking">
                <text class="control-icon">{{ isTracking ? '⏹️' : '🎯' }}</text>
                <text class="control-text">{{ isTracking ? '停止追踪' : '开始追踪' }}</text>
            </view>
            <view class="control-btn" @tap="resetPosition">
                <text class="control-icon">🔃</text>
                <text class="control-text">重置位置</text>
            </view>
        </view>

        <view class="dof-data-panel">
            <view class="data-title">📊 追踪数据</view>
            <view class="data-grid">
                <view class="data-item">
                    <view class="data-label">旋转X</view>
                    <view class="data-value">{{ Math.round(rotationX) }}°</view>
                </view>
                <view class="data-item">
                    <view class="data-label">旋转Y</view>
                    <view class="data-value">{{ Math.round(rotationY) }}°</view>
                </view>
                <view class="data-item">
                    <view class="data-label">旋转Z</view>
                    <view class="data-value">{{ Math.round(rotationZ) }}°</view>
                </view>
                <view class="data-item">
                    <view class="data-label">位置X</view>
                    <view class="data-value">{{ Math.round(objectX) }}px</view>
                </view>
                <view class="data-item">
                    <view class="data-label">位置Y</view>
                    <view class="data-value">{{ Math.round(objectY) }}px</view>
                </view>
                <view class="data-item">
                    <view class="data-label">缩放</view>
                    <view class="data-value">{{ (objectScale * 100).toFixed(0) }}%</view>
                </view>
            </view>
        </view>

        <view class="dof-info-panel">
            <view class="info-title">🎯 3DOF追踪演示</view>
            <view class="info-desc">3自由度头部追踪效果展示</view>
            <view class="info-tips">
                <text class="tip-icon">💡</text>
                <text class="tip-text">移动设备来体验3DOF追踪效果</text>
            </view>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            isTracking: false,
            trackingTimer: null,
            rotationX: 0,
            rotationY: 0,
            rotationZ: 0,
            objectX: 0,
            objectY: 0,
            objectScale: 1,
            baseTime: Date.now()
        };
    },
    onUnload() {
        if (this.trackingTimer) {
            clearInterval(this.trackingTimer);
        }
    },
    methods: {
        handleBack() {
            uni.navigateBack({
                delta: 1
            });
        },
        toggleTracking() {
            this.isTracking = !this.isTracking;
            
            if (this.isTracking) {
                this.startTracking();
            } else {
                this.stopTracking();
            }
        },
        startTracking() {
            this.baseTime = Date.now();
            
            this.trackingTimer = setInterval(() => {
                const elapsed = (Date.now() - this.baseTime) / 1000;
                
                this.rotationX = Math.sin(elapsed * 0.5) * 30;
                this.rotationY = Math.cos(elapsed * 0.7) * 45;
                this.rotationZ = Math.sin(elapsed * 0.3) * 15;
                
                this.objectX = Math.sin(elapsed * 0.8) * 50;
                this.objectY = Math.cos(elapsed * 0.6) * 30;
                this.objectScale = 1 + Math.sin(elapsed) * 0.2;
            }, 30);
        },
        stopTracking() {
            if (this.trackingTimer) {
                clearInterval(this.trackingTimer);
                this.trackingTimer = null;
            }
        },
        resetPosition() {
            this.rotationX = 0;
            this.rotationY = 0;
            this.rotationZ = 0;
            this.objectX = 0;
            this.objectY = 0;
            this.objectScale = 1;
            this.baseTime = Date.now();
            
            uni.showToast({
                title: '已重置位置',
                icon: 'success',
                duration: 1500
            });
        }
    }
};
</script>

<style scoped>
.dof-container {
    position: relative;
    width: 100%;
    height: 100vh;
    background: linear-gradient(135deg, #0c2461 0%, #1e3799 100%);
    overflow: hidden;
}

.dof-header {
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

.dof-viewer {
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding-top: 140rpx;
    padding-bottom: 340rpx;
}

.scene-container {
    width: 80%;
    height: 70%;
    position: relative;
    background: radial-gradient(circle at center, rgba(135, 206, 235, 0.1) 0%, transparent 70%);
    border-radius: 30rpx;
    overflow: hidden;
}

.target-object {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    transition: transform 0.1s ease-out;
}

.target-ring {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 120rpx;
    height: 120rpx;
    border: 4rpx solid rgba(135, 206, 235, 0.6);
    border-radius: 50%;
    animation: ringPulse 2s infinite;
}

@keyframes ringPulse {
    0%, 100% { transform: translate(-50%, -50%) scale(1); opacity: 0.6; }
    50% { transform: translate(-50%, -50%) scale(1.3); opacity: 0.2; }
}

.target-core {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 80rpx;
    height: 80rpx;
    background: linear-gradient(135deg, #87ceeb, #0c2461);
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    box-shadow: 0 0 30rpx rgba(135, 206, 235, 0.5);
}

.target-icon {
    font-size: 40rpx;
}

.target-trail {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 160rpx;
    height: 160rpx;
    border: 2rpx dashed rgba(135, 206, 235, 0.3);
    border-radius: 50%;
}

.reference-grid {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 300rpx;
    height: 300rpx;
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: repeat(3, 1fr);
    gap: 10rpx;
}

.grid-cell {
    background: rgba(255, 255, 255, 0.05);
    border-radius: 8rpx;
}

.motion-indicator {
    position: absolute;
    top: 30rpx;
    right: 30rpx;
    width: 100rpx;
    height: 100rpx;
    position: relative;
}

.motion-line {
    position: absolute;
    top: 50%;
    left: 0;
    width: 100%;
    height: 4rpx;
    transform-origin: center center;
    transition: transform 0.1s ease-out;
}

.motion-line.x {
    background: linear-gradient(90deg, transparent, #ff6b6b, transparent);
}

.motion-line.y {
    background: linear-gradient(90deg, transparent, #4ecdc4, transparent);
    transform: rotate(90deg);
}

.motion-center {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 16rpx;
    height: 16rpx;
    background: #fff;
    border-radius: 50%;
    box-shadow: 0 0 10rpx #fff;
}

.dof-controls {
    position: absolute;
    bottom: 280rpx;
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
    background: rgba(135, 206, 235, 0.2);
    border-color: #87ceeb;
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

.dof-data-panel {
    position: absolute;
    bottom: 160rpx;
    left: 30rpx;
    right: 30rpx;
    background: rgba(255, 255, 255, 0.1);
    backdrop-filter: blur(10px);
    border-radius: 20rpx;
    padding: 25rpx;
}

.data-title {
    font-size: 26rpx;
    color: rgba(255, 255, 255, 0.9);
    margin-bottom: 15rpx;
    font-weight: bold;
}

.data-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 15rpx;
}

.data-item {
    text-align: center;
    padding: 15rpx;
    background: rgba(255, 255, 255, 0.05);
    border-radius: 12rpx;
}

.data-label {
    font-size: 20rpx;
    color: rgba(255, 255, 255, 0.6);
    margin-bottom: 8rpx;
}

.data-value {
    font-size: 26rpx;
    color: white;
    font-weight: bold;
}

.dof-info-panel {
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