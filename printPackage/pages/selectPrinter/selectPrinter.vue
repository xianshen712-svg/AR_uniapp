<template>
    <view class="page">
        <view class="page__bd">
            <!-- Plane Track -->
            <view class="weui-panel">
                <view class="weui-panel__hd">AR高级-对手机有要求</view>
                <view class="weui-panel__bd">
                    <view class="weui-media-box weui-media-box_text">
                        <view class="weui-media-box__info">
                            <navigator url="/pocketPackage/pages/camera/camera?model=https://wifi.pro.youzewang.com/model/duye.glb" class="marginTop10">
                                <button class="weui-btn" type="primary">开始</button>
                            </navigator>
                        </view>
                    </view>
                </view>
            </view>

            <view class="weui-panel">
                <view class="weui-panel__hd">AR高级-对手机有要求</view>
                <view class="weui-panel__bd">
                    <view class="weui-media-box weui-media-box_text">
                        <view class="weui-media-box__info">
                            <navigator url="/pocketPackage/pages/camera/camera?model=https://wifi.pro.youzewang.com/model/robot.glb" class="marginTop10">
                                <button class="weui-btn" type="primary">开始</button>
                            </navigator>
                        </view>
                    </view>
                </view>
            </view>

            <view class="weui-panel">
                <view class="weui-panel__hd">AR高级-对手机有要求</view>
                <view class="weui-panel__bd">
                    <view class="weui-media-box weui-media-box_text">
                        <view class="weui-media-box__info">
                            <navigator url="/pocketPackage/pages/camera/camera?model=https://wifi.pro.youzewang.com/model/cube2.glb" class="marginTop10">
                                <button class="weui-btn" type="primary">开始</button>
                            </navigator>
                        </view>
                    </view>
                </view>
            </view>

            <!-- Measure -->
            <view class="weui-panel">
                <view class="weui-panel__hd">AR通用版-基本兼容所有机型</view>
                <view class="weui-panel__bd">
                    <view class="weui-media-box weui-media-box_text">
                        <view class="weui-media-box__info">
                            <navigator url="/pocketPackage/pages/webview/index?model=https://wifi.pro.youzewang.com/model/6.glb&collectionName=123" class="marginTop10">
                                <button class="weui-btn" type="primary">开始</button>
                            </navigator>
                        </view>
                    </view>
                </view>
            </view>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {};
    },
    onLoad(options) {
        console.log('options', options);
    },
    methods: {}
};
</script>
<style>
@import './selectPrinter.css';
</style>
