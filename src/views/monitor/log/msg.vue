<template>
  <div class="container">
    <div id="console" :style="'height:'+ height" class="console">
      <div v-for="item in data" :key="item.time">
        <span>{{ item.name }}</span>
        <span style="color:#CD0066 ">{{ parseTime(item.timestamp)+' ' }}</span>
        <span style="color: #00CD00">{{ item.threadName+' ' }}</span>
        <span :style="'color:'+ getColor(item.level) ">
          {{ item.level+' ' }}
        </span>
        <span style="color: #DE00CC">{{ item.className+' ' }}</span>
        <span v-html="item.body"/>
      </div>
    </div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import SockJS from 'sockjs-client'
import Stomp from 'stompjs'
import { parseTime } from '@/utils/index'
export default {
  name: 'Msg',
  data() {
    return {
      height: document.documentElement.clientHeight - 95 + 'px;',
      data: [{ name: 'elAdmin-', timestamp: new Date(), threadName: 'system-prompt-message', level: 'INFO', className: 'me.zhengjie.AppRun' + ' :', body: 'Welcome, no log output~' }],
      // level
      INFO: '#0000ff', WARN: '#FFFF00', ERROR: '#FF0000', DEBUG: '#DEA000'
    }
  },
  computed: {
    ...mapGetters([
      'socketApi'
    ])
  },
  // 监听控制滚动条
  watch: {
    data: {
      handler(val, oldVal) {
        this.$nextTick(() => {
          var div = document.getElementById('console')
          div.scrollTop = div.scrollHeight
        })
      }
    }
  },
  mounted: function() {
    this.initWebSocket()
  },
  beforeDestroy: function() {
    // 页面离开时断开连接,清除定时器
    this.disconnect()
    window.clearInterval(this.timer)
  },
  methods: {
    parseTime,
    initWebSocket() {
      this.connection(this)
      // 断开重连机制,尝试发送消息,捕获异常发生时重连
      this.timer = window.setInterval(() => {
        try {
          this.stompClient.send('test')
        } catch (err) {
          console.log('断线了: ' + err)
          this.connection()
        }
      }, 5000)
    },
    connection(_this) {
      const socket = new SockJS(this.socketApi)// 连接服务端提供的通信接口，连接以后才可以订阅广播消息和个人消息
      // 获取STOMP子协议的客户端对象
      this.stompClient = Stomp.over(socket)
      // 定义客户端的认证信息,按需求配置
      var headers = {
        token: 'k1'
      }
      // 向服务器发起websocket连接
      this.stompClient.connect(headers, (frame) => {
        this.stompClient.subscribe('/topic/logMsg', (msg) => { // 订阅服务端提供的某个topic
          var content = JSON.parse(msg.body)
          content.name = 'elAdmin-'
          console.log(_this.data)
          this.data.push(content)
        })
      }, (err) => {
        // 连接发生错误时的处理函数
        console.log(err)
      })
    },
    // 断开连接
    disconnect() {
      if (this.stompClient != null) {
        this.stompClient.disconnect()
        window.clearInterval(this.timer)
      }
    },
    getColor(level) {
      if (level === 'INFO') {
        return this.INFO
      } else if (level === 'WARN') {
        return this.WARN
      } else if (level === 'DEBUG') {
        return this.DEBUG
      } else {
        return this.ERROR
      }
    }
  }
}
</script>

<style scoped>
  button,input,textarea{outline:0}.container .buttons .closes,.container .buttons .maximize,.container .buttons .minimize{padding:0;margin:0;margin-right:6px;width:12px;height:12px;border:1px solid transparent;border-radius:6px}.container{width:100%;margin:5px}.container .console{font-family:consolas;overflow-y:scroll;background:#494949;color:#f7f7f7;padding:10px;font-size:14px}
</style>
