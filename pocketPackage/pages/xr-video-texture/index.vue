<template>
    <view class="ar-page">
        <yn-viewer :showBackBtn="true" @back="handleBack">
            <yn-ar
                id="ar-video"
                :width="renderWidth"
                :height="renderHeight"
                :style="'width:' + width + 'px;height:' + height + 'px'"
                yn_ar_mode="Video"
                yn_video_url="https://uniapp.anxunxinxi.cn/video/demo.mp4"
                yn_camera_clear_color="0.1 0.1 0.1 1"
                @arReady="handleARReady"
            />
        </yn-viewer>
        
        <view class="overlay" v-if="showOverlay">
            <view class="info-panel" v-if="arReady">
                <view class="info-text">🎬 视频纹理映射演示</view>
                <view class="hint-text">拖动屏幕旋转视角</view>
            </view>
            <view class="loading-panel" v-if="!arReady">
                <view class="loading-spinner"></view>
                <view class="loading-text">正在加载资源...</view>
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
            showOverlay: true
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
                title: '资源加载完成',
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

.hint-text {
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