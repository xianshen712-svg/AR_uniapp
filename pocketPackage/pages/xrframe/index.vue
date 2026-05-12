4<template>
    <view class="ar-container">
        <view v-if="!permissionGranted" class="permission-page">
            <view class="permission-icon">📷</view>
            <view class="permission-title">需要摄像头权限</view>
            <view class="permission-desc">为了提供AR体验，需要获取您的摄像头权限</view>
            <button class="permission-btn" @tap="requestPermission">
                授权摄像头
            </button>
            <view class="permission-tips">
                如果授权失败，请在手机设置中手动开启权限
            </view>
        </view>
        
        <yn-viewer v-else :showBackBtn="true" @back="handleBack">
            <yn-ar
                disable-scroll
                id="main-frame"
                :width="renderWidth"
                :height="renderHeight"
                :style="'width:' + width + 'px;height:' + height + 'px'"
                yn_asset="https://uniapp.anxunxinxi.cn/model/RobotExpressive.glb"
                yn_camera_clear_color="0.89 0.69 0.94 1"
                @ready="handleReady"
                @error="handleError"
            />
        </yn-viewer>
        
        <view v-if="showLoading && permissionGranted" class="loading-overlay">
            <view class="loading-spinner"></view>
            <view class="loading-text">{{ loadingText }}</view>
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
            permissionGranted: false,
            showLoading: true,
            loadingText: '正在启动AR引擎...'
        };
    },
    onLoad() {
        const info = uni.getSystemInfoSync();
        this.width = info.windowWidth;
        this.height = info.windowHeight;
        this.renderWidth = this.width * info.pixelRatio;
        this.renderHeight = this.height * info.pixelRatio;
        
        this.checkPermission();
    },
    methods: {
        checkPermission() {
            uni.getSetting({
                success: (res) => {
                    if (res.authSetting['scope.camera']) {
                        this.permissionGranted = true;
                        this.loadingText = '正在启动AR引擎...';
                    } else {
                        this.permissionGranted = false;
                    }
                },
                fail: () => {
                    this.permissionGranted = false;
                }
            });
        },
        
        requestPermission() {
            uni.authorize({
                scope: 'scope.camera',
                success: () => {
                    this.permissionGranted = true;
                    this.showLoading = true;
                    this.loadingText = '正在启动AR引擎...';
                },
                fail: (err) => {
                    uni.showModal({
                        title: '权限申请失败',
                        content: '请在手机系统设置中为小程序开启摄像头权限',
                        showCancel: false,
                        confirmText: '知道了'
                    });
                }
            });
        },
        
        handleReady() {
            this.showLoading = false;
            uni.showToast({
                title: 'AR已就绪',
                icon: 'success',
                duration: 2000
            });
        },
        
        handleError(err) {
            this.showLoading = false;
            console.error('AR Error:', err);
            uni.showToast({
                title: '启动失败，请重试',
                icon: 'error',
                duration: 2000
            });
        },
        
        handleBack() {
            uni.navigateBack({ delta: 1 });
        }
    }
};
</script>

<style scoped>
.ar-container {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: #000;
}

.permission-page {
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.permission-icon {
    font-size: 120rpx;
    margin-bottom: 40rpx;
}

.permission-title {
    font-size: 40rpx;
    color: white;
    font-weight: bold;
    margin-bottom: 20rpx;
}

.permission-desc {
    font-size: 28rpx;
    color: rgba(255, 255, 255, 0.8);
    margin-bottom: 60rpx;
    text-align: center;
    padding: 0 40rpx;
}

.permission-btn {
    width: 60%;
    height: 88rpx;
    background: white;
    color: #667eea;
    border-radius: 44rpx;
    font-size: 32rpx;
    font-weight: bold;
    border: none;
    margin-bottom: 30rpx;
}

.permission-btn:active {
    opacity: 0.8;
}

.permission-tips {
    font-size: 24rpx;
    color: rgba(255, 255, 255, 0.6);
    text-align: center;
    padding: 0 40rpx;
}

.loading-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    background: rgba(0, 0, 0, 0.8);
    z-index: 1000;
}

.loading-spinner {
    width: 60rpx;
    height: 60rpx;
    border: 4rpx solid rgba(255, 255, 255, 0.3);
    border-top-color: white;
    border-radius: 50%;
    animation: spin 1s linear infinite;
    margin-bottom: 20rpx;
}

@keyframes spin {
    to { transform: rotate(360deg); }
}

.loading-text {
    font-size: 28rpx;
    color: white;
}
</style>