<template>
    <view class="ar-page">
        <xr-scene id="xr-scene" bind:ready="handleReady">
            <xr-assets bind:progress="handleAssetsProgress" bind:loaded="handleAssetsLoaded">
                <xr-asset-material asset-id="video-mat" effect="standard" />
                <xr-asset-load type="texture" asset-id="video-tex" src="https://mmbizwxaminiprogram-1258344707.cos.ap-guangzhou.myqcloud.com/xr-frame/demo/video.mp4" />
            </xr-assets>
            
            <xr-node>
                <xr-mesh node-id="video-screen" position="0 0 -3" scale="4 2.25 1" geometry="plane" material="video-mat" uniforms="u_baseColorMap:video-tex" states="cullOn:false" />
                
                <xr-mesh node-id="cube1" position="-2 0 0" scale="1 1 1" rotation="0 45 0" geometry="cube" material="video-mat" uniforms="u_baseColorMap:video-tex" />
                <xr-mesh node-id="cube2" position="2 0 0" scale="1 1 1" rotation="0 -45 0" geometry="cube" material="video-mat" uniforms="u_baseColorMap:video-tex" />
                
                <xr-camera id="camera" node-id="camera" position="0 2 8" clear-color="0.1 0.1 0.1 1" camera-orbit-control>
                </xr-camera>
            </xr-node>
            
            <xr-node node-id="lights">
                <xr-light type="ambient" color="1 1 1" intensity="1" />
                <xr-light type="directional" rotation="40 170 0" color="1 1 1" intensity="3" />
            </xr-node>
        </xr-scene>
        
        <view class="overlay">
            <view class="header">
                <view class="back-btn" @tap="handleBack">
                    <text class="back-icon">←</text>
                </view>
                <view class="header-title">视频纹理</view>
            </view>
            
            <view class="info-panel" v-if="assetsLoaded">
                <view class="info-text">🎬 视频纹理映射演示</view>
                <view class="hint-text">拖动屏幕旋转视角</view>
            </view>
            
            <view class="loading-panel" v-if="!assetsLoaded">
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
            scene: null,
            assetsLoaded: false
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
            uni.showToast({
                title: '资源加载完成',
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