<template>
    <view class="page">
        <view class="page__bd">
            <web-view :src="webviewUrl"></web-view>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            webviewUrl: ''
        };
    },
    onLoad(options) {
        console.log(options);
        let apiUrl = uni.getStorageSync('baseUrl');
        this.setData({
            //webviewUrl: `https://wifi.pro.youzewang.com/ar_h5?apptoken=30bd1652-52b1-44c3-a851-8cf9904e56e71667897051179&model=${options.model}`
            webviewUrl: `https://wifi.pro.youzewang.com/ar_h5?collectionName=${options.collectionName}&apptoken=${uni.getStorageSync('userInfo').apptoken}&model=${
                options.model
            }&printShow=${options.printShow}&modelMp3Url=${options.modelMp3Url}&apiUrl=${apiUrl}`
        });
    },
    onUnload() {
        console.log('onUnload');
    },
    methods: {}
};
</script>
<style>
@import './index.css';
</style>
