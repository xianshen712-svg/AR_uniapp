<template>
    <view class="page">
        <view class="page__bd">
            <!-- 当前 AR (VK Session + Three.js) -->
            <view class="weui-panel">
                <view class="weui-panel__hd">当前 AR（VK + Three.js）</view>
                <view class="weui-panel__bd">
                    <view class="weui-media-box weui-media-box_text">
                        <view class="weui-media-box__info">
                            <navigator
                                url="/pocketPackage/pages/camera/camera?model=https://uniapp.anxunxinxi.cn/model/RobotExpressive.glb&&printShow=1&modelMp3Url=https://wifi.pro.youzewang.com/y860.mp3&scenicAreaId=123&baseUrl=https://yjnft-api.test.yjkjmeta.com"
                                class="marginTop10"
                            >
                                <button class="weui-btn" type="primary">开始当前 AR</button>
                            </navigator>
                        </view>
                    </view>
                </view>
            </view>

            <!-- XR-Frame AR -->
            <view class="weui-panel">
                <view class="weui-panel__hd">XR-Frame AR（云诺插件）</view>
                <view class="weui-panel__bd">
                    <view class="weui-media-box weui-media-box_text">
                        <view class="weui-media-box__info">
                            <navigator url="/pocketPackage/pages/xr-card-list/index" class="marginTop10">
                                <button class="weui-btn" type="warn">XR 能力演示</button>
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
    onLoad() {
        console.log('开始');
        uni.setStorageSync('userInfo', {
            apptoken: 'ac27a546-0321-43f3-991b-b2d8fb33e50f1671263115455',
            id: '1590170977758183424',
            companyId: '1581897423341428736',
            nickname: '16621251116',
            userAvatar: 'public/upload/20221129172022704.jpg',
            phone: '166****1116',
            password: '',
            authentication: 0,
            authenticationTime: null,
            realName: '',
            idNumber: '',
            superiorId: '0',
            openId: 'ohTn34vrzGtFvRnEEdhyOK6eKdPc',
            type: null,
            invitationCode: '1tSzznkuEGe',
            createTime: '2022-11-09 10:34:39',
            lastLoginTime: '2022-12-17 15:45:15',
            userHash: 'iaa1ypsdvk9sw5uk62lfr803c7s0406xdaqgpx75zk',
            updateTime: '2022-12-17 14:26:30',
            del: 0,
            collectionNum: 254,
            blindBoxNum: 0,
            fragmentNum: 0,
            unreadNum: 0,
            unpaid: 0,
            unpaidCollectionNum: 0,
            unpaidBlindBoxNum: 0,
            badges: null,
            pullCount: null,
            pullAuthenticationCount: null,
            transferCodeSwitch: 0,
            apiUrl: 'https://yjnft-api.test.yjkjmeta.com',
            h5ArUrl: 'https://wifi.pro.youzewang.com/ar_h5',
            photoNumberlimit: 1
        });
        uni.setStorageSync('baseUrl', 'https://yjnft-api.test.yjkjmeta.com');
    },
    methods: {}
};
</script>
<style>
@import './index.css';
</style>