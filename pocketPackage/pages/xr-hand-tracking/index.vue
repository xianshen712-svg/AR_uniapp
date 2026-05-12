<template>
    <view class="hand-tracking-container">
        <view class="hand-header">
            <view class="back-btn" @tap="handleBack">
                <text class="back-icon">←</text>
            </view>
            <view class="header-title">手部动作识别</view>
        </view>

        <view class="hand-viewer">
            <view class="camera-preview">
                <view class="scan-area">
                    <view class="hand-detection">
                        <view 
                            v-for="(finger, index) in fingers" 
                            :key="index"
                            class="finger-point"
                            :style="{ 
                                left: finger.x + '%', 
                                top: finger.y + '%',
                                opacity: isDetecting ? 1 : 0.3
                            }"
                        >
                            <view class="finger-dot"></view>
                            <view class="finger-label">{{ finger.name }}</view>
                        </view>
                        <view class="hand-center" :style="{ opacity: isDetecting ? 1 : 0.3 }">
                            <view class="palm-icon">🖐️</view>
                            <view class="palm-label">手掌中心</view>
                        </view>
                    </view>
                    <view class="scan-frame">
                        <view class="corner top-left"></view>
                        <view class="corner top-right"></view>
                        <view class="corner bottom-left"></view>
                        <view class="corner bottom-right"></view>
                    </view>
                    <view class="scan-lines" v-if="isDetecting"></view>
                </view>
            </view>

            <view class="gesture-display">
                <view class="gesture-icon">{{ currentGesture.icon }}</view>
                <view class="gesture-name">{{ currentGesture.name }}</view>
                <view class="gesture-desc">{{ currentGesture.description }}</view>
            </view>
        </view>

        <view class="hand-controls">
            <view :class="getDetectionBtnClass()" @tap="toggleDetection">
                <text class="control-icon">{{ isDetecting ? '⏹️' : '🔍' }}</text>
                <text class="control-text">{{ isDetecting ? '停止识别' : '开始识别' }}</text>
            </view>
            <view class="control-btn" @tap="clearDetection">
                <text class="control-icon">🧹</text>
                <text class="control-text">清除</text>
            </view>
        </view>

        <view class="gesture-list">
            <view class="list-title">已识别手势</view>
            <view class="gesture-items">
                <view 
                    v-for="(gesture, index) in gestureHistory" 
                    :key="index"
                    class="gesture-item"
                >
                    <text class="gesture-emoji">{{ gesture.icon }}</text>
                    <text class="gesture-text">{{ gesture.name }}</text>
                    <text class="gesture-time">{{ gesture.time }}</text>
                </view>
            </view>
        </view>

        <view class="hand-info-panel">
            <view class="info-title">👋 手部动作识别演示</view>
            <view class="info-desc">识别手部手势和动作</view>
            <view class="info-tips">
                <text class="tip-icon">💡</text>
                <text class="tip-text">将手放在扫描框内进行手势识别</text>
            </view>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            isDetecting: false,
            detectionTimer: null,
            fingers: [
                { name: '拇指', x: 25, y: 35 },
                { name: '食指', x: 40, y: 20 },
                { name: '中指', x: 50, y: 18 },
                { name: '无名指', x: 60, y: 22 },
                { name: '小指', x: 70, y: 30 }
            ],
            gestures: [
                { icon: '🤚', name: '手掌张开', description: '五指完全张开' },
                { icon: '✊', name: '拳头', description: '五指紧握' },
                { icon: '👍', name: '点赞', description: '拇指竖起' },
                { icon: '👎', name: '差评', description: '拇指向下' },
                { icon: '✌️', name: '剪刀手', description: '食指和中指伸出' },
                { icon: '🤟', name: '爱你', description: '拇指和小指伸出' },
                { icon: '👌', name: 'OK', description: '拇指和食指成圈' },
                { icon: '🤘', name: '摇滚', description: '食指和小指伸出' }
            ],
            currentGesture: { icon: '🖐️', name: '等待识别', description: '请将手放入扫描框' },
            gestureHistory: []
        };
    },
    onUnload() {
        if (this.detectionTimer) {
            clearInterval(this.detectionTimer);
        }
    },
    methods: {
        handleBack() {
            uni.navigateBack({
                delta: 1
            });
        },
        toggleDetection() {
            this.isDetecting = !this.isDetecting;
            
            if (this.isDetecting) {
                this.startDetection();
            } else {
                this.stopDetection();
            }
        },
        getDetectionBtnClass() {
            return this.isDetecting ? 'control-btn active' : 'control-btn';
        },
        startDetection() {
            this.currentGesture = { icon: '🔍', name: '识别中...', description: '正在分析手部特征' };
            
            this.detectionTimer = setInterval(() => {
                const randomGesture = this.gestures[Math.floor(Math.random() * this.gestures.length)];
                this.currentGesture = randomGesture;
                
                this.gestureHistory.unshift({
                    ...randomGesture,
                    time: new Date().toLocaleTimeString('zh-CN', { hour: '2-digit', minute: '2-digit', second: '2-digit' })
                });
                
                if (this.gestureHistory.length > 5) {
                    this.gestureHistory.pop();
                }
                
                this.updateFingerPositions();
            }, 2000);
        },
        stopDetection() {
            if (this.detectionTimer) {
                clearInterval(this.detectionTimer);
                this.detectionTimer = null;
            }
            this.currentGesture = { icon: '🖐️', name: '识别已停止', description: '点击开始按钮重新识别' };
        },
        clearDetection() {
            this.gestureHistory = [];
            this.currentGesture = { icon: '🖐️', name: '等待识别', description: '请将手放入扫描框' };
            uni.showToast({
                title: '已清除历史记录',
                icon: 'success',
                duration: 1500
            });
        },
        updateFingerPositions() {
            this.fingers = this.fingers.map(finger => ({
                ...finger,
                x: finger.x + (Math.random() - 0.5) * 4,
                y: finger.y + (Math.random() - 0.5) * 4
            }));
        }
    }
};
</script>

<style scoped>
.hand-tracking-container {
    position: relative;
    width: 100%;
    height: 100vh;
    background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
    overflow: hidden;
}

.hand-header {
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

.hand-viewer {
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding-top: 140rpx;
    padding-bottom: 380rpx;
}

.camera-preview {
    width: 90%;
    height: 50%;
    background: linear-gradient(180deg, #2d3436 0%, #000 100%);
    border-radius: 30rpx;
    overflow: hidden;
    position: relative;
}

.scan-area {
    width: 100%;
    height: 100%;
    position: relative;
}

.hand-detection {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    display: flex;
    align-items: center;
    justify-content: center;
}

.finger-point {
    position: absolute;
    display: flex;
    flex-direction: column;
    align-items: center;
    transform: translate(-50%, -50%);
    transition: all 0.5s ease;
}

.finger-dot {
    width: 24rpx;
    height: 24rpx;
    background: #00ffff;
    border-radius: 50%;
    box-shadow: 0 0 15rpx #00ffff;
    animation: pulse 1.5s infinite;
}

@keyframes pulse {
    0%, 100% { transform: scale(1); opacity: 1; }
    50% { transform: scale(1.5); opacity: 0.6; }
}

.finger-label {
    font-size: 20rpx;
    color: #00ffff;
    margin-top: 8rpx;
    text-shadow: 0 0 10rpx #00ffff;
}

.hand-center {
    position: absolute;
    display: flex;
    flex-direction: column;
    align-items: center;
    transition: all 0.5s ease;
}

.palm-icon {
    font-size: 80rpx;
    animation: float 2s ease-in-out infinite;
}

@keyframes float {
    0%, 100% { transform: translateY(0); }
    50% { transform: translateY(-10rpx); }
}

.palm-label {
    font-size: 24rpx;
    color: #00ffff;
    text-shadow: 0 0 10rpx #00ffff;
}

.scan-frame {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 70%;
    height: 70%;
    border: 2rpx solid rgba(221, 160, 221, 0.5);
    border-radius: 20rpx;
}

.corner {
    position: absolute;
    width: 50rpx;
    height: 50rpx;
    border: 6rpx solid #dda0dd;
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

.scan-lines {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    height: 4rpx;
    background: linear-gradient(90deg, transparent, #dda0dd, transparent);
    animation: scanMove 2s linear infinite;
}

@keyframes scanMove {
    0% { top: 10%; }
    100% { top: 90%; }
}

.gesture-display {
    text-align: center;
    margin-top: 40rpx;
}

.gesture-icon {
    font-size: 80rpx;
    margin-bottom: 15rpx;
}

.gesture-name {
    font-size: 36rpx;
    font-weight: bold;
    color: white;
    margin-bottom: 10rpx;
}

.gesture-desc {
    font-size: 26rpx;
    color: rgba(255, 255, 255, 0.7);
}

.hand-controls {
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
    background: rgba(221, 160, 221, 0.2);
    border-color: #dda0dd;
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

.gesture-list {
    position: absolute;
    bottom: 180rpx;
    left: 30rpx;
    right: 30rpx;
    background: rgba(255, 255, 255, 0.1);
    backdrop-filter: blur(10px);
    border-radius: 20rpx;
    padding: 25rpx;
}

.list-title {
    font-size: 26rpx;
    color: rgba(255, 255, 255, 0.9);
    margin-bottom: 15rpx;
    font-weight: bold;
}

.gesture-items {
    display: flex;
    flex-direction: column;
    gap: 12rpx;
}

.gesture-item {
    display: flex;
    align-items: center;
    padding: 15rpx 20rpx;
    background: rgba(255, 255, 255, 0.05);
    border-radius: 12rpx;
}

.gesture-emoji {
    font-size: 32rpx;
    margin-right: 15rpx;
}

.gesture-text {
    flex: 1;
    font-size: 24rpx;
    color: white;
}

.gesture-time {
    font-size: 20rpx;
    color: rgba(255, 255, 255, 0.5);
}

.hand-info-panel {
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