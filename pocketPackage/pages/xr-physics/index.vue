<template>
    <view class="physics-container">
        <view class="physics-header">
            <view class="back-btn" @tap="handleBack">
                <text class="back-icon">←</text>
            </view>
            <view class="header-title">物理引擎</view>
        </view>

        <view class="physics-viewer">
            <view class="physics-scene" @tap="dropObject">
                <view 
                    v-for="(obj, index) in physicsObjects" 
                    :key="index"
                    class="physics-object"
                    :class="obj.type"
                    :style="{ 
                        left: obj.x + '%', 
                        top: obj.y + '%',
                        transform: `rotate(${obj.rotation}deg)`,
                        opacity: obj.opacity
                    }"
                >
                    <text class="object-icon">{{ obj.icon }}</text>
                    <view class="object-shadow" :style="{ opacity: obj.shadowOpacity }"></view>
                </view>
                <view class="ground"></view>
                <view class="drop-hint">点击屏幕投放物体</view>
            </view>
        </view>

        <view class="physics-controls">
            <view class="control-section">
                <view class="section-title">物理参数</view>
                <view class="parameter-row">
                    <view class="parameter-item">
                        <text class="param-label">重力</text>
                        <slider 
                            :value="gravity" 
                            @change="onGravityChange" 
                            :min="0" 
                            :max="10" 
                            :step="1"
                            activeColor="#F7DC6F"
                            backgroundColor="#ddd"
                            block-size="20"
                        />
                        <text class="param-value">{{ gravity }}</text>
                    </view>
                </view>
            </view>

            <view class="control-section">
                <view class="section-title">物体选择</view>
                <view class="object-selector">
                    <view 
                        v-for="(type, index) in objectTypes" 
                        :key="index"
                        class="object-option"
                        :class="{ active: selectedObjectType === type.id }"
                        @tap="selectObjectType(type)"
                    >
                        <text class="object-icon-small">{{ type.icon }}</text>
                        <text class="object-name">{{ type.name }}</text>
                    </view>
                </view>
            </view>

            <view class="control-section">
                <view class="control-buttons">
                    <view class="control-btn" @tap="clearObjects">
                        <text class="control-icon">🗑️</text>
                        <text class="control-text">清除</text>
                    </view>
                    <view class="control-btn" @tap="toggleGravity">
                        <text class="control-icon">{{ isGravityEnabled ? '🌍' : '🚀' }}</text>
                        <text class="control-text">{{ isGravityEnabled ? '重力开启' : '失重模式' }}</text>
                    </view>
                    <view class="control-btn" @tap="addRandomObjects">
                        <text class="control-icon">🎲</text>
                        <text class="control-text">随机投放</text>
                    </view>
                </view>
            </view>
        </view>

        <view class="physics-info-panel">
            <view class="info-title">⚡ 物理引擎演示</view>
            <view class="info-desc">真实的物理碰撞和重力效果</view>
            <view class="info-tips">
                <text class="tip-icon">💡</text>
                <text class="tip-text">点击屏幕投放物体，体验物理效果</text>
            </view>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            physicsObjects: [],
            gravity: 5,
            isGravityEnabled: true,
            selectedObjectType: 'ball',
            objectTypes: [
                { id: 'ball', name: '球', icon: '🔵' },
                { id: 'box', name: '方块', icon: '📦' },
                { id: 'pyramid', name: '锥体', icon: '🔺' },
                { id: 'cylinder', name: '圆柱', icon: '🔷' },
                { id: 'star', name: '星星', icon: '⭐' }
            ],
            animationTimer: null
        };
    },
    onLoad() {
        this.startPhysicsSimulation();
    },
    onUnload() {
        if (this.animationTimer) {
            clearInterval(this.animationTimer);
        }
    },
    methods: {
        handleBack() {
            uni.navigateBack({
                delta: 1
            });
        },
        startPhysicsSimulation() {
            this.animationTimer = setInterval(() => {
                if (this.isGravityEnabled) {
                    this.physicsObjects = this.physicsObjects.map(obj => {
                        if (obj.y < 85) {
                            obj.velocityY += this.gravity * 0.1;
                            obj.y += obj.velocityY;
                            obj.rotation += obj.rotationSpeed;
                            
                            if (obj.y >= 85) {
                                obj.y = 85;
                                obj.velocityY *= -0.7;
                                obj.rotationSpeed *= 0.9;
                                
                                if (Math.abs(obj.velocityY) < 0.3) {
                                    obj.velocityY = 0;
                                    obj.rotationSpeed = 0;
                                }
                            }
                            
                            obj.shadowOpacity = Math.max(0.1, 1 - (obj.y / 100));
                        }
                        
                        return obj;
                    });
                }
            }, 50);
        },
        dropObject(e) {
            const touchX = e.touches ? e.touches[0].clientX : 50;
            const screenWidth = uni.getSystemInfoSync().windowWidth;
            const x = (touchX / screenWidth) * 100;
            
            const type = this.objectTypes.find(t => t.id === this.selectedObjectType);
            
            const newObject = {
                x: Math.min(Math.max(x, 10), 90),
                y: 5,
                velocityY: 0,
                rotation: 0,
                rotationSpeed: (Math.random() - 0.5) * 10,
                type: this.selectedObjectType,
                icon: type.icon,
                opacity: 1,
                shadowOpacity: 0.1
            };
            
            this.physicsObjects.push(newObject);
            
            if (this.physicsObjects.length > 10) {
                this.physicsObjects.shift();
            }
        },
        selectObjectType(type) {
            this.selectedObjectType = type.id;
            uni.showToast({
                title: `已选择: ${type.name}`,
                icon: 'none',
                duration: 1000
            });
        },
        onGravityChange(e) {
            this.gravity = e.detail.value;
        },
        toggleGravity() {
            this.isGravityEnabled = !this.isGravityEnabled;
            
            if (!this.isGravityEnabled) {
                this.physicsObjects.forEach(obj => {
                    obj.velocityY = 0;
                    obj.rotationSpeed = 0;
                });
            }
            
            uni.showToast({
                title: this.isGravityEnabled ? '重力已开启' : '失重模式',
                icon: 'none',
                duration: 1500
            });
        },
        clearObjects() {
            this.physicsObjects = [];
            uni.showToast({
                title: '已清除所有物体',
                icon: 'success',
                duration: 1500
            });
        },
        addRandomObjects() {
            for (let i = 0; i < 3; i++) {
                setTimeout(() => {
                    const randomType = this.objectTypes[Math.floor(Math.random() * this.objectTypes.length)];
                    this.selectedObjectType = randomType.id;
                    
                    const newObject = {
                        x: 20 + Math.random() * 60,
                        y: 5,
                        velocityY: 0,
                        rotation: 0,
                        rotationSpeed: (Math.random() - 0.5) * 10,
                        type: randomType.id,
                        icon: randomType.icon,
                        opacity: 1,
                        shadowOpacity: 0.1
                    };
                    
                    this.physicsObjects.push(newObject);
                    
                    if (this.physicsObjects.length > 10) {
                        this.physicsObjects.shift();
                    }
                }, i * 200);
            }
            
            uni.showToast({
                title: '投放3个随机物体',
                icon: 'none',
                duration: 1500
            });
        }
    }
};
</script>

<style scoped>
.physics-container {
    position: relative;
    width: 100%;
    height: 100vh;
    background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
    overflow: hidden;
}

.physics-header {
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

.physics-viewer {
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding-top: 140rpx;
    padding-bottom: 380rpx;
}

.physics-scene {
    width: 90%;
    height: 100%;
    background: linear-gradient(180deg, rgba(26, 26, 46, 0.5) 0%, rgba(22, 33, 62, 0.8) 100%);
    border-radius: 30rpx;
    position: relative;
    overflow: hidden;
}

.physics-object {
    position: absolute;
    width: 80rpx;
    height: 80rpx;
    display: flex;
    align-items: center;
    justify-content: center;
    transform: translate(-50%, -50%);
    transition: top 0.05s linear, transform 0.05s linear;
}

.object-icon {
    font-size: 50rpx;
    filter: drop-shadow(0 8rpx 16rpx rgba(0, 0, 0, 0.3));
}

.object-shadow {
    position: absolute;
    bottom: -15rpx;
    width: 60rpx;
    height: 10rpx;
    background: rgba(0, 0, 0, 0.4);
    border-radius: 50%;
    filter: blur(8rpx);
    transition: opacity 0.1s ease;
}

.ground {
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    height: 60rpx;
    background: linear-gradient(180deg, rgba(102, 126, 234, 0.3) 0%, rgba(102, 126, 234, 0.6) 100%);
    border-radius: 0 0 30rpx 30rpx;
}

.drop-hint {
    position: absolute;
    bottom: 80rpx;
    left: 50%;
    transform: translateX(-50%);
    font-size: 24rpx;
    color: rgba(255, 255, 255, 0.5);
}

.physics-controls {
    position: absolute;
    bottom: 180rpx;
    left: 30rpx;
    right: 30rpx;
    background: rgba(255, 255, 255, 0.1);
    backdrop-filter: blur(10px);
    border-radius: 20rpx;
    padding: 25rpx;
}

.control-section {
    margin-bottom: 20rpx;
}

.control-section:last-child {
    margin-bottom: 0;
}

.section-title {
    font-size: 24rpx;
    color: rgba(255, 255, 255, 0.8);
    margin-bottom: 15rpx;
    font-weight: bold;
}

.parameter-row {
    display: flex;
    gap: 15rpx;
}

.parameter-item {
    flex: 1;
}

.param-label {
    font-size: 22rpx;
    color: rgba(255, 255, 255, 0.6);
    margin-bottom: 10rpx;
    display: block;
}

.param-value {
    font-size: 24rpx;
    color: #F7DC6F;
    text-align: right;
    margin-top: 10rpx;
    display: block;
}

.object-selector {
    display: flex;
    gap: 15rpx;
}

.object-option {
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

.object-option.active {
    background: rgba(247, 220, 111, 0.2);
    border-color: #F7DC6F;
}

.object-icon-small {
    font-size: 36rpx;
    margin-bottom: 8rpx;
}

.object-name {
    font-size: 20rpx;
    color: rgba(255, 255, 255, 0.8);
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

.physics-info-panel {
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