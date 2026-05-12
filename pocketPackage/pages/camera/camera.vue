<template>
    <view>
        <!-- 后退按钮 -->
        <cover-view @tap="backBtn_callback" :style="'position:absolute;margin-left:10px;padding-top:' + menuButtonTop + 'px;z-index:12000;'">
            <!-- 后退按钮图标 -->
            <cover-image :style="'width:' + menuButtonHeight + 'px;height:' + menuButtonHeight + 'px;'" src="/static/pocketPackage/pages/camera/img/back.png"></cover-image>
        </cover-view>

        <cover-image v-if="modelMp3Url" @tap="voiceHandel" class="voice" :src="voice ? '/static/pocketPackage/pages/camera/img/startVoice.png' : '/static/pocketPackage/pages/camera/img/stopVoice.png'"></cover-image>

        <canvas type="2d" canvas-id="canvasId" id="canvasId" style="width: 150%; height: 150%; position: absolute; left: -150%; top: 150%; z-index: -1"></canvas>

        <canvas
            id="canvas1"
            canvas-id="canvas1"
            class="canvas1"
            type="webgl"
            :disable-scroll="true"
            @touchmove="bindTouchmoveCallback"
            @touchstart="bindTouchstartCallback"
            @touchend="bindTouchendCallback"
        >
            <cover-image
                v-if="modelFlag && !loadingText && arIsStart"
                src="/static/pocketPackage/pages/camera/img/ar.gif"
                style="
                    position: absolute;
                    z-index: 10;
                    display: block;
                    text-align: center;
                    color: #fff;
                    bottom: 50%;
                    left: 0;
                    width: 600rpx;
                    height: 600rpx;
                    left: 50%;
                    margin-left: -300rpx;
                    margin-bottom: -300rpx;
                "
            ></cover-image>
            <cover-view
                v-if="modelFlag && !loadingText && arIsStart"
                style="position: absolute; width: 100%; text-align: center; left: 0; bottom: 50%; margin-bottom: -140rpx; color: #fff"
            >
                继续移动手机寻找平面
            </cover-view>
        </canvas>

        <cover-view class="time" v-if="isRecording">{{ timeText }}</cover-view>

        <cover-view :class="'btn ' + (type == 1 ? 'active' : 'btn-left')" v-if="!videoUrl && !imgUrl" @tap="selectPhoto">拍照</cover-view>
        <cover-view :class="'btn ' + (type == 2 ? 'active' : 'btn-right')" v-if="!videoUrl && !imgUrl && isShowVideo" @tap="selectVideo">拍视频</cover-view>

        <cover-image class="bottom-img" v-if="type == 1 && !videoUrl && !imgUrl" @tap="takePhoto_click" src="/static/pocketPackage/pages/camera/img/photo.png"></cover-image>

        <cover-image
            class="bottom-img"
            v-if="type == 2 && !isRecording && !videoUrl && !imgUrl"
            @touchstart="startVideo"
            src="/static/pocketPackage/pages/camera/img/video.png"
        ></cover-image>
        <cover-image
            class="bottom-img"
            v-if="type == 2 && isRecording && !videoUrl && !imgUrl"
            @touchend="endVideo"
            src="/static/pocketPackage/pages/camera/img/stop.png"
        ></cover-image>

        <video class="full video" v-if="videoUrl" autoplay="autoplay" :src="videoUrl" :controls="controls" loop="loop" :show-screen-lock-button="false"></video>
        <cover-image class="full img" v-if="imgUrl" :src="imgUrl"></cover-image>

        <cover-image class="download-btn" src="/static/pocketPackage/pages/camera/img/downloadVideo.png" @tap="download" v-if="videoUrl"></cover-image>

        <cover-image class="download-btn download-img" src="/static/pocketPackage/pages/camera/img/download.png" @tap="download" v-if="imgUrl"></cover-image>

        <cover-image class="print-btn" src="/static/pocketPackage/pages/camera/img/print.png" @tap="print" v-if="imgUrl && printShow"></cover-image>

        <cover-view class="loading" v-if="loadingText">
            <cover-image class="loading-img" src="/static/pocketPackage/pages/camera/img/loading.png"></cover-image>
            <cover-view class="text">{{ loadingText }}</cover-view>
        </cover-view>

        <cover-view class="loading-col" v-if="loadingText == '模型正在加载中'">
            <cover-image class="loading-ar" src="/static/pocketPackage/pages/camera/img/loadingAr.gif"></cover-image>
            <cover-view class="loading-progress">
                <cover-view class="progress-col" :style="'width:' + width + '%'"></cover-view>
            </cover-view>
        </cover-view>

        <cover-image class="reset" v-if="!videoUrl && !imgUrl" @tap="restart" src="/static/pocketPackage/pages/camera/img/reset.png"></cover-image>
    </view>
</template>

<script>
// threejs库
import * as THREE from './utils/three.js';
// 加载gltf库
import gLTF from './utils/gltf-loader.js';
import webglBusiness from './utils/webglBusiness';
const GLTFLoader = gLTF(THREE);

// webgl画面录制器的帧数
let recorderFPS = 42;

// webgl画面录制器的最长录制时间（单位：秒）
const recorderMaxTime = 15;

// 近截面
const NEAR = 0.001;
// 远截面
const FAR = 1000;
let isIOS = false;
let sysData = uni.getSystemInfoSync();
console.log('系统类型', sysData);
if (sysData.platform == 'ios') {
    isIOS = true;
}
let canOnePointMove = false;
let onePoint = {
    x: 0,
    y: 0
};
let twoPoint = {
    x1: 0,
    y1: 0,
    x2: 0,
    y2: 0
};

let animations;
let mixers=[];

export default {
    data() {
        return {
            videoUrl: '',
            imgUrl: '',
            typeValue: 3,
            isRecording: false,
            type: 1,
            menuButtonTop: 32,
            menuButtonHeight: 33,
            recorderText: '录制',
            modelFlag: false,
            distance: 0,
            scale: 0.04,
            timeText: '00:00',
            timeCount: 0,
            timer: null,
            loadingText: '',
            controls: false,
            isDownload: false,
            robotUrl: '',
            startRender: true,
            arIsStart: false,
            printShow: 0,
            scenicAreaId: '',
            voice: false,
            modelMp3Url: '',
            isShowVideo: true,
            width: 0,
            progressTimer: null,
            videoContext: null,
            baseUrl: ''
        };
    },
    mixins: [webglBusiness],
    onLoad(options) {
        console.log('参数', options);

            this.robotUrl='/static/models/Flamingo.glb';
            this.baseUrl=options.baseUrl;
            this.printShow=parseInt(options.printShow);

        if (options.modelMp3Url) {
                this.modelMp3Url=options.modelMp3Url
        }
        this.scenicAreaId = options.scenicAreaId;
        console.log('##', this.printShow);

        // 获取小程序右上角胶囊按钮的坐标，用作自定义导航栏。
        const menuButton = uni.getMenuButtonBoundingClientRect();
            // 胶囊按钮与手机屏幕顶端的间距
            this.menuButtonTop=menuButton.top;
            // 胶囊按钮的高度
            this.menuButtonHeight=menuButton.height;
        console.log('苹果手机', sysData.model);
      if((!isIOS)||(sysData.model.indexOf('iPhone 12 Pro')>-1)||(sysData.model.indexOf('iPhone 12<')>-1)||(sysData.model.indexOf('iPhone X')>-1)||(sysData.model.indexOf('mini')>-1)){
          this.isShowVideo=false;
      }
        console.log('onload');
        let that = this;

      wx.createSelectorQuery().select('#canvasId').fields({ node: true, size: true }).exec((res) => {
        that.canvas2d = res[0].node;
      })

        if (options.modelMp3Url) {
            console.log('音频地址', options.modelMp3Url);
            //创建内部 audio 上下文 InnerAudioContext 对象。
            this.innerAudioContext = uni.createInnerAudioContext();
            this.innerAudioContext.src = options.modelMp3Url;
            this.innerAudioContext.loop = true;
            uni.setInnerAudioOption({
                obeyMuteSwitch: false
            });
        }
    },
    onShow() {
        let that = this;
        uni.getSetting({
            success(res) {
                if (!res.authSetting['scope.camera']) {
                    //获取摄像头权限
                    uni.authorize({
                        scope: 'scope.camera',
                        success() {
                            console.log('相机授权成功');
                            if (that.arIsStart == false) {
                                console.log('重新启动ar');
                                that.startPage();
                            }
                        },
                        fail() {
                            console.log('相机授权fail');
                            uni.showModal({
                                title: '提示',
                                content: '尚未进行摄像头授权，部分功能将无法使用',
                                cancelText: '取消授权',
                                confirmText: '去授权',
                                success(res) {
                                    console.log(res);
                                    if (res.confirm) {
                                        console.log('用户点击确定');
                                        uni.openSetting({
                                            //这里的方法是调到一个添加权限的页面，这里可以测试在拒绝授权的情况下设置中是否存在相机选项
                                            success: (res) => {
                                                if (!res.authSetting['scope.camera']) {
                                                    uni.authorize({
                                                        scope: 'scope.camera',
                                                        success() {
                                                            console.log('授权成功');
                                                            if (that.arIsStart == false) {
                                                                console.log('重新启动ar');
                                                                that.startPage();
                                                            }
                                                        },
                                                        fail() {
                                                            console.log('用户点击取消');
                                                        }
                                                    });
                                                }
                                            },
                                            fail: function () {
                                                console.log('相机授权设置失败');
                                            }
                                        });
                                    } else if (res.cancel) {
                                        console.log('用户点击取消');
                                        uni.navigateBack();
                                    }
                                }
                            });
                        }
                    });
                } else {
                    console.log('有摄像头权限', that.session);
                    if (that.arIsStart == false) {
                        console.log('重新启动ar');
                        that.startPage();
                    }
                }
            },
            fail(res) {
                console.log('22222222222222222' + JSON.stringify(res));
            }
        });
    },
    onHide() {
        //wx.navigateBack()
        if (this.innerAudioContext) {
            // this.setData({
            //   voice:false
            // })
            // this.innerAudioContext.pause(); //播放音频
        }
    },
    onReady() {
        console.log('onReady');
    },
    onUnload() {
        console.log('onUnload');
        if (this.innerAudioContext) {
            this.innerAudioContext.stop();
            this.innerAudioContext = null;
        }
        if (this.progressTimer) {
            clearInterval(this.progressTimer);
        }
        this.destroy();
    },
    methods: {
        voiceHandel() {
            if (this.voice) {
                this.innerAudioContext.pause();
            } else {
                this.innerAudioContext.play();
            }
                this.voice=this.innerAudioContext.paused;
        },

        restart() {
            if (!this.loadingText) {
                this.mainModel.visible = false;
				this.modelFlag=true;
                setTimeout((res) => {
                    this.initWorldTrack();
                }, 500);
            }
        },

        startPage() {
            let that = this;
            // 获取画布组件
          wx.createSelectorQuery()
              .select('#canvas1')
              .node()
              .exec(res => {
                // 画布组件
                this.canvasDom = res[0].node
                that.init();

              })
        },

        async init() {
            let that = this;
            // 启动AR会话
            this.initEnvironment(this.canvasDom);
            this.arStart();
            // webgl画面录制器
            if (uni.getStorageSync('FpsNumber')) {
                recorderFPS = parseInt(uni.getStorageSync('FpsNumber'));
            }
            console.log('画面fps', recorderFPS);
            this.recorder = uni.createMediaRecorder(this.canvasDom, {
                fps: recorderFPS,
                // gop:1,
                // duration:15,
                videoBitsPerSecond: 3000
            });

            //解决第一帧黑屏的问题
            await this.recorder.start();
            this.recorderHandel();
            setTimeout(async function () {
                let { tempFilePath } = await that.recorder.stop();
                console.log('首次文件地址', tempFilePath);
            }, 2000);
        },

        // 启动AR会话
        initEnvironment(canvasDom) {
            let that = this;
            console.log('initEnvironment');

            // 画布组件的对象
            this.canvas = new THREE.global.registerCanvas(canvasDom);
            // 创建threejs场景
            this.initTHREE();
            console.log('开始', this.gl);
        },

        arStop() {
            console.log('结束');
            this.session.stop();
        },

        arStart() {
            var that = this;

            // 初始化webgl的背景
            this.initGL();
            // 创建AR会话
            this.session = uni.createVKSession({
                track: {
                    plane: {
                        mode: 1
                    }
                },
                version: 'v2',
                gl: this.gl
            });

            // 监视小程序窗口变化
            this.session.on('resize', function () {
                console.log('session on resize');
                that.calcCanvasSize();
            });

            // 开始AR会话
            this.session.start((err) => {
                if (err) {
                    console.log('session.start', err);
                    return;
                }
                console.log('session.start', 'ok');
				that.arIsStart=true;
				that.loadingText='模型正在加载中';

                // 设置画布的大小
                this.calcCanvasSize();
                // 加载3D模型
                that.loadModel(that.robotUrl, function () {
                    // 创建AR的坐标系
                    that.initWorldTrack(1);
                    // 加载3D模型的动画
                    that.createAnimation();
                });

                // 创建AR的坐标系
                that.initWorldTrack(1);
                // 加载3D模型的动画
                that.createAnimation();

                // 每帧渲染
                const onFrame = function (timestamp) {
                    if (!that.session) {
                        return;
                    }
                    // 从AR会话获取每帧图像
                    const frame = that.session.getVKFrame(that.canvas.width, that.canvas.height);
                    if (frame && that.startRender) {
                        // threejs渲染过程
                        that.render(frame);
                        //that.ceshi();
                    }
                    that.session.requestAnimationFrame(onFrame);
                };
                that.session.requestAnimationFrame(onFrame);
            });
        },

        // 在threejs的每帧渲染中，使用AR相机更新threejs相机的变换。
        render(frame) {
            //that.data.startRender=false;
            //console.log("开始渲染");
            // 更新threejs场景的背景
            this.renderGL(frame);
            // 更新3D模型的动画
            this.updateAnimation();
            // 从ar每帧图像获取ar相机对象
            this.ar_camera = frame.camera;
            //console.log("相机",frame.camera);
            if (this.ar_camera) {
                // 更新three.js相机对象的视图矩阵
                this.camera.matrixAutoUpdate = false;
                this.camera.matrixWorldInverse.fromArray(this.ar_camera.viewMatrix);
                this.camera.matrixWorld.getInverse(this.camera.matrixWorldInverse);

                // 更新three.js相机对象的投影矩阵
                const projectionMatrix = this.ar_camera.getProjectionMatrix(NEAR, FAR);
                this.camera.projectionMatrix.fromArray(projectionMatrix);
                this.camera.projectionMatrixInverse.getInverse(this.camera.projectionMatrix);
            }
            this.renderer.autoClearColor = false;
            // 这个是three.js相机对象
            this.renderer.render(this.scene, this.camera);
            // 保留模型的正面和背面
            this.renderer.state.setCullFace(THREE.CullFaceNone);
        },

        // 更新3D模型的动画
        updateAnimation() {
            const dt = this.clock.getDelta();
            if (mixers) {
                mixers.forEach(function (mixer) {
                    mixer.update(dt);
                });
            }
        },

        print() {
            uni.navigateTo({
                url: '/printPackage/pages/selectPrinter/selectPrinter?previewSrc=' + this.imgUrl + '&scenicAreaId=' + this.scenicAreaId
            });
        },

        // 加载3D模型
        loadModel(modelUrl, callback) {
            let that = this;
            const loader = new GLTFLoader();
            console.log('数据', loader);
            let count = 0;
            this.progressTimer = setInterval(() => {
                count = count + 2;
                if (count >= 100) {
                    clearInterval(this.progressTimer);
                }
				this.width=count;
            }, 50);
            loader.load(
                modelUrl,
                function (gltf) {
                    console.log('loadModel', 'success');
                    if (that.progressTimer) {
                        clearInterval(that.progressTimer);
                    }
					that.loadingText='';
					that.modelFlag=true;
                    that.mainModel = gltf.scene;

                    // const geometry = new THREE.BoxBufferGeometry(2, 2, 2);
                    // const texture = new THREE.TextureLoader().load('./img/tietu.jpg');
                    // const texture1 = new THREE.TextureLoader().load('./img/tietu.jpg');
                    // const material = new THREE.MeshBasicMaterial({ map: texture,size: THREE.Doubleside,
                    //   normalMap: texture1, });
                    //  //const material = new THREE.MeshBasicMaterial({ color: 0x44aa88 });
                    // const mesh = new THREE.Mesh(geometry, material);
                    // let group = new THREE.Group();
                    // var object3d = new THREE.Object3D();
                    // object3d.add(mesh);
                    // group.add(object3d);

                    // that.mainModel = group;

                    console.log('模型', gltf.animations);
                    animations = gltf.animations;
                    that.mainModel.scale.set(that.scale, that.scale, that.scale);
                    if (callback) {
                        callback();
                    }
                },
                null,
                function (error) {
                    console.log('loadModel', error);
                    //wx.hideLoading();
					          that.loadingText='';
                    console.log('加载模型失败');
                }
            );
        },

        // 创建AR的坐标系
        initWorldTrack(isAdd) {
            let that = this;
            console.log('创建ar坐标');
            // 必须 hitTest 才会创建空间坐标系
            const calPosition = function () {
                const hitTestRes = that.session.hitTest(0.5, 0.5);
                //console.log("hitTestRes",hitTestRes);
                if (hitTestRes && hitTestRes.length) {
                    console.log('initWorld ok');
                    if (that.mainModel) {
						that.modelFlag=false;
                        that.mainModel.matrixAutoUpdate = true;
                        // 将hitTest返回的transform，变换到3D模型的姿态。
                        that.mainModel.matrix.fromArray(hitTestRes[0].transform);
                        // 将矩阵分解到平移position、旋转quaternion，但不修改缩放scale。
                        that.mainModel.matrix.decompose(that.mainModel.position, that.mainModel.quaternion, new THREE.Vector3());
                        that.mainModel.visible = true;
                        // 添加模型到场景
                        if (isAdd) {
                            that.scene.add(that.mainModel);
                        }
                    }
                } else {
                    // 如果创建不成功，则1秒后重试。
                    setTimeout(calPosition, 1000);
                }
            };
            calPosition();
        },

        // 创建threejs场景
        initTHREE() {
            // 相机
            this.camera = new THREE.Camera();
            console.log('相机');
            // 场景
            this.scene = new THREE.Scene();
            console.log('场景');

            // 半球光
            this.light1 = new THREE.HemisphereLight(16777215, 4473924);
            this.light1.position.set(0, 8, 0);
            this.scene.add(this.light1);

            // 平行光
            this.light2 = new THREE.DirectionalLight(16777215);
            this.light2.position.set(0, 10, 10);
            //light2.castShadow = true;
            this.scene.add(this.light2);

            // 渲染层
            this.renderer = new THREE.WebGLRenderer({
                antialias: true,
                alpha: true
            });
            this.renderer.toneMapping = THREE.ACESFilmicToneMapping;

            // gamma色彩空间校正，以适应人眼对亮度的感觉。
            this.renderer.gammaOutput = true;
            this.renderer.gammaFactor = 2.2;

            // 时间跟踪器用作3D模型动画的更新
            this.clock = new THREE.Clock();
        },

        // 加载3D模型的动画
        createAnimation() {
            if (!this.mainModel || !animations) {
                return;
            }
            if (animations.length > 0) {
                // 动画混合器
                let mixer = new THREE.AnimationMixer(this.mainModel);
                mixer.clipAction(animations[0]).play();
                mixers.push(mixer);
            }
        },

        // 更新3D模型的动画
        updateAnimation() {
            const dt = this.clock.getDelta();
            if (mixers) {
                mixers.forEach(function (mixer) {
                    mixer.update(dt);
                });
            }
        },

        // 调整画布的大小
        calcCanvasSize() {
            console.log('calcCanvasSize');
            const info = uni.getSystemInfoSync();
            this.devicePixelRatio = info.pixelRatio;
            const width = info.windowWidth;
            const height = info.windowHeight;
            console.log('分辨率', this.devicePixelRatio);
            /* 官方示例的代码
      canvas.width = width * devicePixelRatio / 2
      canvas.height = height * devicePixelRatio / 2
      */
            this.renderer.setPixelRatio(this.devicePixelRatio);
            this.renderer.setSize(width, height);
        },

        rotateHandel(evt, resetPanel, angle) {
            this.mainModel.matrixAutoUpdate = true;
            if (angle > 0) {
                this.mainModel.rotation.y = this.mainModel.rotation.y + 0.1;
            } else {
                this.mainModel.rotation.y = this.mainModel.rotation.y - 0.1;
            }
        },

        scaleHandel(evt, resetPanel, scale) {
            this.mainModel.matrixAutoUpdate = true;
            this.mainModel.scale.set(scale, scale, scale);
        },

        destroy() {
            if (this.recorder) {
                console.log('清楚录视频');
                this.recorder.destroy();
                this.recorder = null;
            }

            // 将对象回收
            this.dispose();
                this.isRecording=false;
                this.loadingText='';
                this.timeText='00:00';
            if (this.timer) {
                clearInterval(this.timer);
            }
        },

        // 将对象回收
        dispose() {
            if (this.renderer) {
                this.renderer.clear();
                this.renderer.renderLists.dispose();
                this.renderer.dispose();
                this.renderer = null;
            }
            if (this.scene) {
                if (this.mainModel) {
                    console.log('#########', this.mainModel);
                    this.scene.remove(this.mainModel);
                }
                if (this.light1) {
                    this.scene.remove(this.light1);
                }
                if (this.light2) {
                    this.scene.remove(this.light2);
                }
                this.scene.dispose();
                this.scene = null;
            }
            if (this.camera) {
                this.camera = null;
            }
            if (this.mainModel) {
                this.mainModel = null;
            }
            if (this.loader) {
                this.loader = null;
            }
            if (mixers) {
                mixers.forEach(function (mixer) {
                    mixer.uncacheRoot(mixer.getRoot());
                });
                mixers = [];
            }
            if (this.clock) {
                this.clock = null;
            }
            if (this.canvas) {
                this.canvas = null;
            }
            if (this.session) {
                this.session.destroy();
                this.session = null;
            }
            if (this.reticle) {
                this.reticle = null;
            }
            if (this.devicePixelRatio) {
                this.devicePixelRatio = null;
            }
            if (this._program && this._program.gl) {
                this._program.gl.deleteProgram(this._program);
                this._program = null;
            }
            THREE.global.unregisterCanvas(this.canvasDom);
            this.canvasDom = null;
            console.log('清除所有数据');
        },

        bindTouchstartCallback(e) {
            if (e.touches.length < 2) {
                canOnePointMove = true;
                onePoint.x = e.touches[0].pageX * 2;
                onePoint.y = e.touches[0].pageY * 2;
            } else {
                twoPoint.x1 = e.touches[0].pageX * 2;
                twoPoint.y1 = e.touches[0].pageY * 2;
                twoPoint.x2 = e.touches[1].pageX * 2;
                twoPoint.y2 = e.touches[1].pageY * 2;
                // 当两根手指放上去的时候，将距离(distance)初始化。
                let xMove = e.touches[1].clientX - e.touches[0].clientX;
                let yMove = e.touches[1].clientY - e.touches[0].clientY;
                //计算开始触发两个手指坐标的距离
                let distance = Math.sqrt(xMove * xMove + yMove * yMove);
                this.distance = distance;
            }
        },

        bindTouchendCallback: function (e) {
            canOnePointMove = false;
        },

        bindTouchmoveCallback(e) {
            console.log('bindtouchmove_callback');
            // 在手指点击的位置放置3D模型
            if (e.touches.length == 1 && canOnePointMove) {
                this.addModelByHitTest(e, false, false);
            } else if (e.touches.length == 2) {
                var preTwoPoint = JSON.parse(JSON.stringify(twoPoint));
                twoPoint.x1 = e.touches[0].pageX * 2;
                twoPoint.y1 = e.touches[0].pageY * 2;
                twoPoint.x2 = e.touches[1].pageX * 2;
                twoPoint.y2 = e.touches[1].pageY * 2;

                // 计算角度，旋转(优先)

                var perAngle = (Math.atan((preTwoPoint.y1 - preTwoPoint.y2) / (preTwoPoint.x1 - preTwoPoint.x2)) * 180) / Math.PI;
                var curAngle = (Math.atan((twoPoint.y1 - twoPoint.y2) / (twoPoint.x1 - twoPoint.x2)) * 180) / Math.PI;
                let angle = perAngle - curAngle;
                if (Math.abs(angle) > 1.2 && Math.abs(angle) < 100) {
                    console.log('旋转', (angle * Math.PI) / 180);
                    this.rotateHandel(e, false, (angle * Math.PI) / 180);
                } else {
                    //双手指运动 x移动后的坐标和y移动后的坐标
                    let xMove = e.touches[1].clientX - e.touches[0].clientX;
                    let yMove = e.touches[1].clientY - e.touches[0].clientY;

                    //双手指运动新的 ditance
                    let distance = Math.sqrt(xMove * xMove + yMove * yMove);
                    //计算移动的过程中实际移动了多少的距离
                    let distanceDiff = distance - this.distance;
                    this.distance = distance;
                    // 为了防止缩放得太大，所以scale需要限制，同理最小值也是
                    if (distanceDiff >= 0) {
                        if (this.scale < 0.4) {
                            this.scale = this.scale * 1.03;
                            this.scaleHandel(e, false, this.scale);
                        }
                    }
                    //为了防止缩放得太小，所以scale需要限制
                    else if (distanceDiff < 0) {
                        if (this.scale > 0.001) {
                            this.scale = this.scale * 0.97;
                            this.scaleHandel(e, false, this.scale);
                        }
                    }
                }
            }
        },

        // 在手指点击的位置放置3D模型
        // resetPanel：是否用现实环境中新的平面作为AR的空间坐标系
        // evt：触摸事件
        // isAddModel:是否将3D模型加入到threejs场景
        addModelByHitTest(evt, resetPanel, isAddModel) {
            // 点击可移动3D模型位置
            const touches = evt.changedTouches.length ? evt.changedTouches : evt.touches;
            if (touches.length === 1) {
                const touch = touches[0];
                const hitTestRes = this.session.hitTest((touch.x * this.devicePixelRatio) / this.canvas.width, (touch.y * this.devicePixelRatio) / this.canvas.height, resetPanel);
                if (hitTestRes && hitTestRes.length) {
                    this.mainModel.matrixAutoUpdate = true;
                    this.mainModel.matrix.fromArray(hitTestRes[0].transform);
                    // 将矩阵分解到平移position、旋转quaternion，但不修改缩放scale。
                    this.mainModel.matrix.decompose(this.mainModel.position, new THREE.Quaternion(), new THREE.Vector3());
                    console.log('addModelByHitTest', this.mainModel.position);
                }
            }
        },

        renderSceneFun() {
            this.renderer.clear();
            const frameData = this.session.getVKFrame(this.canvas.width, this.canvas.height);
            this.render(frameData);
        },

        ceshi() {
            let t;
            let width = this.gl.drawingBufferWidth;
            let height = this.gl.drawingBufferHeight;
            t = new THREE.WebGLRenderTarget(width, height);
            this.renderer.setRenderTarget(t);
            this.renderSceneFun();
            this.renderer.readRenderTargetPixels(t, 0, 0, width, height, pixels);
            this.renderer.setRenderTarget(null);
            t.dispose();
        },

        readPixelsFun() {
            let t;
            let width = this.gl.drawingBufferWidth;
            let height = this.gl.drawingBufferHeight;
            var pixels = new Uint8Array(width * height * 4);
            console.log('是否ios', isIOS);
            if (isIOS) {
                t = new THREE.WebGLRenderTarget(width, height, {
                    encoding: THREE.sRGBEncoding,
                    minFilter: THREE.LinearFilter,
                    magFilter: THREE.LinearFilter,
                    wrapS: THREE.ClampToEdgeWrapping,
                    wrapT: THREE.ClampToEdgeWrapping,
                    format: THREE.RGBAFormat,
                    type: THREE.UnsignedByteType
                });
                this.renderer.setRenderTarget(t);

                //this.data.startRender=false;
                this.renderSceneFun();
                this.renderer.readRenderTargetPixels(t, 0, 0, width, height, pixels);
                this.renderer.setRenderTarget(null);
                t.dispose();
            } else {
                this.renderSceneFun();
                this.gl.readPixels(0, 0, width, height, this.gl.RGBA, this.gl.UNSIGNED_BYTE, pixels);
            }
            var a = new Uint8ClampedArray(width * height * 4);
            var A = 4 * width;
            for (var s = 0; s < height; s += 1) {
                var c = s * A;
                a.set(pixels.slice(c, c + A), (height - s - 1) * A);
            }
            return [a, width, height];
        },

        async takePhoto_click(e) {
            //this.data.startRender=false;

			this.loadingText='拍照中';
            console.log('机型', sysData.model); //有些机型拍照白屏，从视频里面取图片
            if (sysData.model.indexOf('iPhone 13 Pro Max') > -1) {
                //if(true)
                // 开始录制
                this.startRender = false;
                await this.recorder.start();
                let frames = 20;
                // 逐帧绘制
                while (frames--) {
                    await this.recorder.requestFrame();
                }
                // 停止录制，生成视频的地址
                const { tempFilePath } = await this.recorder.stop();
                console.log('拍照文件地址', tempFilePath);
                this.handelVideo(tempFilePath);
                return;
            }
            let imgData = this.readPixelsFun();
            let that = this;
            console.log(imgData);
            console.log(this.canvas2d);
            const ctx = this.canvas2d.getContext('2d');
            console.log(ctx);
            const img = this.canvas2d.createImageData(imgData[0], imgData[1], imgData[2]);
            this.canvas2d.width = imgData[1];
            this.canvas2d.height = imgData[2];
            ctx.putImageData(img, 0, 0);
            console.log('图片', this.canvas2d.width);
            uni.canvasToTempFilePath({
                quality: 1,
                destWidth: that.canvas2d.width,
                destHeight: that.canvas2d.height,
                canvas: that.canvas2d,
                success(res) {
                    console.log('photo绘制成功', res);

					that.loadingText='';
					that.imgUrl=res.tempFilePath;
                    /* wx.previewImage({
             current: res.tempFilePath, // 当前显示图片的http链接
             urls: [res.tempFilePath] // 需要预览的图片http链接列表
           })*/
                },
                fail(err) {
                    console.log(err);
                },
                complete() {} // end complete
            });
        },

        async startRecord() {
                this.recorderText='停止';

            //this.data.startRender=false;

            if (this.timer) {
                clearInterval(this.timer);
            }
            this.timeCount = 0;
            this.timer = setInterval((res) => {
                if (this.timeCount >= 15) {
                    this.stopRecord();
                }
                this.timeCount++;
                let timeText = this.timeCount;
                if (timeText < 10) {
                    timeText = '0' + timeText;
                }
                timeText = '00:' + timeText;
				this.timeText=timeText;
            }, 1000);

            // 开始录制
            await this.recorder.start();
            // 录制 30s 的视频
            let frames = recorderFPS * recorderMaxTime;
            this.recorderHandel();
        },

        async recorderHandel() {
            // 逐帧绘制
            while (this.isRecording) {
                await this.recorder.requestFrame();
                // const frameData = this.session.getVKFrame(this.canvas.width, this.canvas.height)
                // this.render(frameData);

                //  if(!isIOS){
                //    console.log("安卓");
                //    this._renderScene();
                //  }
            }
        },

        downloadImg(imgUrl) {
            var that = this;
            uni.saveImageToPhotosAlbum({
                filePath: imgUrl,
                //图片文件路径
                success: function (data) {
                    that.isDownload = true;
                    uni.showToast({
                        title: '图片已保存到相册',
                        icon: 'none',
                        duration: 2000
                    });
                },
                // 接口调用失败的回调函数
                fail: function (err) {
                    console.log('错误', err);
                    if (
                        err.errMsg === 'saveImageToPhotosAlbum:fail:auth denied' ||
                        err.errMsg === 'saveImageToPhotosAlbum:fail auth deny' ||
                        err.errMsg === 'saveImageToPhotosAlbum:fail authorize no response'
                    ) {
                        uni.showModal({
                            title: '提示',
                            content: '需要您授权保存相册',
                            modalType: false,
                            success: (modalSuccess) => {
                                uni.openSetting({
                                    success(settingdata) {
                                        console.log('settingdata', settingdata);
                                        if (settingdata.authSetting['scope.writePhotosAlbum']) {
                                            uni.showModal({
                                                title: '提示',
                                                content: '获取权限成功,再次点击图片即可保存',
                                                modalType: false
                                            });
                                        } else {
                                            uni.showModal({
                                                title: '提示',
                                                content: '获取权限失败，将无法保存到相册哦~',
                                                modalType: false
                                            });
                                        }
                                    },
                                    fail(failData) {
                                        console.log('failData', failData);
                                    },
                                    complete(finishData) {
                                        console.log('finishData', finishData);
                                    }
                                });
                            }
                        });
                    }
                },
                complete(res) {}
            });
        },

        async handelVideo(tempFilePath) {
            var that = this;
            //上传图片
            uni.uploadFile({
                //请求后台的路径
                url: this.baseUrl + '/app/sign/management/upload',
                //小程序本地的路径
                filePath: tempFilePath,
                header: {
                    appId: '1581897423341428736',
                    type: 3,
                    apptoken: uni.getStorageSync('userInfo').apptoken
                },
                //后台获取我们图片的key
                name: 'file',
                success: function (res) {
                    let data = JSON.parse(res.data);
                    console.log(data);
                    let imgUrl = 'https://yjkjtest11.oss-cn-hangzhou.aliyuncs.com/' + data.data + '?x-oss-process=video/snapshot,t_2,f_jpg';
                    //imgUrl="https://athletesactivity.oss-cn-shanghai.aliyuncs.com/file/123.mp4?x-oss-process=video/snapshot,t_0,f_jpg";
                    console.log('图片地址', imgUrl);
                        that.loadingText='';
                        that.imgUrl=imgUrl;
                    that.startRender = true;
                },
                fail: function (res) {
                    uni.showToast({
                        title: '拍照失败',
                        icon: 'none'
                    });
					that.loadingText='';
                    that.startRender = true;
                }
            });
        },

        uploadVideo() {
            var that = this;
            uni.chooseVideo({
                count: 1,
                //最多还能上传的图片数,这里最多可以上传5张
                sizeType: ['original', 'compressed'],
                //可以指定是原图还是压缩图,默认二者都有
                success: function (res) {
                    console.log('文件', res);
                    var imgList = res.tempFilePath;
                    console.log(imgList);
                    var tempFilePath = imgList;
                    that.handelVideo(tempFilePath);
                },
                fail: function () {
                    uni.showToast({
                        title: '图片上传失败',
                        icon: 'none'
                    });
                    uni.hideLoading();
                    return;
                }
            });
        },

        async stopRecord() {
            if (this.timer) {
                clearInterval(this.timer);
            }
            // 停止录制，生成视频的地址
            const { tempFilePath } = await this.recorder.stop();
            console.log('文件地址', tempFilePath);
                this.videoUrl=tempFilePath;
                this.isRecording=false;
                this.timeText='00:00';
        },

        download() {
            var that = this;
            if (this.videoUrl) {
                // 保存视频到手机相册
                uni.saveVideoToPhotosAlbum({
                    filePath: this.videoUrl,
                    success(res) {
                        console.log('wx.saveVideoToPhotosAlbum', res.errMsg);
                        if (res.errMsg === 'saveVideoToPhotosAlbum:ok') {
                            var message = '视频已保存到相册';
                            that.isDownload = true;
                            uni.showToast({
                                title: message,
                                icon: 'none',
                                duration: 2000
                            });
                        }
                    }
                });
            } else {
                console.log('图片地址', this.imgUrl);
                if (sysData.model.indexOf('iPhone 13 Pro Max') > -1) {
                    uni.downloadFile({
                        url: this.imgUrl,
                        //图片的地址
                        success: function (res) {
                            const tempFilePath = res.tempFilePath;
                            that.downloadImg(tempFilePath);
                        },
                        fail: function (err) {
                            uni.showToast({
                                title: '图片保存失败',
                                icon: 'none',
                                duration: 2000
                            });
                        }
                    });
                } else {
                    this.downloadImg(this.imgUrl);
                }
            }
        },

        // 画面录制按钮
        async startVideo() {
            if (!this.isRecording) {
				this.isRecording=true;
                await this.startRecord();
            }
        },

        async endVideo() {
            if (this.timeCount >= 2) {
                await this.stopRecord();
            }
        },

        selectVideo() {
                this.type=2;
        },

        selectPhoto() {
            if (this.isRecording) {
                uni.showToast({
                    title: '录制中不能拍照',
                    icon: 'none'
                });
                return;
            }
			this.type=1;
        },

        // 后退按钮的点击事件
        backBtn_callback() {
            console.log('返回');
            if (this.isRecording) {
                uni.showToast({
                    title: '录制中不能后退',
                    icon: 'none'
                });
                return;
            }
            if (this.imgUrl || this.videoUrl) {
                let that = this;
                if (that.isDownload) {
					that.imgUrl='';
					that.videoUrl='';
                    that.isDownload = false;
                    //that.data.startRender=true;
                } else {
                    uni.showModal({
                        title: '提示',
                        content: '返回后拍摄内容将不会保留',
                        success(res) {
                            if (res.confirm) {
								that.imgUrl='';
								that.videoUrl='';
                                that.isDownload = false;
                                //that.data.startRender=true;
                            } else if (res.cancel) {
                                console.log('用户点击取消');
                            }
                        }
                    });
                }
            } else {
                uni.navigateBack();
            }
        }
    }
};
</script>
<style>
@import './camera.css';
</style>
