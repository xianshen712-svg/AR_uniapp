<template>
    <view class="ar-page">
        <xr-scene ar-system="modes:Marker" id="xr-scene" bind:ready="handleReady" bind:ar-ready="handleARReady">
            <xr-assets bind:progress="handleAssetsProgress" bind:loaded="handleAssetsLoaded">
                <xr-asset-load type="gltf" asset-id="butterfly" src="https://mmbizwxaminiprogram-1258344707.cos.ap-guangzhou.myqcloud.com/xr-frame/demo/butterfly/index.glb" />
            </xr-assets>
            
            <xr-env env-data="xr-frame-team-workspace-day" />
            
            <xr-node wx:if="{{arReady}}">
                <xr-ar-tracker mode="Marker" src="{{markerImg}}">
                    <xr-gltf model="butterfly" anim-autoplay position="0.2 0 -0.2" scale="0.6 0.6 0.6" rotation="0 -50 0" />
                    <xr-gltf model="butterfly" anim-autoplay position="0.4 0 0.3" scale="0.5 0.5 0.5" rotation="0 -50 0" />
                    <xr-gltf model="butterfly" anim-autoplay position="-0.3 0 0.3" scale="0.4 0.4 0.4" rotation="0 -50 0" />
                </xr-ar-tracker>
                
                <xr-camera id="camera" node-id="camera" position="0.8 2.2 -5" clear-color="0.925 0.925 0.925 1" background="ar" is-ar-camera></xr-camera>
            </xr-node>
            
            <xr-node node-id="lights">
                <xr-light type="ambient" color="1 1 1" intensity="1" />
                <xr-light type="directional" rotation="180 0 0" color="1 1 1" intensity="3" />
            </xr-node>
        </xr-scene>
        
        <view class="overlay">
            <view class="header">
                <view class="back-btn" @tap="handleBack">
                    <text class="back-icon">←</text>
                </view>
                <view class="header-title">AR图片Marker</view>
            </view>
            
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
            scene: null,
            arReady: false,
            assetsLoaded: false,
            markerImg: ''
        };
    },
    methods: {
        handleReady({detail}) {
            this.scene = detail.value;
            console.log('XR Scene Ready');
        },
        handleAssetsProgress({detail}) {
            console.log('Assets Progress:', detail.value);
        },
        handleAssetsLoaded({detail}) {
            this.assetsLoaded = true;
            console.log('Assets Loaded');
        },
        handleARReady({detail}) {
            this.arReady = true;
            console.log('AR Ready');
            uni.showToast({
                title: 'AR已就绪',
                icon: 'success',
                duration: 2000
            });
        },
        handleBack() {
            uni.navigateBack({
                delta: 1
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

xr-scene {
    width: 100%;
    height: 100%;
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

.header {
    display: flex;
    align-items: center;
    padding: 60rpx 30rpx 30rpx;
}

.back-btn {
    width: 80rpx;
    height: 80rpx;
    display: flex;
    align-items: center;
    justify-content: center;
    background: rgba(255, 255, 255, 0.2);
    backdrop-filter: blur(10px);
    border-radius: 50%;
    pointer-events: auto;
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