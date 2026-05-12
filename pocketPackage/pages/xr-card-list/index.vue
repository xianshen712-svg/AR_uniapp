<template>
    <view class="card-list-container">
        <view class="header">
            <view class="header-title">XR 能力演示</view>
            <view class="header-subtitle">选择一个能力开始体验</view>
        </view>

        <view class="card-grid">
            <view 
                v-for="(card, index) in cardList" 
                :key="index" 
                class="card-item"
                @tap="handleCardClick(card)"
            >
                <view class="card-icon" :style="{ backgroundColor: card.color }">
                    <text class="card-icon-text">{{ card.icon }}</text>
                </view>
                <view class="card-content">
                    <view class="card-title">{{ card.title }}</view>
                    <view class="card-desc">{{ card.description }}</view>
                    <view class="card-tag" :style="{ backgroundColor: card.tagColor }">
                        {{ card.tag }}
                    </view>
                </view>
                <view class="card-arrow">→</view>
            </view>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            cardList: [
                {
                    id: 'basic-3d',
                    title: '基础3D渲染',
                    description: '展示基础3D几何体和材质效果',
                    icon: '🎲',
                    color: '#FF6B6B',
                    tag: '基础',
                    tagColor: '#FFE5E5',
                    url: '/pocketPackage/pages/xr-basic-3d/index'
                },
                {
                    id: 'ar-plane',
                    title: 'AR平面检测',
                    description: '识别水平面并放置3D物体',
                    icon: '📱',
                    color: '#4ECDC4',
                    tag: 'AR',
                    tagColor: '#E5F9F8',
                    url: '/pocketPackage/pages/xr-ar-plane/index'
                },
                {
                    id: 'gltf-model',
                    title: 'glTF模型展示',
                    description: '加载和展示glTF 3D模型',
                    icon: '🤖',
                    color: '#45B7D1',
                    tag: '模型',
                    tagColor: '#E5F3F8',
                    url: '/pocketPackage/pages/xr-gltf-model/index'
                },
                {
                    id: 'video-texture',
                    title: '视频纹理',
                    description: '在3D物体上播放视频',
                    icon: '🎬',
                    color: '#96CEB4',
                    tag: '视频',
                    tagColor: '#E8F5E9',
                    url: '/pocketPackage/pages/xr-video-texture/index'
                },
                {
                    id: 'hand-tracking',
                    title: '手部动作识别',
                    description: '识别手部手势和动作',
                    icon: '👋',
                    color: '#DDA0DD',
                    tag: 'AI',
                    tagColor: '#F3E5F5',
                    url: '/pocketPackage/pages/xr-hand-tracking/index'
                },
                {
                    id: 'image-marker',
                    title: 'AR图片Marker',
                    description: '识别特定图片并触发AR',
                    icon: '🖼️',
                    color: '#FFB347',
                    tag: '识别',
                    tagColor: '#FFF3E0',
                    url: '/pocketPackage/pages/xr-image-marker/index'
                },
                {
                    id: '3dof-tracking',
                    title: '3DOF追踪',
                    description: '3自由度头部追踪效果',
                    icon: '🎯',
                    color: '#87CEEB',
                    tag: '追踪',
                    tagColor: '#E1F5FE',
                    url: '/pocketPackage/pages/xr-3dof-tracking/index'
                },
                {
                    id: 'physics-engine',
                    title: '物理引擎',
                    description: '真实的物理碰撞和重力效果',
                    icon: '⚡',
                    color: '#F7DC6F',
                    tag: '物理',
                    tagColor: '#FEF9E7',
                    url: '/pocketPackage/pages/xr-physics/index'
                }
            ]
        };
    },
    methods: {
        handleCardClick(card) {
            uni.navigateTo({
                url: card.url,
                success: () => {
                    console.log('进入XR场景:', card.title);
                },
                fail: (err) => {
                    console.error('导航失败:', err);
                    uni.showToast({
                        title: '该功能开发中',
                        icon: 'none'
                    });
                }
            });
        }
    }
};
</script>

<style scoped>
.card-list-container {
    min-height: 100vh;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    padding: 20rpx;
}

.header {
    text-align: center;
    padding: 40rpx 0;
    color: white;
}

.header-title {
    font-size: 48rpx;
    font-weight: bold;
    margin-bottom: 16rpx;
}

.header-subtitle {
    font-size: 28rpx;
    opacity: 0.9;
}

.card-grid {
    display: grid;
    grid-template-columns: 1fr;
    gap: 24rpx;
    padding-bottom: 40rpx;
}

.card-item {
    background: white;
    border-radius: 24rpx;
    padding: 32rpx;
    display: flex;
    align-items: center;
    box-shadow: 0 8rpx 24rpx rgba(0, 0, 0, 0.1);
    transition: transform 0.3s, box-shadow 0.3s;
    position: relative;
    overflow: hidden;
}

.card-item::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    height: 8rpx;
    background: linear-gradient(90deg, #667eea, #764ba2);
}

.card-item:active {
    transform: scale(0.98);
    box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.15);
}

.card-icon {
    width: 100rpx;
    height: 100rpx;
    border-radius: 20rpx;
    display: flex;
    align-items: center;
    justify-content: center;
    margin-right: 24rpx;
    flex-shrink: 0;
}

.card-icon-text {
    font-size: 48rpx;
}

.card-content {
    flex: 1;
    min-width: 0;
}

.card-title {
    font-size: 32rpx;
    font-weight: bold;
    color: #333;
    margin-bottom: 8rpx;
}

.card-desc {
    font-size: 24rpx;
    color: #666;
    margin-bottom: 12rpx;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
}

.card-tag {
    display: inline-block;
    padding: 4rpx 16rpx;
    border-radius: 20rpx;
    font-size: 20rpx;
    color: #666;
}

.card-arrow {
    font-size: 40rpx;
    color: #999;
    margin-left: 16rpx;
    flex-shrink: 0;
}
</style>