<template>
    <view>
        <yn-viewer :showBackBtn="true">
            <yn-ar
                disable-scroll
                id="main-frame"
                :width="renderWidth"
                :height="renderHeight"
                :style="'width:' + width + 'px;height:' + height + 'px'"
                yn_asset="https://uniapp.anxunxinxi.cn/model/RobotExpressive.glb"
                yn_camera_clear_color="0.89 0.69 0.94 1"
            />
        </yn-viewer>
    </view>
</template>

<script>
export default {
    data() {
        return {
            width: 300,
            height: 300,
            renderWidth: 300,
            renderHeight: 300
        };
    },
    onLoad() {
        const info = uni.getSystemInfoSync();
        const width = info.windowWidth;
        const height = info.windowHeight;
        const dpi = info.pixelRatio;
        this.width = width;
        this.height = height;
        this.renderWidth = width * dpi;
        this.renderHeight = height * dpi;
    }
};
</script>
