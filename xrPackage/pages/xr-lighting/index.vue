<template>
  <yn-viewer
    id="viewer"
    ref="viewer"
    :showBackBtn="true"
    :showFullScreenBtn="false"
    @ready="handleReady"
  >
    <yn-3d
      id="lighting-scene"
      :width="renderWidth"
      :height="renderHeight"
      :style="'width:' + width + 'px;height:' + height + 'px'"
      yn_camera_clear_color="0.05 0.05 0.08 1"
      @sceneReady="handleSceneReady"
    />

    <!-- 光照类型选择 -->
    <view class="lighting-panel">
      <view class="panel-header">
        <text class="panel-title">光照效果演示</text>
        <text class="panel-subtitle">切换不同光源观察效果</text>
      </view>
      
      <scroll-view class="light-list" scroll-x>
        <view 
          v-for="light in lightTypes" 
          :key="light.id"
          class="light-item"
          :class="{ active: currentLight === light.id }"
          @click="switchLight(light)"
        >
          <view class="light-icon" :style="{ background: light.gradient }">
            <text class="light-icon-text">{{ light.icon }}</text>
          </view>
          <text class="light-name">{{ light.name }}</text>
          <text class="light-desc">{{ light.shortDesc }}</text>
        </view>
      </scroll-view>
    </view>

    <!-- 光照详情 -->
    <view v-if="currentLightInfo" class="light-detail">
      <view class="detail-header">
        <text class="detail-title">{{ currentLightInfo.name }}</text>
        <view class="detail-tags">
          <text v-for="tag in currentLightInfo.tags" :key="tag" class="detail-tag">{{ tag }}</text>
        </view>
      </view>
      <text class="detail-desc">{{ currentLightInfo.description }}</text>
      
      <!-- 参数调节 -->
      <view class="param-list">
        <view class="param-item">
          <text class="param-label">强度</text>
          <slider 
            class="param-slider" 
            :value="lightIntensity" 
            min="0" 
            max="200" 
            @change="handleIntensityChange"
            activeColor="#6366f1"
            backgroundColor="rgba(255,255,255,0.2)"
          />
          <text class="param-value">{{ lightIntensity }}%</text>
        </view>
        <view class="param-item">
          <text class="param-label">颜色</text>
          <view class="color-picker">
            <view 
              v-for="color in colorOptions" 
              :key="color"
              class="color-option"
              :class="{ active: currentColor === color }"
              :style="{ background: color }"
              @click="changeLightColor(color)"
            />
          </view>
        </view>
      </view>
    </view>

    <!-- 场景物体选择 -->
    <view class="object-panel">
      <text class="object-title">展示物体</text>
      <view class="object-list">
        <view 
          v-for="obj in objectTypes" 
          :key="obj.id"
          class="object-item"
          :class="{ active: currentObject === obj.id }"
          @click="switchObject(obj)"
        >
          <text class="object-icon">{{ obj.icon }}</text>
          <text class="object-name">{{ obj.name }}</text>
        </view>
      </view>
    </view>

    <!-- 提示信息 -->
    <view class="tips-panel">
      <text class="tips-icon">💡</text>
      <text class="tips-text">{{ currentTip }}</text>
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
      currentLight: 'directional',
      currentLightInfo: null,
      currentObject: 'sphere',
      lightIntensity: 100,
      currentColor: '#ffffff',
      currentLightNode: null,
      currentObjectNode: null,
      lightTypes: [
        {
          id: 'directional',
          name: '平行光',
          icon: '☀️',
          gradient: 'linear-gradient(135deg, #FFD700 0%, #FFA500 100%)',
          shortDesc: '模拟太阳光',
          description: '平行光模拟远距离光源（如太阳），光线平行照射，产生清晰的阴影效果。适合户外场景照明。',
          tags: ['主光源', '阴影', '户外'],
          defaultIntensity: 1.0,
          defaultDirection: '1 -1 -1'
        },
        {
          id: 'point',
          name: '点光源',
          icon: '💡',
          gradient: 'linear-gradient(135deg, #FF6B6B 0%, #FFE66D 100%)',
          shortDesc: '向四周发散',
          description: '点光源从一点向四周均匀发光，光线强度随距离衰减。适合模拟灯泡、蜡烛等局部光源。',
          tags: ['局部', '衰减', '氛围'],
          defaultIntensity: 1.5,
          defaultPosition: '2 2 2'
        },
        {
          id: 'spot',
          name: '聚光灯',
          icon: '🔦',
          gradient: 'linear-gradient(135deg, #4ECDC4 0%, #44A08D 100%)',
          shortDesc: '锥形光束',
          description: '聚光灯产生锥形光束，可以控制照射范围和角度。适合舞台灯光、手电筒等效果。',
          tags: ['聚焦', '舞台', '效果'],
          defaultIntensity: 2.0,
          defaultPosition: '0 3 3',
          defaultTarget: '0 0 0',
          angle: 30
        },
        {
          id: 'ambient',
          name: '环境光',
          icon: '🌌',
          gradient: 'linear-gradient(135deg, #667eea 0%, #764ba2 100%)',
          shortDesc: '均匀照明',
          description: '环境光提供均匀的背景照明，没有方向性，不产生阴影。通常用于填充暗部和基础照明。',
          tags: ['基础', '填充', '无阴影'],
          defaultIntensity: 0.4
        },
        {
          id: 'hemisphere',
          name: '半球光',
          icon: '🌍',
          gradient: 'linear-gradient(135deg, #87CEEB 0%, #98D8C8 100%)',
          shortDesc: '天空地面渐变',
          description: '半球光模拟天空和地面的自然光照，上方为天空色，下方为地面色，产生柔和的环境效果。',
          tags: ['自然', '柔和', '环境'],
          defaultIntensity: 0.6,
          skyColor: '0.6 0.8 1',
          groundColor: '0.2 0.2 0.1'
        }
      ],
      objectTypes: [
        { id: 'sphere', name: '球体', icon: '🔵' },
        { id: 'cube', name: '立方体', icon: '🟦' },
        { id: 'torus', name: '圆环', icon: '⭕' },
        { id: 'knot', name: '扭结', icon: '🌀' }
      ],
      colorOptions: ['#ffffff', '#FFD700', '#FF6B6B', '#4ECDC4', '#9C27B0', '#00BCD4'],
      tips: {
        directional: '平行光方向决定阴影角度，适合表现时间变化',
        point: '点光源位置影响光照范围，可创建温馨氛围',
        spot: '调整聚光灯角度可控制光束宽窄',
        ambient: '环境光强度不宜过高，否则会失去立体感',
        hemisphere: '半球光适合模拟户外自然光照环境'
      }
    }
  },

  computed: {
    currentTip() {
      return this.tips[this.currentLight] || '选择不同光源观察效果差异'
    }
  },

  onLoad() {
    const sysInfo = uni.getSystemInfoSync()
    this.width = sysInfo.windowWidth
    this.height = sysInfo.windowHeight
    this.renderWidth = sysInfo.windowWidth
    this.renderHeight = sysInfo.windowHeight
    
    this.currentLightInfo = this.lightTypes[0]
  },

  methods: {
    handleReady() {
      console.log('Viewer ready')
    },

    handleSceneReady({ detail }) {
      this.scene = detail.scene
      this.camera = detail.camera
      
      if (this.camera) {
        this.camera.position.set(0, 0, 4)
        this.camera.lookAt(0, 0, 0)
      }
      
      // 初始化光照和物体
      this.setupInitialScene()
    },

    setupInitialScene() {
      if (!this.scene) return
      
      // 创建默认物体
      this.createObject('sphere')
      
      // 创建默认光照
      this.createLight('directional')
    },

    createObject(type) {
      // 清除之前的物体
      if (this.currentObjectNode) {
        this.scene.remove(this.currentObjectNode)
      }

      let geometry, material
      material = this.scene.createMaterial('StandardMaterial', {
        color: '0.8 0.8 0.8',
        metalness: '0.3',
        roughness: '0.4'
      })

      switch(type) {
        case 'sphere':
          geometry = this.scene.createGeometry('SphereGeometry', {
            radius: '1',
            widthSegments: '32',
            heightSegments: '32'
          })
          break
        case 'cube':
          geometry = this.scene.createGeometry('BoxGeometry', {
            width: '1.5',
            height: '1.5',
            depth: '1.5'
          })
          break
        case 'torus':
          geometry = this.scene.createGeometry('TorusGeometry', {
            radius: '0.8',
            tube: '0.3',
            radialSegments: '16',
            tubularSegments: '100'
          })
          break
        case 'knot':
          geometry = this.scene.createGeometry('TorusKnotGeometry', {
            radius: '0.6',
            tube: '0.2',
            tubularSegments: '64',
            radialSegments: '8'
          })
          break
      }

      if (geometry && material) {
        this.currentObjectNode = this.scene.createMesh(geometry, material)
        this.scene.add(this.currentObjectNode)
        
        // 添加旋转动画
        this.animateObject()
      }
    },

    animateObject() {
      const animate = () => {
        if (this.currentObjectNode) {
          this.currentObjectNode.rotation.y += 0.005
          this.currentObjectNode.rotation.x += 0.002
        }
        requestAnimationFrame(animate)
      }
      animate()
    },

    createLight(type) {
      // 清除之前的光照
      if (this.currentLightNode) {
        this.scene.remove(this.currentLightNode)
      }

      const lightInfo = this.lightTypes.find(l => l.id === type)
      if (!lightInfo) return

      let light
      const color = this.hexToRgb(this.currentColor)

      switch(type) {
        case 'directional':
          light = this.scene.createElement('DirectionalLight', {
            color: color,
            intensity: (lightInfo.defaultIntensity * this.lightIntensity / 100).toString(),
            direction: lightInfo.defaultDirection
          })
          break
        case 'point':
          light = this.scene.createElement('PointLight', {
            color: color,
            intensity: (lightInfo.defaultIntensity * this.lightIntensity / 100).toString(),
            position: lightInfo.defaultPosition,
            distance: '10',
            decay: '2'
          })
          break
        case 'spot':
          light = this.scene.createElement('SpotLight', {
            color: color,
            intensity: (lightInfo.defaultIntensity * this.lightIntensity / 100).toString(),
            position: lightInfo.defaultPosition,
            target: lightInfo.defaultTarget,
            angle: lightInfo.angle.toString(),
            penumbra: '0.3',
            distance: '20'
          })
          break
        case 'ambient':
          light = this.scene.createElement('AmbientLight', {
            color: color,
            intensity: (lightInfo.defaultIntensity * this.lightIntensity / 100).toString()
          })
          break
        case 'hemisphere':
          light = this.scene.createElement('HemisphereLight', {
            skyColor: lightInfo.skyColor,
            groundColor: lightInfo.groundColor,
            intensity: (lightInfo.defaultIntensity * this.lightIntensity / 100).toString()
          })
          break
      }

      if (light) {
        this.scene.add(light)
        this.currentLightNode = light
      }
    },

    switchLight(light) {
      this.currentLight = light.id
      this.currentLightInfo = light
      this.lightIntensity = 100
      this.createLight(light.id)
    },

    switchObject(obj) {
      this.currentObject = obj.id
      this.createObject(obj.id)
    },

    handleIntensityChange(e) {
      this.lightIntensity = e.detail.value
      if (this.currentLightNode) {
        const lightInfo = this.lightTypes.find(l => l.id === this.currentLight)
        if (lightInfo) {
          this.currentLightNode.intensity = lightInfo.defaultIntensity * this.lightIntensity / 100
        }
      }
    },

    changeLightColor(color) {
      this.currentColor = color
      this.createLight(this.currentLight)
    },

    hexToRgb(hex) {
      const result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex)
      if (result) {
        const r = parseInt(result[1], 16) / 255
        const g = parseInt(result[2], 16) / 255
        const b = parseInt(result[3], 16) / 255
        return `${r.toFixed(2)} ${g.toFixed(2)} ${b.toFixed(2)}`
      }
      return '1 1 1'
    }
  }
}
</script>

<style scoped>
.lighting-panel {
  position: absolute;
  bottom: 420rpx;
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

.light-list {
  white-space: nowrap;
}

.light-item {
  display: inline-flex;
  flex-direction: column;
  align-items: center;
  margin-right: 20rpx;
  padding: 20rpx;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 16rpx;
  min-width: 140rpx;
  transition: all 0.3s;
}

.light-item.active {
  background: rgba(99, 102, 241, 0.3);
  border: 2rpx solid #6366f1;
}

.light-icon {
  width: 80rpx;
  height: 80rpx;
  border-radius: 16rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 12rpx;
}

.light-icon-text {
  font-size: 40rpx;
}

.light-name {
  color: #fff;
  font-size: 24rpx;
  font-weight: 500;
}

.light-desc {
  color: rgba(255, 255, 255, 0.5);
  font-size: 20rpx;
  margin-top: 4rpx;
}

.light-detail {
  position: absolute;
  top: 180rpx;
  left: 30rpx;
  right: 30rpx;
  background: rgba(0, 0, 0, 0.7);
  backdrop-filter: blur(20rpx);
  border-radius: 24rpx;
  padding: 30rpx;
}

.detail-header {
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 16rpx;
}

.detail-title {
  color: #fff;
  font-size: 36rpx;
  font-weight: bold;
}

.detail-tags {
  flex-direction: row;
}

.detail-tag {
  background: rgba(99, 102, 241, 0.3);
  color: #a5b4fc;
  font-size: 20rpx;
  padding: 6rpx 16rpx;
  border-radius: 8rpx;
  margin-left: 10rpx;
}

.detail-desc {
  color: rgba(255, 255, 255, 0.7);
  font-size: 26rpx;
  line-height: 1.5;
  margin-bottom: 20rpx;
}

.param-list {
  padding-top: 20rpx;
  border-top: 1rpx solid rgba(255, 255, 255, 0.1);
}

.param-item {
  flex-direction: row;
  align-items: center;
  margin-bottom: 20rpx;
}

.param-label {
  color: rgba(255, 255, 255, 0.7);
  font-size: 26rpx;
  width: 100rpx;
}

.param-slider {
  flex: 1;
  margin: 0 20rpx;
}

.param-value {
  color: #fff;
  font-size: 24rpx;
  width: 80rpx;
  text-align: right;
}

.color-picker {
  flex: 1;
  flex-direction: row;
  flex-wrap: wrap;
}

.color-option {
  width: 60rpx;
  height: 60rpx;
  border-radius: 30rpx;
  margin-right: 16rpx;
  margin-bottom: 10rpx;
  border: 4rpx solid transparent;
}

.color-option.active {
  border-color: #fff;
  box-shadow: 0 0 0 2rpx #6366f1;
}

.object-panel {
  position: absolute;
  bottom: 220rpx;
  left: 30rpx;
  right: 30rpx;
  background: rgba(0, 0, 0, 0.6);
  backdrop-filter: blur(20rpx);
  border-radius: 24rpx;
  padding: 24rpx 30rpx;
}

.object-title {
  color: rgba(255, 255, 255, 0.7);
  font-size: 24rpx;
  margin-bottom: 16rpx;
  display: block;
}

.object-list {
  flex-direction: row;
  justify-content: space-around;
}

.object-item {
  align-items: center;
  padding: 16rpx 30rpx;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 16rpx;
  transition: all 0.3s;
}

.object-item.active {
  background: rgba(99, 102, 241, 0.4);
}

.object-icon {
  font-size: 40rpx;
  margin-bottom: 8rpx;
}

.object-name {
  color: #fff;
  font-size: 22rpx;
}

.tips-panel {
  position: absolute;
  bottom: 60rpx;
  left: 30rpx;
  right: 30rpx;
  background: rgba(99, 102, 241, 0.2);
  backdrop-filter: blur(10rpx);
  border-radius: 16rpx;
  padding: 20rpx 30rpx;
  flex-direction: row;
  align-items: center;
  border: 1rpx solid rgba(99, 102, 241, 0.3);
}

.tips-icon {
  font-size: 36rpx;
  margin-right: 16rpx;
}

.tips-text {
  color: rgba(255, 255, 255, 0.9);
  font-size: 24rpx;
  flex: 1;
}
</style>
