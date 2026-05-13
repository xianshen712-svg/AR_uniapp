<template>
  <view class="ar-page">
    <yn-viewer :showBackBtn="true" @back="handleBack">
      <yn-ar
        id="hand-product"
        :width="renderWidth"
        :height="renderHeight"
        :style="'width:' + width + 'px;height:' + height + 'px'"
        yn_ar_mode="Hand"
        yn_camera_clear_color="0.05 0.05 0.05 1"
        @arReady="handleARReady"
        @handDetected="handleHandDetected"
        @handLost="handleHandLost"
      />
    </yn-viewer>

    <!-- 手部识别引导 -->
    <view class="hand-guide" v-if="!isHandDetected && arReady">
      <view class="guide-icon">✋</view>
      <view class="guide-title">虚拟试戴体验</view>
      <view class="guide-text">请将手掌对准摄像头</view>
      <view class="guide-tips">👆 点击手势查看详情 | ✌️ 切换产品</view>
    </view>

    <!-- 3D产品展示 -->
    <view class="product-display" v-if="isHandDetected">
      <view class="product-model" :class="currentProduct.animationClass">
        <view class="model-glow"></view>
        <view class="product-icon">{{ currentProduct.icon }}</view>
        <view class="product-rings">
          <view class="ring ring-1"></view>
          <view class="ring ring-2"></view>
          <view class="ring ring-3"></view>
        </view>
      </view>
    </view>

    <!-- 产品信息卡片 -->
    <view class="product-card" v-if="isHandDetected">
      <view class="card-header">
        <view class="product-badge">{{ currentProduct.category }}</view>
        <view class="product-price">{{ currentProduct.price }}</view>
      </view>
      <view class="card-body">
        <view class="product-name">{{ currentProduct.name }}</view>
        <view class="product-desc">{{ currentProduct.description }}</view>
        <view class="product-features">
          <view class="feature-tag" v-for="(feature, index) in currentProduct.features" :key="index">
            {{ feature }}
          </view>
        </view>
      </view>
      <view class="card-actions">
        <view class="action-btn like" @tap="toggleLike">
          <text class="btn-icon">{{ isLiked ? '❤️' : '🤍' }}</text>
          <text class="btn-text">{{ isLiked ? '已收藏' : '收藏' }}</text>
        </view>
        <view class="action-btn detail" @tap="showDetail">
          <text class="btn-icon">📋</text>
          <text class="btn-text">详情</text>
        </view>
        <view class="action-btn buy" @tap="buyProduct">
          <text class="btn-icon">🛒</text>
          <text class="btn-text">购买</text>
        </view>
      </view>
    </view>

    <!-- 产品切换指示器 -->
    <view class="product-indicator" v-if="isHandDetected">
      <view 
        class="indicator-dot" 
        v-for="(product, index) in productList" 
        :key="index"
        :class="{ active: currentProductIndex === index }"
        @tap="switchProduct(index)"
      ></view>
    </view>

    <!-- 手势提示 -->
    <view class="gesture-hint" v-if="isHandDetected">
      <view class="hint-item">
        <text class="hint-icon">✌️</text>
        <text class="hint-text">切换产品</text>
      </view>
      <view class="hint-item">
        <text class="hint-icon">👆</text>
        <text class="hint-text">查看详情</text>
      </view>
      <view class="hint-item">
        <text class="hint-icon">🤏</text>
        <text class="hint-text">缩放查看</text>
      </view>
    </view>

    <!-- 底部信息面板 -->
    <view class="info-panel">
      <view class="panel-header">
        <view class="info-title">💍 手部产品销售</view>
        <view class="cart-badge" v-if="cartCount > 0">{{ cartCount }}</view>
      </view>
      <view class="info-desc">AR虚拟试戴，手势交互购物</view>

      <view class="status-section">
        <view class="status-item" v-if="!arReady">
          <view class="status-dot loading"></view>
          <text class="status-label">正在启动AR...</text>
        </view>
        <view class="status-item" v-else-if="!isHandDetected">
          <view class="status-dot scanning"></view>
          <text class="status-label">等待手部识别...</text>
        </view>
        <view class="status-item success" v-else>
          <view class="status-dot detected"></view>
          <text class="status-label">正在试戴: {{ currentProduct.name }}</text>
        </view>
      </view>

      <view class="product-list-preview">
        <view class="preview-title">产品系列 ({{ currentProductIndex + 1 }}/{{ productList.length }}):</view>
        <view class="preview-items">
          <view 
            class="preview-item" 
            v-for="(product, index) in productList" 
            :key="index"
            :class="{ active: currentProductIndex === index, viewed: product.viewed }"
            @tap="switchProduct(index)"
          >
            <view class="preview-icon">{{ product.icon }}</view>
            <view class="preview-info">
              <text class="preview-name">{{ product.name }}</text>
              <text class="preview-price">{{ product.price }}</text>
            </view>
            <view class="preview-tag" v-if="product.isNew">NEW</view>
          </view>
        </view>
      </view>

      <view class="stats-row">
        <view class="stat-box">
          <text class="stat-num">{{ viewedCount }}</text>
          <text class="stat-label">已浏览</text>
        </view>
        <view class="stat-box">
          <text class="stat-num">{{ likedCount }}</text>
          <text class="stat-label">已收藏</text>
        </view>
        <view class="stat-box">
          <text class="stat-num">{{ cartCount }}</text>
          <text class="stat-label">购物车</text>
        </view>
      </view>

      <view class="info-tips">
        <text class="tip-icon">💡</text>
        <text class="tip-text">使用手势与产品互动，体验AR购物</text>
      </view>
    </view>

    <!-- 加载状态 -->
    <view class="loading-overlay" v-if="!arReady">
      <view class="loading-spinner"></view>
      <text class="loading-text">正在启动AR试戴...</text>
    </view>

    <!-- 购买成功提示 -->
    <view class="success-toast" v-if="showSuccessToast">
      <view class="toast-content">
        <text class="toast-icon">✓</text>
        <text class="toast-text">{{ successMessage }}</text>
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
      arReady: false,
      isHandDetected: false,
      currentProductIndex: 0,
      isLiked: false,
      cartCount: 0,
      likedCount: 0,
      showSuccessToast: false,
      successMessage: '',
      toastTimer: null,
      productList: [
        {
          id: 'prod-001',
          name: '星辰钻戒',
          category: '戒指',
          icon: '💍',
          price: '¥12,999',
          description: '18K白金镶嵌1克拉钻石，璀璨如星辰',
          features: ['天然钻石', '18K白金', 'GIA认证'],
          animationClass: 'anim-rotate',
          viewed: false,
          isNew: true
        },
        {
          id: 'prod-002',
          name: '时光手表',
          category: '手表',
          icon: '⌚',
          price: '¥8,888',
          description: '瑞士机芯，蓝宝石镜面，经典永恒',
          features: ['瑞士机芯', '蓝宝石镜面', '50米防水'],
          animationClass: 'anim-float',
          viewed: false,
          isNew: false
        },
        {
          id: 'prod-003',
          name: '翡翠手镯',
          category: '手镯',
          icon: '💎',
          price: '¥28,888',
          description: '缅甸A货翡翠，温润如玉，传世之作',
          features: ['缅甸翡翠', 'A货认证', '手工雕刻'],
          animationClass: 'anim-pulse',
          viewed: false,
          isNew: true
        },
        {
          id: 'prod-004',
          name: '珍珠项链',
          category: '项链',
          icon: '📿',
          price: '¥3,999',
          description: '日本Akoya珍珠，优雅经典，百搭时尚',
          features: ['Akoya珍珠', '925银链', '正圆强光'],
          animationClass: 'anim-swing',
          viewed: false,
          isNew: false
        }
      ]
    };
  },
  computed: {
    currentProduct() {
      return this.productList[this.currentProductIndex];
    },
    viewedCount() {
      return this.productList.filter(p => p.viewed).length;
    }
  },
  onLoad() {
    const info = uni.getSystemInfoSync();
    this.width = info.windowWidth;
    this.height = info.windowHeight;
    this.renderWidth = this.width * info.pixelRatio;
    this.renderHeight = this.height * info.pixelRatio;
  },
  methods: {
    handleBack() {
      uni.navigateBack({ delta: 1 });
    },
    handleARReady() {
      this.arReady = true;
      uni.showToast({
        title: 'AR试戴已就绪',
        icon: 'success',
        duration: 1500
      });
    },
    handleHandDetected(e) {
      this.isHandDetected = true;
      this.currentProduct.viewed = true;
      
      // 解析手势
      if (e.detail && e.detail.gesture) {
        const gestureName = e.detail.gesture.name;
        this.handleGesture(gestureName);
      }
    },
    handleHandLost(e) {
      this.isHandDetected = false;
    },
    handleGesture(gestureName) {
      switch(gestureName) {
        case '比耶':
          this.nextProduct();
          break;
        case '点赞':
          if (!this.isLiked) {
            this.toggleLike();
          }
          break;
        case 'OK':
          this.showDetail();
          break;
      }
    },
    nextProduct() {
      this.currentProductIndex = (this.currentProductIndex + 1) % this.productList.length;
      this.currentProduct.viewed = true;
      this.isLiked = false;
      uni.showToast({
        title: `切换到: ${this.currentProduct.name}`,
        icon: 'none',
        duration: 1500
      });
    },
    switchProduct(index) {
      this.currentProductIndex = index;
      this.productList[index].viewed = true;
      this.isLiked = false;
    },
    toggleLike() {
      this.isLiked = !this.isLiked;
      if (this.isLiked) {
        this.likedCount++;
        this.showSuccess('已添加到收藏');
      } else {
        this.likedCount--;
      }
    },
    showDetail() {
      uni.showModal({
        title: this.currentProduct.name,
        content: `${this.currentProduct.description}\n\n价格: ${this.currentProduct.price}\n\n特点: ${this.currentProduct.features.join('、')}`,
        showCancel: false,
        confirmText: '知道了'
      });
    },
    buyProduct() {
      this.cartCount++;
      this.showSuccess(`已将 ${this.currentProduct.name} 加入购物车`);
    },
    showSuccess(message) {
      this.successMessage = message;
      this.showSuccessToast = true;
      if (this.toastTimer) {
        clearTimeout(this.toastTimer);
      }
      this.toastTimer = setTimeout(() => {
        this.showSuccessToast = false;
      }, 2000);
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

/* 手部识别引导 */
.hand-guide {
  position: absolute;
  top: 30%;
  left: 50%;
  transform: translate(-50%, -50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  z-index: 50;
}

.guide-icon {
  font-size: 120rpx;
  margin-bottom: 24rpx;
  animation: handPulse 2s infinite;
}

@keyframes handPulse {
  0%, 100% { transform: scale(1); opacity: 1; }
  50% { transform: scale(1.1); opacity: 0.8; }
}

.guide-title {
  font-size: 40rpx;
  font-weight: bold;
  color: white;
  margin-bottom: 16rpx;
}

.guide-text {
  font-size: 28rpx;
  color: rgba(255, 255, 255, 0.9);
  background: rgba(0, 0, 0, 0.6);
  padding: 16rpx 32rpx;
  border-radius: 30rpx;
  margin-bottom: 16rpx;
}

.guide-tips {
  font-size: 24rpx;
  color: rgba(255, 255, 255, 0.7);
  text-align: center;
}

/* 产品展示 */
.product-display {
  position: absolute;
  top: 25%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 40;
}

.product-model {
  position: relative;
  width: 300rpx;
  height: 300rpx;
  display: flex;
  align-items: center;
  justify-content: center;
}

.model-glow {
  position: absolute;
  width: 100%;
  height: 100%;
  background: radial-gradient(circle, rgba(255, 215, 0, 0.3) 0%, transparent 70%);
  animation: glowPulse 3s infinite;
}

@keyframes glowPulse {
  0%, 100% { transform: scale(1); opacity: 0.5; }
  50% { transform: scale(1.2); opacity: 0.8; }
}

.product-icon {
  font-size: 120rpx;
  z-index: 10;
  filter: drop-shadow(0 0 30rpx rgba(255, 215, 0, 0.8));
}

.product-rings {
  position: absolute;
  width: 100%;
  height: 100%;
}

.ring {
  position: absolute;
  border: 2rpx solid rgba(255, 215, 0, 0.4);
  border-radius: 50%;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.ring-1 {
  width: 350rpx;
  height: 350rpx;
  animation: ringRotate 4s linear infinite;
}

.ring-2 {
  width: 400rpx;
  height: 400rpx;
  animation: ringRotate 6s linear infinite reverse;
}

.ring-3 {
  width: 450rpx;
  height: 450rpx;
  animation: ringRotate 8s linear infinite;
}

@keyframes ringRotate {
  from { transform: translate(-50%, -50%) rotate(0deg); }
  to { transform: translate(-50%, -50%) rotate(360deg); }
}

/* 动画效果 */
.anim-rotate {
  animation: productRotate 4s linear infinite;
}

@keyframes productRotate {
  from { transform: rotateY(0deg); }
  to { transform: rotateY(360deg); }
}

.anim-float {
  animation: productFloat 3s ease-in-out infinite;
}

@keyframes productFloat {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-30rpx); }
}

.anim-pulse {
  animation: productPulse 2s ease-in-out infinite;
}

@keyframes productPulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.15); }
}

.anim-swing {
  animation: productSwing 3s ease-in-out infinite;
}

@keyframes productSwing {
  0%, 100% { transform: rotateZ(-10deg); }
  50% { transform: rotateZ(10deg); }
}

/* 产品信息卡片 */
.product-card {
  position: absolute;
  top: 45%;
  left: 50%;
  transform: translateX(-50%);
  width: 600rpx;
  background: rgba(255, 255, 255, 0.98);
  border-radius: 24rpx;
  padding: 30rpx;
  z-index: 60;
  box-shadow: 0 8rpx 32rpx rgba(0, 0, 0, 0.3);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20rpx;
  padding-bottom: 20rpx;
  border-bottom: 2rpx solid rgba(0, 0, 0, 0.1);
}

.product-badge {
  padding: 8rpx 20rpx;
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
  border-radius: 20rpx;
  font-size: 22rpx;
}

.product-price {
  font-size: 36rpx;
  font-weight: bold;
  color: #ff6b6b;
}

.card-body {
  margin-bottom: 24rpx;
}

.product-name {
  font-size: 32rpx;
  font-weight: bold;
  color: #333;
  margin-bottom: 12rpx;
}

.product-desc {
  font-size: 26rpx;
  color: #666;
  margin-bottom: 16rpx;
  line-height: 1.5;
}

.product-features {
  display: flex;
  flex-wrap: wrap;
  gap: 12rpx;
}

.feature-tag {
  padding: 8rpx 16rpx;
  background: rgba(102, 126, 234, 0.1);
  color: #667eea;
  border-radius: 16rpx;
  font-size: 22rpx;
}

.card-actions {
  display: flex;
  gap: 20rpx;
}

.action-btn {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20rpx;
  border-radius: 16rpx;
  transition: all 0.3s;
}

.action-btn:active {
  transform: scale(0.95);
}

.action-btn.like {
  background: rgba(255, 107, 107, 0.1);
}

.action-btn.detail {
  background: rgba(102, 126, 234, 0.1);
}

.action-btn.buy {
  background: linear-gradient(135deg, #667eea, #764ba2);
}

.btn-icon {
  font-size: 36rpx;
  margin-bottom: 8rpx;
}

.btn-text {
  font-size: 24rpx;
  color: #333;
}

.action-btn.buy .btn-text {
  color: white;
  font-weight: bold;
}

/* 产品切换指示器 */
.product-indicator {
  position: absolute;
  top: 72%;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 16rpx;
  z-index: 55;
}

.indicator-dot {
  width: 16rpx;
  height: 16rpx;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.4);
  transition: all 0.3s;
}

.indicator-dot.active {
  width: 40rpx;
  border-radius: 8rpx;
  background: #ffd700;
}

/* 手势提示 */
.gesture-hint {
  position: absolute;
  top: 76%;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 30rpx;
  z-index: 55;
}

.hint-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  background: rgba(0, 0, 0, 0.6);
  padding: 16rpx 24rpx;
  border-radius: 16rpx;
}

.hint-icon {
  font-size: 36rpx;
  margin-bottom: 8rpx;
}

.hint-text {
  font-size: 20rpx;
  color: white;
}

/* 底部信息面板 */
.info-panel {
  position: absolute;
  bottom: 40rpx;
  left: 30rpx;
  right: 30rpx;
  background: rgba(255, 255, 255, 0.98);
  border-radius: 24rpx;
  padding: 30rpx;
  z-index: 100;
}

.panel-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10rpx;
}

.info-title {
  font-size: 32rpx;
  font-weight: bold;
  color: #333;
}

.cart-badge {
  width: 40rpx;
  height: 40rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #ff6b6b;
  color: white;
  border-radius: 50%;
  font-size: 22rpx;
  font-weight: bold;
}

.info-desc {
  font-size: 26rpx;
  color: #666;
  margin-bottom: 24rpx;
}

.status-section {
  margin-bottom: 24rpx;
}

.status-item {
  display: flex;
  align-items: center;
  padding: 16rpx 20rpx;
  background: rgba(0, 0, 0, 0.05);
  border-radius: 12rpx;
}

.status-item.success {
  background: rgba(255, 215, 0, 0.15);
}

.status-dot {
  width: 16rpx;
  height: 16rpx;
  border-radius: 50%;
  margin-right: 16rpx;
}

.status-dot.loading {
  background: #faad14;
  animation: blink 1s infinite;
}

.status-dot.scanning {
  background: #1890ff;
  animation: blink 1s infinite;
}

.status-dot.detected {
  background: #ffd700;
}

@keyframes blink {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

.status-label {
  font-size: 28rpx;
  color: #333;
}

.status-item.success .status-label {
  color: #b8860b;
  font-weight: bold;
}

/* 产品列表预览 */
.product-list-preview {
  margin-bottom: 24rpx;
}

.preview-title {
  font-size: 28rpx;
  font-weight: bold;
  color: #333;
  margin-bottom: 16rpx;
  display: block;
}

.preview-items {
  display: flex;
  flex-direction: column;
  gap: 12rpx;
}

.preview-item {
  display: flex;
  align-items: center;
  padding: 16rpx 20rpx;
  background: rgba(0, 0, 0, 0.05);
  border-radius: 16rpx;
  border: 2rpx solid transparent;
  transition: all 0.3s;
  position: relative;
}

.preview-item.active {
  background: rgba(255, 215, 0, 0.15);
  border-color: #ffd700;
}

.preview-item.viewed {
  background: rgba(82, 196, 26, 0.1);
}

.preview-icon {
  font-size: 40rpx;
  margin-right: 16rpx;
}

.preview-info {
  flex: 1;
}

.preview-name {
  font-size: 28rpx;
  color: #333;
  font-weight: 500;
  display: block;
}

.preview-price {
  font-size: 24rpx;
  color: #ff6b6b;
}

.preview-tag {
  position: absolute;
  top: 8rpx;
  right: 8rpx;
  padding: 4rpx 12rpx;
  background: #ff6b6b;
  color: white;
  border-radius: 12rpx;
  font-size: 18rpx;
  font-weight: bold;
}

/* 统计行 */
.stats-row {
  display: flex;
  justify-content: space-around;
  padding: 20rpx 0;
  margin-bottom: 20rpx;
  border-top: 1rpx solid rgba(0, 0, 0, 0.1);
  border-bottom: 1rpx solid rgba(0, 0, 0, 0.1);
}

.stat-box {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.stat-num {
  font-size: 36rpx;
  font-weight: bold;
  color: #667eea;
}

.stat-label {
  font-size: 22rpx;
  color: #666;
  margin-top: 4rpx;
}

.info-tips {
  display: flex;
  align-items: center;
  padding-top: 20rpx;
}

.tip-icon {
  font-size: 28rpx;
  margin-right: 10rpx;
}

.tip-text {
  font-size: 24rpx;
  color: #666;
}

/* 加载状态 */
.loading-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  z-index: 200;
}

.loading-spinner {
  width: 80rpx;
  height: 80rpx;
  border: 6rpx solid rgba(255, 255, 255, 0.3);
  border-top-color: #ffd700;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.loading-text {
  margin-top: 30rpx;
  font-size: 28rpx;
  color: white;
}

/* 成功提示 */
.success-toast {
  position: absolute;
  top: 200rpx;
  left: 50%;
  transform: translateX(-50%);
  z-index: 150;
}

.toast-content {
  display: flex;
  align-items: center;
  background: rgba(82, 196, 26, 0.95);
  padding: 20rpx 40rpx;
  border-radius: 40rpx;
  box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.3);
}

.toast-icon {
  font-size: 32rpx;
  color: white;
  margin-right: 12rpx;
}

.toast-text {
  font-size: 28rpx;
  color: white;
  font-weight: bold;
}
</style>
