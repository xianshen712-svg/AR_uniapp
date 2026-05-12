<template>
    <view class="video-texture-container">
        <view class="video-header">
            <view class="back-btn" @tap="handleBack">
                <text class="back-icon">←</text>
            </view>
            <view class="header-title">视频纹理</view>
        </view>

        <view class="video-viewer">
            <view class="video-3d-scene">
                <view class="video-cube" :style="{ transform: `rotateY(${rotationY}deg) rotateX(${rotationX}deg)` }">
                    <view class="cube-face front">
                        <video 
                            v-if="currentVideo === 'front'"
                            :src="videoUrl" 
                            class="face-video"
                            autoplay
                            loop
                            muted
                            playsinline
                        />
                        <view v-else class="face-placeholder">🎬</view>
                    </view>
                    <view class="cube-face back">
                        <video 
                            v-if="currentVideo === 'back'"
                            :src="videoUrl" 
                            class="face-video"
                            autoplay
                            loop
                            muted
                            playsinline
                        />
                        <view v-else class="face-placeholder">🎥</view>
                    </view>
                    <view class="cube-face left">
                        <video 
                            v-if="currentVideo === 'left'"
                            :src="videoUrl" 
                            class="face-video"
                            autoplay
                            loop
                            muted
                            playsinline
                        />
                        <view v-else class="face-placeholder">📹</view>
                    </view>
                    <view class="cube-face right">
                        <video 
                            v-if="currentVideo === 'right'"
                            :src="videoUrl" 
                            class="face-video"
                            autoplay
                            loop
                            muted
                            playsinline
                        />
                        <view v-else class="face-placeholder">🎞️</view>
                    </view>
                    <view class="cube-face top">
                        <view class="face-placeholder">🔝</view>
                    </view>
                    <view class="cube-face bottom">
                        <view class="face-placeholder">🔻</view>
                    </view>
                </view>
                <view class="cube-shadow"></view>
            </view>

            <view class="video-info">
                <view class="video-name">视频纹理立方体</view>
                <view class="video-status">
                    <text :class="['status-dot', isPlaying ? 'playing' : 'paused']"></text>
                    <text class="status-text">{{ isPlaying ? '播放中' : '已暂停' }}</text>
                </view>
            </view>
        </view>

        <view class="video-controls">
            <view class="control-section">
                <view class="section-title">视频选择</view>
                <view class="video-selector">
                    <view 
                        v-for="(video, index) in videoList" 
                        :key="index"
                        class="video-option"
                        :class="{ active: selectedVideoIndex === index }"
                        @tap="selectVideo(index)"
                    >
                        <text class="video-icon">{{ video.icon }}</text>
                        <text class="video-label">{{ video.name }}</text>
                    </view>
                </view>
            </view>

            <view class="control-section">
                <view class="section-title">播放控制</view>
                <view class="control-buttons">
                    <view class="control-btn" @tap="togglePlay">
                        <text class="control-icon">{{ isPlaying ? '⏸️' : '▶️' }}</text>
                        <text class="control-text">{{ isPlaying ? '暂停' : '播放' }}</text>
                    </view>
                    <view class="control-btn" @tap="toggleAutoRotate">
                        <text class="control-icon">{{ isAutoRotate ? '⏹️' : '🔄' }}</text>
                        <text class="control-text">{{ isAutoRotate ? '停止旋转' : '自动旋转' }}</text>
                    </view>
                    <view class="control-btn" @tap="switchFace">
                        <text class="control-icon">🔀</text>
                        <text class="control-text">切换面</text>
                    </view>
                </view>
            </view>

            <view class="control-section">
                <view class="section-title">旋转速度</view>
                <slider 
                    :value="rotateSpeed" 
                    @change="onRotateSpeedChange" 
                    :min="1" 
                    :max="10" 
                    :step="1"
                    activeColor="#96CEB4"
                    backgroundColor="#ddd"
                    block-size="20"
                />
                <view class="speed-value">速度: {{ rotateSpeed }}</view>
            </view>
        </view>

        <view class="video-info-panel">
            <view class="info-title">🎬 视频纹理演示</view>
            <view class="info-desc">在3D物体表面播放视频内容</view>
            <view class="info-tips">
                <text class="tip-icon">💡</text>
                <text class="tip-text">拖动屏幕旋转立方体，观看不同面的视频纹理</text>
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
            isPlaying: true,
            animationId: null,
            touchStartX: 0,
            touchStartY: 0,
            selectedVideoIndex: 0,
            currentVideo: 'front',
            videoList: [
                { name: '自然风景', icon: '🌄', url: 'https://www.w3schools.com/html/mov_bbb.mp4' },
                { name: '城市夜景', icon: '🌃', url: 'https://www.w3schools.com/html/mov_bbb.mp4' },
                { name: '海洋生物', icon: '🐠', url: 'https://www.w3schools.com/html/mov_bbb.mp4' },
                { name: '太空探索', icon: '🚀', url: 'https://www.w3schools.com/html/mov_bbb.mp4' }
            ],
            faces: ['front', 'back', 'left', 'right']
        };
    },
    computed: {
        videoUrl() {
            return this.videoList[this.selectedVideoIndex].url;
        }
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
        togglePlay() {
            this.isPlaying = !this.isPlaying;
        },
        onRotateSpeedChange(e) {
            this.rotateSpeed = e.detail.value;
        },
        selectVideo(index) {
            this.selectedVideoIndex = index;
            uni.showToast({
                title: `已切换到 ${this.videoList[index].name}`,
                icon: 'none',
                duration: 1500
            });
        },
        switchFace() {
            const currentIndex = this.faces.indexOf(this.currentVideo);
            const nextIndex = (currentIndex + 1) % this.faces.length;
            this.currentVideo = this.faces[nextIndex];
            uni.showToast({
                title: `视频显示在${this.getFaceName(this.currentVideo)}`,
                icon: 'none',
                duration: 1500
            });
        },
        getFaceName(face) {
            const names = {
                front: '前面',
                back: '后面',
                left: '左面',
                right: '右面'
            };
            return names[face] || face;
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
.video-texture-container {
    position: relative;
    width: 100%;
    height: 100vh;
    background: linear-gradient(135deg, #0f3460 0%, #16213e 100%);
    overflow: hidden;
}

.video-header {
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

.video-viewer {
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding-top: 140rpx;
    padding-bottom: 420rpx;
}

.video-3d-scene {
    width: 70%;
    height: 70%;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    perspective: 1200px;
}

.video-cube {
    width: 240rpx;
    height: 240rpx;
    transform-style: preserve-3d;
    transition: transform 0.1s ease-out;
}

.cube-face {
    position: absolute;
    width: 240rpx;
    height: 240rpx;
    display: flex;
    align-items: center;
    justify-content: center;
    border: 3rpx solid rgba(255, 255, 255, 0.2);
    border-radius: 15rpx;
    backface-visibility: visible;
    overflow: hidden;
}

.face-video {
    width: 100%;
    height: 100%;
    object-fit: cover;
}

.face-placeholder {
    font-size: 60rpx;
    background: linear-gradient(135deg, rgba(150, 206, 180, 0.3), rgba(102, 126, 234, 0.3));
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
}

.front {
    background: linear-gradient(135deg, rgba(150, 206, 180, 0.5), rgba(102, 126, 234, 0.5));
    transform: translateZ(120rpx);
}

.back {
    background: linear-gradient(135deg, rgba(78, 205, 196, 0.5), rgba(69, 183, 209, 0.5));
    transform: rotateY(180deg) translateZ(120rpx);
}

.left {
    background: linear-gradient(135deg, rgba(255, 107, 107, 0.5), rgba(255, 179, 71, 0.5));
    transform: rotateY(-90deg) translateZ(120rpx);
}

.right {
    background: linear-gradient(135deg, rgba(221, 160, 221, 0.5), rgba(135, 206, 235, 0.5));
    transform: rotateY(90deg) translateZ(120rpx);
}

.top {
    background: linear-gradient(135deg, rgba(247, 220, 111, 0.5), rgba(150, 206, 180, 0.5));
    transform: rotateX(90deg) translateZ(120rpx);
}

.bottom {
    background: linear-gradient(135deg, rgba(102, 126, 234, 0.5), rgba(118, 75, 162, 0.5));
    transform: rotateX(-90deg) translateZ(120rpx);
}

.cube-shadow {
    width: 200rpx;
    height: 30rpx;
    background: rgba(0, 0, 0, 0.4);
    border-radius: 50%;
    margin-top: 60rpx;
    filter: blur(15px);
}

.video-info {
    text-align: center;
    margin-top: 40rpx;
}

.video-name {
    font-size: 32rpx;
    font-weight: bold;
    color: white;
}

.video-status {
    display: flex;
    align-items: center;
    justify-content: center;
    margin-top: 10rpx;
}

.status-dot {
    width: 16rpx;
    height: 16rpx;
    border-radius: 50%;
    margin-right: 10rpx;
    transition: all 0.3s;
}

.status-dot.playing {
    background: #00ff00;
    box-shadow: 0 0 10rpx #00ff00;
}

.status-dot.paused {
    background: #ff6b6b;
    box-shadow: 0 0 10rpx #ff6b6b;
}

.status-text {
    font-size: 24rpx;
    color: rgba(255, 255, 255, 0.8);
}

.video-controls {
    position: absolute;
    bottom: 180rpx;
    left: 30rpx;
    right: 30rpx;
    background: rgba(255, 255, 255, 0.1);
    backdrop-filter: blur(10px);
    border-radius: 20rpx;
    padding: 30rpx;
}

.control-section {
    margin-bottom: 25rpx;
}

.control-section:last-child {
    margin-bottom: 0;
}

.section-title {
    font-size: 26rpx;
    color: rgba(255, 255, 255, 0.9);
    margin-bottom: 15rpx;
    font-weight: bold;
}

.video-selector {
    display: flex;
    gap: 15rpx;
}

.video-option {
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 20rpx 15rpx;
    background: rgba(255, 255, 255, 0.05);
    border-radius: 15rpx;
    border: 2rpx solid transparent;
    transition: all 0.3s;
}

.video-option.active {
    background: rgba(150, 206, 180, 0.2);
    border-color: #96CEB4;
}

.video-icon {
    font-size: 36rpx;
    margin-bottom: 8rpx;
}

.video-label {
    font-size: 20rpx;
    color: rgba(255, 255, 255, 0.8);
    text-align: center;
}

.control-buttons {
    display: flex;
    gap: 15rpx;
}

.control-btn {
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 25rpx 15rpx;
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
    font-size: 36rpx;
    margin-bottom: 8rpx;
}

.control-text {
    font-size: 22rpx;
    color: white;
    text-align: center;
}

.speed-value {
    font-size: 22rpx;
    color: rgba(255, 255, 255, 0.6);
    text-align: center;
    margin-top: 10rpx;
}

.video-info-panel {
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