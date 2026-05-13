<template>
    <view class="ar-page">
        <yn-viewer :showBackBtn="true" @back="handleBack">
            <yn-ar
                id="ar-marker"
                :width="renderWidth"
                :height="renderHeight"
                :style="'width:' + width + 'px;height:' + height + 'px'"
                yn_ar_mode="Marker"
                yn_marker_url="https://uniapp.anxunxinxi.cn/marker/marker.png"
                yn_asset="https://uniapp.anxunxinxi.cn/model/butterfly.glb"
                yn_camera_clear_color="0.925 0.925 0.925 1"
                @arReady="handleARReady"
                @markerDetected="handleMarkerDetected"
            />
        </yn-viewer>
        
        <view class="overlay" v-if="showOverlay">
            <view class="info-panel" v-if="arReady">
                <view class="info-text">📱 扫描Marker图片</view>
                <view class="marker-guide">
                    <text class="guide-icon">💡</text>
                    <text class="guide-text">将Marker图片对准摄像头</text>
                </view>
            </view>
            <view class="loading-panel" v-if="!arReady">
                <view class="loading-spinner"></view>
                <view class="loading-text">正在启动AR...</view>
            </view>
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
            showOverlay: true,
            detectedMarker: null
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
                duration: 2000
            });
        },
        handleMarkerDetected(e) {
            this.detectedMarker = e.detail;
            uni.showToast({
                title: 'Marker识别成功',
                icon: 'success',
                duration: 2000
            });
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

.overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    pointer-events: none;
    z-index: 100;
}

.info-panel {
    position: absolute;
    bottom: 60rpx;
    left: 50%;
    transform: translateX(-50%);
    background: rgba(0, 0, 0, 0.6);
    backdrop-filter: blur(10px);
    padding: 30rpx 60rpx;
    border-radius: 40rpx;
    display: flex;
    flex-direction: column;
    align-items: center;
}

.info-text {
    font-size: 28rpx;
    color: white;
    margin-bottom: 10rpx;
}

.marker-guide {
    display: flex;
    align-items: center;
}

.guide-icon {
    font-size: 24rpx;
    margin-right: 10rpx;
}

.guide-text {
    font-size: 24rpx;
    color: rgba(255, 255, 255, 0.8);
}

.loading-panel {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    display: flex;
    flex-direction: column;
    align-items: center;
}

.loading-spinner {
    width: 60rpx;
    height: 60rpx;
    border: 4rpx solid rgba(255, 255, 255, 0.3);
    border-top-color: white;
    border-radius: 50%;
    animation: spin 1s linear infinite;
}

@keyframes spin {
    to { transform: rotate(360deg); }
}

.loading-text {
    margin-top: 20rpx;
    font-size: 28rpx;
    color: white;
}
</style>