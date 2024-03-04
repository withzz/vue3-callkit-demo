<script setup>
import { ref } from 'vue'
import NIM from "@yxim/nim-web-sdk/dist/SDK/NIM_Web_NIM"
import { NECall } from '@xkit-yx/call-kit'

const neCall = NECall.getInstance()
const appkey = ref('')
const accId = ref('')
const token = ref('')
const callee = ref('')
const message = ref('')
const flag = ref(1)

const login = () => {
    const im = NIM.getInstance({
        appKey: appkey.value, // im appkey
        token: token.value, // im token
        account: accId.value, // im account
        logLevel: 'debug',
        onconnect: () => {
            neCall.setup({
                nim: im, // im 实例用户需要先初始化
                appkey: appkey.value, // 应用的 appKey，G2 平台的 appKey
                debug: true, // 是否需要开启日志，默认开启
                rtcConfig: {
                    videoResolution: 'VIDEO_QUALITY.VIDEO_QUALITY_720p',
                    videoFrameRate: 'VIDEO_FRAME_RATE.CHAT_VIDEO_FRAME_RATE_NORMAL',
                    audioQuality: 'speech_low_quality',
                }, // rtcConfig 用来设置 RTC 可选。相关参数参照 可以参考[G2官方文档](https://doc.yunxin.163.com/docs/jcyOTA0ODM/zUxMDQzOTM?platformId=50082)
            })
            // 来电监听
            neCall.on('onReceiveInvited', (value) => {
                // 用户可以触发视图，比如弹起邀请页面
                console.log(value)
                message.value = `收到 ${value.callerAccId} 呼叫，可以点击接听按钮了`

            })
            // 通话结束的回调。通话被取消、拒绝、通话中挂断，即所有导致通话结束的行为都会触发此回调
            neCall.on('onCallEnd', (value) => {
                console.log(value)
                message.value = ''
            })
            // 话单回调
            neCall.on('onRecordSend', (value) => {
                console.log(value)
            })
            // 主被叫加入通道的回调，通话建立后触发
            neCall.on('onCallConnected', (value) => {
                console.log(value)
            })
        },
        ondisconnect: () => {
            neCall?.destroy()
        },
    })
}

const call = () => {
    // 发起呼叫
    const localView = document.getElementById('NE_local')
    const remoteView = document.getElementById('NE_remote')
    neCall.setLocalView(localView)
    neCall.setRemoteView(remoteView, callee.value)
    neCall.setTimeout(30)//设置呼叫超时取消时间，单位：秒
    neCall.call({
        accId: callee.value, // 被叫 im 的 id
        callType: '2', //呼叫类型，1表示语音通话，2表示视频通话 
    })
}

const maxRender = () => {
    const localView = document.getElementById('NE_local')
    localView.style.height = '640px'
    localView.style.width = '720px'
    neCall.rtcController.localStream.setLocalRenderMode({
        width: 720,
        height: 640,
        cut: true
    })

    const remoteView = document.getElementById('NE_remote')
    remoteView.style.height = '480px'
    remoteView.style.width = '640px'
    neCall.rtcController.remoteStreams[0].setRemoteRenderMode({
        width: 640,
        height: 480,
        cut: true
    })
}

const miniRender = () => {
    const localView = document.getElementById('NE_local')

    localView.style.height = '480px'
    localView.style.width = '640px'
    neCall.rtcController.localStream.setLocalRenderMode({
        width: 640,
        height: 480,
        cut: true
    })

    const remoteView = document.getElementById('NE_remote')

    remoteView.style.height = '640px'
    remoteView.style.width = '720px'
    neCall.rtcController.remoteStreams[0].setRemoteRenderMode({
        width: 720,
        height: 640,
        cut: true
    })
}


const switchView = () => {
    const localView = document.getElementById('NE_local')
    const remoteView = document.getElementById('NE_remote')
    neCall.rtcController.localStream.stop('video')
    neCall.rtcController.remoteStreams[0].stop('video')
    if (flag.value) {
        neCall.rtcController.localStream.play(remoteView)
        neCall.rtcController.localStream.setLocalRenderMode({
            width: 640,
            height: 480,
            cut: true
        })
        neCall.rtcController.remoteStreams[0].play(localView)
        neCall.rtcController.remoteStreams[0].setRemoteRenderMode({
            width: 720,
            height: 640,
            cut: true
        })
        flag.value = 0
    } else {
        neCall.rtcController.localStream.play(localView)
        neCall.rtcController.localStream.setLocalRenderMode({
            width: 720,
            height: 640,
            cut: true
        })
        neCall.rtcController.remoteStreams[0].play(remoteView)
        neCall.rtcController.remoteStreams[0].setRemoteRenderMode({
            width: 640,
            height: 480,
            cut: true
        })
        flag.value = 1
    }
}

const accept = () => {
    // 接受邀请
    const localView = document.getElementById('NE_local')
    const remoteView = document.getElementById('NE_remote')
    neCall.setLocalView(localView)
    neCall.setRemoteView(remoteView, callee.value)
    // 接受邀请，需要先设置视图
    neCall.accept()
    message.value = ''
}

const hangup = () => {
    // 挂断
    neCall.hangup()
    message.value = ''
}
</script>

<template>
    <div>
        <p style="color: red;">{{ message }}</p>
        呼叫组件简易demo，使用步骤如下：
        1、输入appkey等账号信息
        2、后点击登录
        3、呼叫对方或者收到对方呼叫之后点击接听
        <div>
            <button @click="login">登录</button>
            <button @click="call">呼叫</button>
            <button @click="accept">接听</button>
            <button @click="hangup">挂断</button>
            <button @click="miniRender">缩小视图</button>
            <button @click="maxRender">放大试图</button>
            <button @click="switchView">切换视图</button>
        </div>
        <div>
            appkey:
            <input type="text" v-model="appkey" />
            accId:
            <input type="text" v-model="accId" />
            token:
            <input type="text" v-model="token" />
            callee:
            <input type="text" v-model="callee" />
        </div>
        <b>本地视图</b>
        <div id="NE_local" style="width:640px; height:480px; background-color:red;">
        </div>
        <b>远端视图</b>
        <div id="NE_remote" style="width:320px; height:240px; background-color:green;"></div>
    </div>
</template>