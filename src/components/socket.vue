<template>
    <div>
    </div>
</template>

<script>
    import Vue from 'vue'
    import {bindClientId} from "../api/common";

    export default {
        data() {
            return {
                websocket: null,
            }
        },
        methods: {
            threadPoxi() {  // 实际调用的方法
                //参数
                const agentData = {uid: this.$store.state.userInfo.id};
                //若是ws开启状态
                if (this.websocket.readyState === this.websocket.OPEN) {
                    this.websocketSend(agentData)
                }
                // 若是 正在开启状态，则等待300毫秒
                else if (this.websocket.readyState === this.websocket.CONNECTING) {
                    let that = this;//保存当前对象this
                    setTimeout(function () {
                        that.websocketSend(agentData)
                    }, 300);
                }
                // 若未开启 ，则等待500毫秒
                else {
                    this.initWebSocket();
                    let that = this;//保存当前对象this
                    setTimeout(function () {
                        that.websocketSend(agentData)
                    }, 500);
                }
            },
            initWebSocket() { //初始化weosocket
                //ws地址
                const wsuri = "ws://192.168.1.106:2345";
                this.websocket = new WebSocket(wsuri);
                this.websocket.onmessage = this.websocketOnMessage;
                this.websocket.onclose = this.websocketClose;
                Vue.prototype.$websocket = this.websocket;

            },
            websocketOnMessage(e) { //数据接收
                const data = eval("(" + e.data + ")");
                this.$store.commit('catchSocketAction', data);
                if (data.action === 'connect') {
                    data.data.client_id && window.localStorage.setItem('client_id', data.data.client_id);
                    bindClientId(data.data.client_id);
                }
                console.log("收到消息：");
                console.log(data);
            },
            websocketSend(agentData) {//数据发送
                console.log("发送消息：");
                console.log(agentData);
                this.websocket.send(agentData);
            },
            websocketClose(e) {  //关闭
                console.log("connection closed (" + e.code + ")");
            }
        },
        created() {
            this.initWebSocket()
        }
    }
</script>
