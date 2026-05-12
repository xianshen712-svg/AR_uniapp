<template>
    <view class="model-scene-container">
        <view class="model-header">
            <view class="back-btn" @tap="handleBack">
                <text class="back-icon">←</text>
            </view>
            <view class="header-title">glTF模型展示</view>
        </view>

        <view class="model-viewer">
            <view class="model-canvas">
                <view class="model-3d-container">
                    <view class="model-3d" :style="{ transform: `rotateY(${rotationY}deg) rotateX(${rotationX}deg)` }">
                        <view class="model-box">
                            <view class="face front">
                                <text class="face-icon">🤖</text>
                            </view>
                            <view class="face back">
                                <text class="face-icon">⚙️</text>
                            </view>
                            <view class="face left">
                                <text class="face-icon">🔧</text>
                            </view>
                            <view class="face right">
                                <text class="face-icon">🛠️</text>
                            </view>
                            <view class="face top">
                                <text class="face-icon">🔩</text>
                            </view>
                            <view class="face bottom">
                                <text class="face-icon">⚡</text>
                            </view>
                        </view>
                    </view>
                    <view class="model-shadow"></view>
                </view>
            </view>

            <view class="model-info">
                <view class="model-name">RobotExpressive</view>
                <view class="model-format">glTF 2.0</view>
            </view>
        </view>

        <view class="model-controls">
            <view class="control-row">
                <view class="control-item">
                    <text class="control-label">旋转速度</text>
                    <slider 
                        :value="rotateSpeed" 
                        @change="onRotateSpeedChange" 
                        :min="0" 
                        :max="10" 
                        :step="1"
                        activeColor="#667eea"
                        backgroundColor="#ddd"
                        block-size="20"
                    />
                </view>
            </view>
            <view class="control-row">
                <view class="control-btn" @tap="toggleAutoRotate">
                    <text class="control-icon">{{ isAutoRotate ? '⏹️' : '🔄' }}</text>
                    <text class="control-text">{{ isAutoRotate ? '停止旋转' : '自动旋转' }}</text>
                </view>
                <view class="control-btn" @tap="resetView">
                    <text class="control-icon">🔃</text>
                    <text class="control-text">重置视角</text>
                </view>
                <view class="control-btn" @tap="loadModel">
                    <text class="control-icon">📥</text>
                    <text class="control-text">加载模型</text>
                </view>
            </view>
        </view>

        <view class="model-info-panel">
            <view class="info-title">🤖 glTF模型展示演示</view>
            <view class="info-desc">加载和展示glTF 3D模型</view>
            <view class="info-tips">
                <text class="tip-icon">💡</text>
                <text class="tip-text">拖动屏幕旋转模型视角</text>
            </view>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            rotationX: -15,
            rotationY: 45,
            rotateSpeed: 5,
            isAutoRotate: true,
            animationId: null,
            touchStartX: 0,
            touchStartY: 0
        };
    },
    onLoad() {
        this.startAutoRotate();
    },
    onUnload() {
        if (this.animationId) {
            cancelAnimationFrame(this.animationId);
        }
    },
    methods: {
        handleBack() {
            uni.navigateBack({
                delta: 1
            });
        },
        startAutoRotate() {
            const animate = () => {
                if (this.isAutoRotate) {
                    this.rotationY += this.rotateSpeed * 0.1;
                }
                this.animationId = requestAnimationFrame(animate);
            };
            animate();
        },
        toggleAutoRotate() {
            this.isAutoRotate = !this.isAutoRotate;
        },
        onRotateSpeedChange(e) {
            this.rotateSpeed = e.detail.value;
        },
        resetView() {
            this.rotationX = -15;
            this.rotationY = 45;
        },
        loadModel() {
            uni.showLoading({
                title: '加载模型中...'
            });
            
            setTimeout(() => {
                uni.hideLoading();
                uni.showToast({
                    title: '模型加载完成',
                    icon: 'success',
                    duration: 2000
                });
            }, 1500);
        },
        handleTouchStart(e) {
            this.touchStartX = e.touches[0].clientX;
            this.touchStartY = e.touches[0].clientY;
        },
        handleTouchMove(e) {
            const deltaX = e.touches[0].clientX - this.touchStartX;
            const deltaY = e.touches[0].clientY - this.touchStartY;
            
            this.rotationY += deltaX * 0.5;
            this.rotationX -= deltaY * 0.5;
            
            this.touchStartX = e.touches[0].clientX;
            this.touchStartY = e.touches[0].clientY;
        }
    }
};
</script>

<style scoped>
.model-scene-container {
    position: relative;
    width: 100%;
    height: 100vh;
    background: linear-gradient(135deg, #2d3436 0%, #000 100%);
    overflow: hidden;
}

.model-header {
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

.model-viewer {
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding-top: 140rpx;
    padding-bottom: 380rpx;
}

.model-canvas {
    width: 60%;
    height: 60%;
    display: flex;
    align-items: center;
    justify-content: center;
    perspective: 1000px;
}

.model-3d-container {
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    perspective: 1000px;
}

.model-3d {
    width: 200rpx;
    height: 200rpx;
    transform-style: preserve-3d;
    transition: transform 0.1s ease-out;
}

.model-box {
    width: 100%;
    height: 100%;
    position: relative;
    transform-style: preserve-3d;
}

.face {
    position: absolute;
    width: 200rpx;
    height: 200rpx;
    display: flex;
    align-items: center;
    justify-content: center;
    border: 3rpx solid rgba(255, 255, 255, 0.3);
    border-radius: 10rpx;
    backface-visibility: visible;
}

.face-icon {
    font-size: 60rpx;
}

.front {
    background: linear-gradient(135deg, rgba(102, 126, 234, 0.8), rgba(118, 75, 162, 0.8));
    transform: translateZ(100rpx);
}

.back {
    background: linear-gradient(135deg, rgba(78, 205, 196, 0.8), rgba(69, 183, 209, 0.8));
    transform: rotateY(180deg) translateZ(100rpx);
}

.left {
    background: linear-gradient(135deg, rgba(255, 107, 107, 0.8), rgba(255, 179, 71, 0.8));
    transform: rotateY(-90deg) translateZ(100rpx);
}

.right {
    background: linear-gradient(135deg, rgba(150, 206, 180, 0.8), rgba(221, 160, 221, 0.8));
    transform: rotateY(90deg) translateZ(100rpx);
}

.top {
    background: linear-gradient(135deg, rgba(135, 206, 235, 0.8), rgba(102, 126, 234, 0.8));
    transform: rotateX(90deg) translateZ(100rpx);
}

.bottom {
    background: linear-gradient(135deg, rgba(247, 220, 111, 0.8), rgba(255, 179, 71, 0.8));
    transform: rotateX(-90deg) translateZ(100rpx);
}

.model-shadow {
    width: 160rpx;
    height: 20rpx;
    background: rgba(0, 0, 0, 0.3);
    border-radius: 50%;
    margin-top: 40rpx;
    filter: blur(10px);
}

.model-info {
    text-align: center;
    margin-top: 40rpx;
}

.model-name {
    font-size: 32rpx;
    font-weight: bold;
    color: white;
}

.model-format {
    font-size: 24rpx;
    color: rgba(255, 255, 255, 0.6);
    margin-top: 10rpx;
}

.model-controls {
    position: absolute;
    bottom: 160rpx;
    left: 30rpx;
    right: 30rpx;
    background: rgba(255, 255, 255, 0.1);
    backdrop-filter: blur(10px);
    border-radius: 20rpx;
    padding: 30rpx;
}

.control-row {
    display: flex;
    gap: 20rpx;
    margin-bottom: 20rpx;
}

.control-row:last-child {
    margin-bottom: 0;
}

.control-item {
    flex: 1;
}

.control-label {
    font-size: 24rpx;
    color: rgba(255, 255, 255, 0.8);
    margin-bottom: 15rpx;
    display: block;
}

.control-btn {
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 25rpx 20rpx;
    background: rgba(255, 255, 255, 0.1);
    border-radius: 15rpx;
    border: 2rpx solid rgba(255, 255, 255, 0.2);
    transition: all 0.3s;
}

.control-btn:active {
    transform: scale(0.95);
    background: rgba(255, 255, 255, 0.2);
}

.control-icon {
    font-size: 40rpx;
    margin-bottom: 10rpx;
}

.control-text {
    font-size: 22rpx;
    color: white;
    text-align: center;
}

.model-info-panel {
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