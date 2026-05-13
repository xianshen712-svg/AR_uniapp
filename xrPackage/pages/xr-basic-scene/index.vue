<template>
    <view class="xr-page">
        <yn-viewer :showBackBtn="true" @back="handleBack">
            <yn-3d
                id="basic-scene"
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

        <!-- 信息面板 -->
        <view class="info-panel" v-if="showInfo">
            <view class="info-title">🎲 基础3D场景演示</view>
            <view class="info-desc">立方体、灯光、纹理、环境天空盒</view>
            <view class="info-features">
                <view class="feature-item">
                    <text class="feature-icon">📦</text>
                    <text class="feature-text">PBR材质立方体</text>
                </view>
                <view class="feature-item">
                    <text class="feature-icon">💡</text>
                    <text class="feature-text">环境光+平行光</text>
                </view>
                <view class="feature-item">
                    <text class="feature-icon">🖼️</text>
                    <text class="feature-text">纹理贴图</text>
                </view>
            </view>
            <view class="info-tips">
                <text class="tip-icon">💡</text>
                <text class="tip-text">拖动屏幕旋转视角，双指缩放</text>
            </view>
        </view>

        <!-- 加载状态 -->
        <view class="loading-overlay" v-if="isLoading">
            <view class="loading-spinner"></view>
            <text class="loading-text">正在加载3D场景...</text>
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
            showInfo: false
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
        handleReady() {
            this.isLoading = false;
            this.showInfo = true;
            uni.showToast({
                title: '3D场景加载完成',
                icon: 'success',
                duration: 2000
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
    margin-bottom: 20rpx;
}

.info-features {
    display: flex;
    flex-wrap: wrap;
    gap: 16rpx;
    margin-bottom: 20rpx;
}

.feature-item {
    display: flex;
    align-items: center;
    background: rgba(102, 126, 234, 0.1);
    padding: 12rpx 20rpx;
    border-radius: 30rpx;
}

.feature-icon {
    font-size: 28rpx;
    margin-right: 10rpx;
}

.feature-text {
    font-size: 22rpx;
    color: #667eea;
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
