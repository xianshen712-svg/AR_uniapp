<template>
    <view class="xr-scene-container">
        <view class="xr-header">
            <view class="back-btn" @tap="handleBack">
                <text class="back-icon">←</text>
            </view>
            <view class="header-title">基础3D渲染</view>
        </view>

        <view class="xr-canvas-wrapper">
            <canvas 
                canvas-id="xrCanvas" 
                id="xrCanvas"
                class="xr-canvas"
                :style="{ width: canvasWidth + 'px', height: canvasHeight + 'px' }"
                @touchstart="handleTouchStart"
                @touchmove="handleTouchMove"
                @touchend="handleTouchEnd"
            ></canvas>
        </view>

        <view class="xr-info-panel">
            <view class="info-title">🎲 基础3D渲染演示</view>
            <view class="info-desc">展示基础3D几何体和材质效果</view>
            <view class="info-tips">
                <text class="tip-icon">💡</text>
                <text class="tip-text">拖动屏幕旋转视角</text>
            </view>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            canvasWidth: 300,
            canvasHeight: 300,
            ctx: null,
            rotation: 0,
            touchStartX: 0,
            shapes: [
                { type: 'cube', x: -60, y: 0, size: 40, color: '#FF6B6B', rotation: 0 },
                { type: 'sphere', x: 0, y: 0, size: 35, color: '#4ECDC4', rotation: 0 },
                { type: 'cylinder', x: 60, y: 0, size: 30, color: '#45B7D1', rotation: 0 },
                { type: 'cone', x: -30, y: -50, size: 25, color: '#FFB347', rotation: 0 },
                { type: 'cube', x: 30, y: -50, size: 25, color: '#96CEB4', rotation: 0 }
            ]
        };
    },
    onLoad() {
        const info = uni.getSystemInfoSync();
        this.canvasWidth = info.windowWidth;
        this.canvasHeight = info.windowHeight;
        
        setTimeout(() => {
            this.initCanvas();
            this.startAnimation();
        }, 100);
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
        initCanvas() {
            this.ctx = uni.createCanvasContext('xrCanvas');
            this.ctx.setFillStyle('#1a1a2e');
            this.ctx.fillRect(0, 0, this.canvasWidth, this.canvasHeight);
            this.ctx.draw();
        },
        startAnimation() {
            const animate = () => {
                this.rotation += 0.02;
                this.shapes.forEach((shape, index) => {
                    shape.rotation += 0.01 + index * 0.005;
                });
                this.drawScene();
                this.animationId = requestAnimationFrame(animate);
            };
            animate();
        },
        drawScene() {
            const ctx = this.ctx;
            const centerX = this.canvasWidth / 2;
            const centerY = this.canvasHeight / 2;

            ctx.setFillStyle('#1a1a2e');
            ctx.fillRect(0, 0, this.canvasWidth, this.canvasHeight);

            this.drawGrid(ctx, centerX, centerY);

            this.shapes.forEach(shape => {
                ctx.save();
                ctx.translate(centerX + shape.x, centerY + shape.y);
                ctx.rotate(this.rotation + shape.rotation);
                
                const scale = 1 + Math.sin(Date.now() / 500 + shape.x) * 0.1;
                ctx.scale(scale, scale);

                this.drawShape(ctx, shape);
                ctx.restore();
            });

            ctx.draw();
        },
        drawGrid(ctx, cx, cy) {
            ctx.setStrokeStyle('rgba(255, 255, 255, 0.1)');
            ctx.setLineWidth(1);

            const gridSize = 40;
            const gridCount = 10;

            for (let i = -gridCount; i <= gridCount; i++) {
                ctx.beginPath();
                ctx.moveTo(cx + i * gridSize, cy - gridCount * gridSize);
                ctx.lineTo(cx + i * gridSize, cy + gridCount * gridSize);
                ctx.stroke();

                ctx.beginPath();
                ctx.moveTo(cx - gridCount * gridSize, cy + i * gridSize);
                ctx.lineTo(cx + gridCount * gridSize, cy + i * gridSize);
                ctx.stroke();
            }
        },
        drawShape(ctx, shape) {
            const { type, size, color } = shape;
            
            const gradient = ctx.createLinearGradient(-size, -size, size, size);
            gradient.addColorStop(0, color);
            gradient.addColorStop(1, this.darkenColor(color, 30));

            ctx.setFillStyle(gradient);
            ctx.setStrokeStyle('rgba(255, 255, 255, 0.3)');
            ctx.setLineWidth(2);

            switch (type) {
                case 'cube':
                    this.drawCube(ctx, size);
                    break;
                case 'sphere':
                    this.drawSphere(ctx, size);
                    break;
                case 'cylinder':
                    this.drawCylinder(ctx, size);
                    break;
                case 'cone':
                    this.drawCone(ctx, size);
                    break;
            }
        },
        drawCube(ctx, size) {
            ctx.beginPath();
            ctx.moveTo(-size/2, -size/2);
            ctx.lineTo(size/2, -size/2);
            ctx.lineTo(size/2, size/2);
            ctx.lineTo(-size/2, size/2);
            ctx.closePath();
            ctx.fill();
            ctx.stroke();
        },
        drawSphere(ctx, size) {
            const segments = 36;
            const radius = size;
            
            for (let i = 0; i < segments; i++) {
                const lat1 = Math.PI * (-0.5 + (i - 1) / segments);
                const lat2 = Math.PI * (-0.5 + i / segments);
                
                ctx.beginPath();
                for (let j = 0; j <= segments; j++) {
                    const lng = 2 * Math.PI * (j - 1) / segments;
                    const x = radius * Math.cos(lat2) * Math.cos(lng);
                    const y = radius * Math.sin(lat2);
                    if (j === 0) {
                        ctx.moveTo(x, y);
                    } else {
                        ctx.lineTo(x, y);
                    }
                }
                ctx.fill();
            }
        },
        drawCylinder(ctx, size) {
            ctx.beginPath();
            ctx.arc(0, 0, size, 0, 2 * Math.PI);
            ctx.fill();
            ctx.stroke();

            ctx.beginPath();
            ctx.moveTo(-size, -size);
            ctx.lineTo(-size, size);
            ctx.lineTo(size, size);
            ctx.lineTo(size, -size);
            ctx.closePath();
            ctx.fill();
            ctx.stroke();
        },
        drawCone(ctx, size) {
            ctx.beginPath();
            ctx.moveTo(0, -size);
            ctx.lineTo(size, size);
            ctx.lineTo(-size, size);
            ctx.closePath();
            ctx.fill();
            ctx.stroke();

            ctx.beginPath();
            ctx.arc(0, size, size * 0.3, 0, 2 * Math.PI);
            ctx.fill();
        },
        darkenColor(color, amount) {
            const num = parseInt(color.replace('#', ''), 16);
            const amt = Math.round(2.55 * amount);
            const R = Math.max((num >> 16) - amt, 0);
            const G = Math.max((num >> 8 & 0x00FF) - amt, 0);
            const B = Math.max((num & 0x0000FF) - amt, 0);
            return '#' + (0x1000000 + R * 0x10000 + G * 0x100 + B).toString(16).slice(1);
        },
        handleTouchStart(e) {
            this.touchStartX = e.touches[0].clientX;
        },
        handleTouchMove(e) {
            const deltaX = e.touches[0].clientX - this.touchStartX;
            this.rotation += deltaX * 0.01;
            this.touchStartX = e.touches[0].clientX;
        },
        handleTouchEnd() {
            // 触摸结束处理
        }
    }
};
</script>

<style scoped>
.xr-scene-container {
    position: relative;
    width: 100%;
    height: 100vh;
    background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
    overflow: hidden;
}

.xr-header {
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

.xr-canvas-wrapper {
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
}

.xr-canvas {
    display: block;
}

.xr-info-panel {
    position: absolute;
    bottom: 60rpx;
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