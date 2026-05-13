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
                // 基于 XR-Frame官方文档 实现
                {
                    id: 'xr-basic-scene',
                    title: '基础3D场景',
                    description: '立方体、灯光、纹理、环境天空盒',
                    icon: '🎲',
                    color: '#FF6B6B',
                    tag: '官方文档',
                    tagColor: '#FFE5E5',
                    url: '/pocketPackage/pages/xr-basic-scene/index'
                },
                // 基于 XR-Frame典型案例 实现
                {
                    id: 'xr-marker-video',
                    title: 'AR扫描播放视频',
                    description: '扫描Marker图片播放视频内容',
                    icon: '📱',
                    color: '#4ECDC4',
                    tag: '典型案例',
                    tagColor: '#E5F9F8',
                    url: '/pocketPackage/pages/xr-marker-video/index'
                },
                {
                    id: 'xr-osd-marker',
                    title: 'OSD物体识别',
                    description: '识别特定角度的物体并展示信息',
                    icon: '🎯',
                    color: '#FFB347',
                    tag: '典型案例',
                    tagColor: '#FFF3E0',
                    url: '/pocketPackage/pages/xr-osd-marker/index'
                },
                {
                    id: 'xr-hand-tracking',
                    title: '手部动作识别',
                    description: '识别手部手势和动作',
                    icon: '👋',
                    color: '#DDA0DD',
                    tag: '典型案例',
                    tagColor: '#F3E5F5',
                    url: '/pocketPackage/pages/xr-hand-tracking/index'
                },
                {
                    id: 'xr-model-animation',
                    title: '模型动画',
                    description: '播放模型动画，支持多种动画效果',
                    icon: '🎬',
                    color: '#96CEB4',
                    tag: '典型案例',
                    tagColor: '#E8F5E9',
                    url: '/pocketPackage/pages/xr-model-animation/index'
                },
                {
                    id: 'xr-hand-animation',
                    title: '手部动画交互',
                    description: '手势控制3D模型动画',
                    icon: '🎮',
                    color: '#667eea',
                    tag: '典型案例',
                    tagColor: '#E8E5FF',
                    url: '/pocketPackage/pages/xr-hand-animation/index'
                },
                {
                    id: 'xr-tracker-2d',
                    title: '综合图片视频识别',
                    description: '支持多目标图片识别和视频播放',
                    icon: '📸',
                    color: '#FF6B9D',
                    tag: '⭐⭐⭐',
                    tagColor: '#FFE5F0',
                    url: '/pocketPackage/pages/xr-tracker-2d/index'
                },
                {
                    id: 'xr-hand-product',
                    title: '手部产品销售',
                    description: 'AR虚拟试戴，手势交互购物',
                    icon: '💍',
                    color: '#FFD700',
                    tag: '⭐⭐⭐',
                    tagColor: '#FFF8DC',
                    url: '/pocketPackage/pages/xr-hand-product/index'
                },
                {
                    id: 'xr-gltf-dynamic',
                    title: '动态模型加载',
                    description: '运行时动态加载不同glTF模型',
                    icon: '🔄',
                    color: '#9C27B0',
                    tag: '⭐⭐⭐',
                    tagColor: '#F3E5F5',
                    url: '/pocketPackage/pages/xr-gltf-dynamic/index'
                },
                {
                    id: 'xr-lighting',
                    title: '光照效果',
                    description: '多种光源类型与参数调节',
                    icon: '💡',
                    color: '#FFD700',
                    tag: '⭐⭐⭐',
                    tagColor: '#FFF8DC',
                    url: '/pocketPackage/pages/xr-lighting/index'
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
