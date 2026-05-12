<template>
    <view class="ar-container">
        <view v-if="!permissionGranted" class="permission-page">
            <view class="permission-icon">📷</view>
            <view class="permission-title">需要摄像头权限</view>
            <view class="permission-desc">为了提供AR体验，需要获取您的摄像头权限</view>
            <button class="permission-btn" @tap="requestPermission">授权摄像头</button>
            <view class="permission-tips">如果授权失败，请在手机设置中手动开启权限</view>
        </view>
        
        <yn-viewer v-else :showBackBtn="true" @back="handleBack">
            <yn-ar
                disable-scroll
                id="main-frame"
                :width="renderWidth"
                :height="renderHeight"
                :style="'width:' + width + 'px;height:' + height + 'px'"
                :yn_asset="currentModel.url"
                yn_camera_clear_color="0.95 0.95 0.95 1"
                @ready="handleReady"
                @error="handleError"
            />
        </yn-viewer>
        
        <view v-if="showLoading && permissionGranted" class="loading-overlay">
            <view class="loading-spinner"></view>
            <view class="loading-text">{{ loadingText }}</view>
        </view>
        
        <view v-if="permissionGranted && !showLoading" class="control-panel">
            <view class="control-row">
                <view class="control-btn" @tap="scaleModel(1.1)" title="放大">
                    <text class="btn-icon">🔍+</text>
                </view>
                <view class="control-btn" @tap="scaleModel(0.9)" title="缩小">
                    <text class="btn-icon">🔍-</text>
                </view>
                <view class="control-btn" @tap="rotateModel" title="旋转">
                    <text class="btn-icon">🔄</text>
                </view>
                <view class="control-btn" @tap="resetModel" title="重置">
                    <text class="btn-icon">🔁</text>
                </view>
            </view>
        </view>
        
        <view v-if="permissionGranted && !showLoading" class="model-selector">
            <view class="selector-title">选择模型</view>
            <scroll-view class="model-list" scroll-x>
                <view 
                    v-for="model in modelList" 
                    :key="model.id"
                    class="model-item"
                    :class="{ active: currentModel.id === model.id }"
                    @tap="selectModel(model)"
                >
                    <view class="model-thumb">{{ model.icon }}</view>
                    <view class="model-name">{{ model.name }}</view>
                </view>
            </scroll-view>
        </view>
        
        <view v-if="permissionGranted && !showLoading" class="status-bar">
            <view class="status-item">
                <text class="status-label">缩放:</text>
                <text class="status-value">{{ (modelScale * 100).toFixed(0) }}%</text>
            </view>
            <view class="status-item">
                <text class="status-label">旋转:</text>
                <text class="status-value">{{ modelRotation }}°</text>
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
            permissionGranted: false,
            showLoading: true,
            loadingText: '正在启动AR引擎...',
            modelScale: 1,
            modelRotation: 0,
            
            currentModel: {
                id: 1,
                name: '机器人',
                url: 'https://uniapp.anxunxinxi.cn/model/RobotExpressive.glb',
                icon: '🤖'
            },
            
            modelList: [
                { id: 1, name: '机器人', url: 'https://uniapp.anxunxinxi.cn/model/RobotExpressive.glb', icon: '🤖' },
                { id: 2, name: '恐龙', url: 'https://uniapp.anxunxinxi.cn/model/dinosaur.glb', icon: '🦕' },
                { id: 3, name: '汽车', url: 'https://uniapp.anxunxinxi.cn/model/car.glb', icon: '🚗' },
                { id: 4, name: '建筑', url: 'https://uniapp.anxunxinxi.cn/model/building.glb', icon: '🏢' }
            ]
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
                    this.permissionGranted = res.authSetting['scope.camera'] || false;
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
                fail: () => {
                    uni.showModal({
                        title: '权限申请失败',
                        content: '请在手机系统设置中为小程序开启摄像头权限',
                        showCancel: false
                    });
                }
            });
        },
        
        handleReady() {
            this.showLoading = false;
            uni.showToast({ title: 'AR已就绪', icon: 'success', duration: 2000 });
        },
        
        handleError(err) {
            this.showLoading = false;
            console.error('AR Error:', err);
            uni.showToast({ title: '启动失败，请重试', icon: 'error', duration: 2000 });
        },
        
        handleBack() {
            uni.navigateBack({ delta: 1 });
        },
        
        scaleModel(factor) {
            this.modelScale = Math.max(0.5, Math.min(2, this.modelScale * factor));
        },
        
        rotateModel() {
            this.modelRotation = (this.modelRotation + 90) % 360;
        },
        
        resetModel() {
            this.modelScale = 1;
            this.modelRotation = 0;
        },
        
        selectModel(model) {
            this.currentModel = model;
            this.showLoading = true;
            this.loadingText = '正在加载模型...';
            setTimeout(() => {
                this.showLoading = false;
            }, 1500);
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

.permission-icon { font-size: 120rpx; margin-bottom: 40rpx; }
.permission-title { font-size: 40rpx; color: white; font-weight: bold; margin-bottom: 20rpx; }
.permission-desc { font-size: 28rpx; color: rgba(255,255,255,0.8); margin-bottom: 60rpx; text-align: center; padding: 0 40rpx; }
.permission-btn {
    width: 60%; height: 88rpx; background: white; color: #667eea;
    border-radius: 44rpx; font-size: 32rpx; font-weight: bold; border: none;
    margin-bottom: 30rpx;
}
.permission-btn:active { opacity: 0.8; }
.permission-tips { font-size: 24rpx; color: rgba(255,255,255,0.6); text-align: center; padding: 0 40rpx; }

.loading-overlay {
    position: fixed; top: 0; left: 0; right: 0; bottom: 0;
    display: flex; flex-direction: column; align-items: center; justify-content: center;
    background: rgba(0,0,0,0.8); z-index: 1000;
}

.loading-spinner {
    width: 60rpx; height: 60rpx; border: 4rpx solid rgba(255,255,255,0.3);
    border-top-color: white; border-radius: 50%; animation: spin 1s linear infinite;
    margin-bottom: 20rpx;
}

@keyframes spin { to { transform: rotate(360deg); } }
.loading-text { font-size: 28rpx; color: white; }

.control-panel {
    position: fixed; bottom: 200rpx; left: 50%; transform: translateX(-50%);
    z-index: 100;
}

.control-row { display: flex; gap: 20rpx; }

.control-btn {
    width: 80rpx; height: 80rpx; background: rgba(0,0,0,0.6);
    border-radius: 50%; display: flex; align-items: center; justify-content: center;
    backdrop-filter: blur(10px); border: 2rpx solid rgba(255,255,255,0.2);
}

.control-btn:active { background: rgba(0,0,0,0.8); }
.btn-icon { font-size: 32rpx; }

.model-selector {
    position: fixed; top: 120rpx; left: 0; right: 0; z-index: 100;
    padding: 20rpx;
}

.selector-title { font-size: 24rpx; color: rgba(255,255,255,0.8); margin-bottom: 10rpx; text-align: center; }

.model-list { white-space: nowrap; }

.model-item {
    display: inline-flex; flex-direction: column; align-items: center;
    width: 120rpx; padding: 15rpx; margin-right: 20rpx;
    background: rgba(0,0,0,0.5); border-radius: 20rpx;
    border: 2rpx solid transparent;
}

.model-item.active { border-color: #667eea; background: rgba(102,126,234,0.3); }

.model-thumb { font-size: 48rpx; margin-bottom: 10rpx; }
.model-name { font-size: 22rpx; color: white; text-align: center; }

.status-bar {
    position: fixed; top: 60rpx; right: 20rpx; z-index: 100;
    background: rgba(0,0,0,0.6); border-radius: 15rpx;
    padding: 15rpx 20rpx; backdrop-filter: blur(10px);
}

.status-item { display: flex; align-items: center; margin-bottom: 8rpx; }
.status-item:last-child { margin-bottom: 0; }
.status-label { font-size: 22rpx; color: rgba(255,255,255,0.6); margin-right: 10rpx; }
.status-value { font-size: 22rpx; color: white; font-weight: bold; }
</style>