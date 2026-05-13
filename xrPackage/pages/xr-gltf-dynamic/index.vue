<template>
  <yn-viewer
    id="viewer"
    ref="viewer"
    :showBackBtn="true"
    :showFullScreenBtn="false"
    @ready="handleReady"
  >
    <yn-3d
      id="dynamic-model"
      :width="renderWidth"
      :height="renderHeight"
      :style="'width:' + width + 'px;height:' + height + 'px'"
      yn_camera_clear_color="0.1 0.1 0.15 1"
      @sceneReady="handleSceneReady"
    />

    <!-- 模型选择面板 -->
    <view class="model-panel">
      <view class="panel-header">
        <text class="panel-title">动态模型加载</text>
        <text class="panel-subtitle">运行时切换不同glTF模型</text>
      </view>
      
      <scroll-view class="model-list" scroll-x>
        <view 
          v-for="model in modelList" 
          :key="model.id"
          class="model-item"
          :class="{ active: currentModel === model.id, loading: loading && currentModel === model.id }"
          @click="loadModel(model)"
        >
          <view class="model-preview" :style="{ background: model.color }">
            <text class="model-icon">{{ model.icon }}</text>
          </view>
          <text class="model-name">{{ model.name }}</text>
          <text class="model-size">{{ model.size }}</text>
          <view v-if="loading && currentModel === model.id" class="loading-indicator">
            <view class="loading-spinner"></view>
          </view>
        </view>
      </scroll-view>
    </view>

    <!-- 加载进度 -->
    <view v-if="loading" class="progress-overlay">
      <view class="progress-container">
        <text class="progress-text">加载中...</text>
        <view class="progress-bar">
          <view class="progress-fill" :style="{ width: progress + '%' }"></view>
        </view>
        <text class="progress-percent">{{ progress }}%</text>
      </view>
    </view>

    <!-- 模型信息卡片 -->
    <view v-if="currentModelInfo && !loading" class="info-card">
      <view class="info-header">
        <text class="info-name">{{ currentModelInfo.name }}</text>
        <view class="info-tags">
          <text v-for="tag in currentModelInfo.tags" :key="tag" class="info-tag">{{ tag }}</text>
        </view>
      </view>
      <text class="info-desc">{{ currentModelInfo.description }}</text>
      <view class="info-stats">
        <view class="stat-item">
          <text class="stat-value">{{ currentModelInfo.vertices }}</text>
          <text class="stat-label">顶点数</text>
        </view>
        <view class="stat-item">
          <text class="stat-value">{{ currentModelInfo.triangles }}</text>
          <text class="stat-label">三角面</text>
        </view>
        <view class="stat-item">
          <text class="stat-value">{{ currentModelInfo.materials }}</text>
          <text class="stat-label">材质数</text>
        </view>
      </view>
    </view>

    <!-- 控制按钮 -->
    <view class="control-panel">
      <view class="control-btn" @click="toggleAnimation">
        <text class="btn-icon">{{ isAnimating ? '⏸️' : '▶️' }}</text>
        <text class="btn-text">{{ isAnimating ? '暂停' : '播放' }}</text>
      </view>
      <view class="control-btn" @click="resetCamera">
        <text class="btn-icon">🎥</text>
        <text class="btn-text">重置视角</text>
      </view>
      <view class="control-btn" @click="toggleWireframe">
        <text class="btn-icon">🔲</text>
        <text class="btn-text">线框模式</text>
      </view>
    </view>
  </yn-viewer>
</template>

<script>
export default {
  data() {
    return {
      width: 300,
      height: 300,
      renderWidth: 300,
      renderHeight: 300,
      scene: null,
      camera: null,
      currentModel: null,
      currentModelInfo: null,
      loading: false,
      progress: 0,
      isAnimating: true,
      isWireframe: false,
      loadedModels: {},
      modelList: [
        {
          id: 'helmet',
          name: '飞行头盔',
          icon: '🪖',
          color: 'linear-gradient(135deg, #667eea 0%, #764ba2 100%)',
          size: '2.3 MB',
          url: 'https://mmbizwxaminiprogram-1258344707.cos.ap-guangzhou.myqcloud.com/xr-frame/demo/glTF/DamagedHelmet/glTF/DamagedHelmet.gltf',
          description: '受损的科幻飞行头盔，展示PBR材质和法线贴图效果',
          tags: ['PBR', '法线贴图', '金属'],
          vertices: '15.2K',
          triangles: '8.4K',
          materials: '5'
        },
        {
          id: 'suzanne',
          name: '苏珊娜',
          icon: '🐵',
          color: 'linear-gradient(135deg, #f093fb 0%, #f5576c 100%)',
          size: '1.1 MB',
          url: 'https://mmbizwxaminiprogram-1258344707.cos.ap-guangzhou.myqcloud.com/xr-frame/demo/glTF/Suzanne/Suzanne.gltf',
          description: '经典的Blender猴子模型，用于测试渲染效果',
          tags: ['经典', '低模', '测试'],
          vertices: '2.0K',
          triangles: '968',
          materials: '1'
        },
        {
          id: 'duck',
          name: '小黄鸭',
          icon: '🦆',
          color: 'linear-gradient(135deg, #4facfe 0%, #00f2fe 100%)',
          size: '0.8 MB',
          url: 'https://mmbizwxaminiprogram-1258344707.cos.ap-guangzhou.myqcloud.com/xr-frame/demo/glTF/Duck/Duck.gltf',
          description: '可爱的黄色橡皮鸭，glTF格式标准测试模型',
          tags: ['可爱', '标准', '动画'],
          vertices: '4.2K',
          triangles: '2.1K',
          materials: '2'
        },
        {
          id: 'box',
          name: '纹理盒子',
          icon: '📦',
          color: 'linear-gradient(135deg, #43e97b 0%, #38f9d7 100%)',
          size: '0.3 MB',
          url: 'https://mmbizwxaminiprogram-1258344707.cos.ap-guangzhou.myqcloud.com/xr-frame/demo/glTF/BoxTextured/BoxTextured.gltf',
          description: '带纹理的立方体，展示基础纹理映射',
          tags: ['基础', '纹理', '立方体'],
          vertices: '24',
          triangles: '12',
          materials: '1'
        },
        {
          id: 'boombox',
          name: '复古音响',
          icon: '📻',
          color: 'linear-gradient(135deg, #fa709a 0%, #fee140 100%)',
          size: '5.6 MB',
          url: 'https://mmbizwxaminiprogram-1258344707.cos.ap-guangzhou.myqcloud.com/xr-frame/demo/glTF/BoomBox/BoomBox.gltf',
          description: '80年代复古风格音响，精细的金属和塑料材质',
          tags: ['复古', '精细', '金属'],
          vertices: '36.5K',
          triangles: '18.2K',
          materials: '8'
        }
      ]
    }
  },

  onLoad() {
    const sysInfo = uni.getSystemInfoSync()
    this.width = sysInfo.windowWidth
    this.height = sysInfo.windowHeight
    this.renderWidth = sysInfo.windowWidth
    this.renderHeight = sysInfo.windowHeight
  },

  methods: {
    handleReady() {
      console.log('Viewer ready')
    },

    handleSceneReady({ detail }) {
      this.scene = detail.scene
      this.camera = detail.camera
      
      // 设置相机位置
      if (this.camera) {
        this.camera.position.set(0, 0, 3)
        this.camera.lookAt(0, 0, 0)
      }
      
      // 添加基础光照
      this.setupLighting()
      
      // 加载默认模型
      this.loadModel(this.modelList[0])
    },

    setupLighting() {
      if (!this.scene) return
      
      // 环境光
      const ambientLight = this.scene.createElement('AmbientLight', {
        color: '1 1 1',
        intensity: '0.4'
      })
      this.scene.add(ambientLight)
      
      // 主光源
      const dirLight = this.scene.createElement('DirectionalLight', {
        color: '1 1 1',
        intensity: '1.0',
        direction: '1 -1 -1'
      })
      this.scene.add(dirLight)
      
      // 补光
      const fillLight = this.scene.createElement('DirectionalLight', {
        color: '0.8 0.9 1',
        intensity: '0.5',
        direction: '-1 0.5 -0.5'
      })
      this.scene.add(fillLight)
    },

    async loadModel(model) {
      if (this.currentModel === model.id && this.loadedModels[model.id]) {
        return
      }

      this.currentModel = model.id
      this.loading = true
      this.progress = 0

      try {
        // 清除之前的模型
        this.clearCurrentModel()

        // 模拟加载进度
        const progressInterval = setInterval(() => {
          if (this.progress < 90) {
            this.progress += Math.random() * 15
            if (this.progress > 90) this.progress = 90
          }
        }, 200)

        // 加载新模型
        const gltfModel = await this.scene.loadGLTF(model.url)
        
        clearInterval(progressInterval)
        this.progress = 100

        if (gltfModel) {
          // 调整模型位置和大小
          gltfModel.position.set(0, 0, 0)
          gltfModel.scale.set(1, 1, 1)
          
          // 添加到场景
          this.scene.add(gltfModel)
          
          // 保存引用
          this.loadedModels[model.id] = gltfModel
          this.currentModelNode = gltfModel
          this.currentModelInfo = model
          
          // 启动旋转动画
          this.startRotation()
        }

        setTimeout(() => {
          this.loading = false
        }, 300)

      } catch (error) {
        console.error('模型加载失败:', error)
        uni.showToast({
          title: '模型加载失败',
          icon: 'none'
        })
        this.loading = false
      }
    },

    clearCurrentModel() {
      if (this.currentModelNode) {
        this.scene.remove(this.currentModelNode)
        this.currentModelNode = null
      }
      this.stopRotation()
    },

    startRotation() {
      if (!this.isAnimating) return
      
      const animate = () => {
        if (this.currentModelNode && this.isAnimating) {
          this.currentModelNode.rotation.y += 0.01
          requestAnimationFrame(animate)
        }
      }
      animate()
    },

    stopRotation() {
      this.isAnimating = false
    },

    toggleAnimation() {
      this.isAnimating = !this.isAnimating
      if (this.isAnimating) {
        this.startRotation()
      }
    },

    resetCamera() {
      if (this.camera) {
        this.camera.position.set(0, 0, 3)
        this.camera.lookAt(0, 0, 0)
      }
    },

    toggleWireframe() {
      this.isWireframe = !this.isWireframe
      if (this.currentModelNode) {
        // 切换线框模式
        this.currentModelNode.traverse((node) => {
          if (node.material) {
            node.material.wireframe = this.isWireframe
          }
        })
      }
    }
  }
}
</script>

<style scoped>
.model-panel {
  position: absolute;
  bottom: 200rpx;
  left: 30rpx;
  right: 30rpx;
  background: rgba(0, 0, 0, 0.7);
  backdrop-filter: blur(20rpx);
  border-radius: 24rpx;
  padding: 30rpx;
}

.panel-header {
  margin-bottom: 20rpx;
}

.panel-title {
  display: block;
  color: #fff;
  font-size: 32rpx;
  font-weight: bold;
}

.panel-subtitle {
  display: block;
  color: rgba(255, 255, 255, 0.6);
  font-size: 24rpx;
  margin-top: 8rpx;
}

.model-list {
  white-space: nowrap;
}

.model-item {
  display: inline-flex;
  flex-direction: column;
  align-items: center;
  margin-right: 20rpx;
  padding: 20rpx;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 16rpx;
  min-width: 140rpx;
  position: relative;
  transition: all 0.3s;
}

.model-item.active {
  background: rgba(99, 102, 241, 0.3);
  border: 2rpx solid #6366f1;
}

.model-item.loading {
  opacity: 0.7;
}

.model-preview {
  width: 80rpx;
  height: 80rpx;
  border-radius: 16rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 12rpx;
}

.model-icon {
  font-size: 40rpx;
}

.model-name {
  color: #fff;
  font-size: 24rpx;
  font-weight: 500;
}

.model-size {
  color: rgba(255, 255, 255, 0.5);
  font-size: 20rpx;
  margin-top: 4rpx;
}

.loading-indicator {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.loading-spinner {
  width: 40rpx;
  height: 40rpx;
  border: 4rpx solid rgba(255, 255, 255, 0.3);
  border-top-color: #6366f1;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.progress-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
}

.progress-container {
  background: rgba(30, 30, 40, 0.95);
  border-radius: 24rpx;
  padding: 60rpx 80rpx;
  align-items: center;
}

.progress-text {
  color: #fff;
  font-size: 32rpx;
  font-weight: 500;
  margin-bottom: 30rpx;
}

.progress-bar {
  width: 400rpx;
  height: 12rpx;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 6rpx;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #6366f1, #8b5cf6);
  border-radius: 6rpx;
  transition: width 0.3s ease;
}

.progress-percent {
  color: #6366f1;
  font-size: 48rpx;
  font-weight: bold;
  margin-top: 20rpx;
}

.info-card {
  position: absolute;
  top: 180rpx;
  left: 30rpx;
  right: 30rpx;
  background: rgba(0, 0, 0, 0.7);
  backdrop-filter: blur(20rpx);
  border-radius: 24rpx;
  padding: 30rpx;
}

.info-header {
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 16rpx;
}

.info-name {
  color: #fff;
  font-size: 36rpx;
  font-weight: bold;
}

.info-tags {
  flex-direction: row;
}

.info-tag {
  background: rgba(99, 102, 241, 0.3);
  color: #a5b4fc;
  font-size: 20rpx;
  padding: 6rpx 16rpx;
  border-radius: 8rpx;
  margin-left: 10rpx;
}

.info-desc {
  color: rgba(255, 255, 255, 0.7);
  font-size: 26rpx;
  line-height: 1.5;
  margin-bottom: 20rpx;
}

.info-stats {
  flex-direction: row;
  justify-content: space-around;
  padding-top: 20rpx;
  border-top: 1rpx solid rgba(255, 255, 255, 0.1);
}

.stat-item {
  align-items: center;
}

.stat-value {
  color: #fff;
  font-size: 32rpx;
  font-weight: bold;
}

.stat-label {
  color: rgba(255, 255, 255, 0.5);
  font-size: 22rpx;
  margin-top: 4rpx;
}

.control-panel {
  position: absolute;
  bottom: 60rpx;
  left: 30rpx;
  right: 30rpx;
  flex-direction: row;
  justify-content: space-around;
}

.control-btn {
  background: rgba(0, 0, 0, 0.6);
  backdrop-filter: blur(10rpx);
  border-radius: 16rpx;
  padding: 20rpx 40rpx;
  align-items: center;
  border: 1rpx solid rgba(255, 255, 255, 0.1);
}

.btn-icon {
  font-size: 36rpx;
  margin-bottom: 8rpx;
}

.btn-text {
  color: #fff;
  font-size: 22rpx;
}
</style>
