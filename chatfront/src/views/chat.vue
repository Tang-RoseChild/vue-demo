<template>
  <div>
    <v-layout row>
    <v-flex xs8 offset-xs4 sm4 offset-sm8>
      <v-btn icon id="chatmsg-side-btn" v-show="sideBtnVisible" @click.native="sideBtnClicked">
        <v-icon>chat</v-icon>
      </v-btn>
      <v-card class="chatmsg" v-show="msgBoxVisible">
        <v-layout row class="chatmsg-header">
            <v-btn icon @click.native.stop="backBtnClicked">
              <v-icon class="white--text" >arrow_back</v-icon>
            </v-btn>
            <span class="white--text" v-html="header"></span>
        </v-layout>
        <div class="chatmsg-content">
          <v-divider style="margin-bottom:20px;"></v-divider>
          <template v-for="(msg, idx) in messages" >
          <v-layout row  v-bind:key="idx" class="chatmsg-other" v-if="msg.msgType === 'other'">
              <img src="/static/avatar.jpeg" />
              <v-card>
                <v-card-text v-html="replaceBreakLine(msg.content)">
                </v-card-text>
              </v-card>
          </v-layout>
          <v-layout row justify-end class="chatmsg-me" v-bind:key="idx" v-else-if="msg.msgType === 'me'">
              <v-card>
                <v-card-text v-html="replaceBreakLine(msg.content)">
                </v-card-text>
              </v-card>
              <img src="/static/avatar.jpeg" />
          </v-layout>
          </template>
        </div>
        <v-divider></v-divider>
        <v-layout row class="chatmsg-footer">
           <v-text-field multi-line rows="1" placeholder="说点什么" v-model="sendMsgContent"></v-text-field>
          <v-btn icon @click="sendMsg">
            <v-icon >send</v-icon>
          </v-btn>
        </v-layout>
      </v-card>
    </v-flex>
    </v-layout>
  </div>
</template>

<script>
import uid from '../utils/utils.js'
export default {
  data () {
    return {
      conn: '',
      sendMsgContent: '',
      msgBoxVisible: true,
      sideBtnVisible: false,
      header: 'this is a text demo',
      messages: [],
      contentType: 'me'
    }
  },
  methods: {
    replaceBreakLine (src) {
      return src.replace(/\n/g, '<br>')
    },
    backBtnClicked () {
      this.msgBoxVisible = !this.msgBoxVisible
      this.sideBtnVisible = !this.sideBtnVisible
    },
    sideBtnClicked () {
      console.log('side btn clicked')
      this.msgBoxVisible = !this.msgBoxVisible
      this.sideBtnVisible = !this.sideBtnVisible
    },
    sendMsg () {
      let data = {type: 1, payload: {chatMessage: {from: uid, content: this.sendMsgContent}}}
      this.sendMsgContent = ''
      this.conn.send(JSON.stringify(data))
    },
    messageHandler (data) {
      let self = this
      let msgs = JSON.parse(data)
      msgs.forEach(function (msg) {
        if (msg.type === 1) {
          if (msg.payload.chatMessage.from === uid) {
            msg.payload.chatMessage.msgType = 'me'
          } else {
            msg.payload.chatMessage.msgType = 'other'
          }

          self.messages.push(msg.payload.chatMessage)
          setTimeout(function () {
            let el = self.$el.querySelector('.chatmsg-content')
            el.scrollTop = el.scrollHeight - el.offsetHeight
          })
        }
      }, this)
    }
  },
  created () {
    let self = this
    console.log('window websocket', window['WebSocket'])
    if (window['WebSocket']) {
      this.conn = new WebSocket('ws://' + document.location.host + '/ws')
      // this.conn = new WebSocket('ws://127.0.0.1:9998/ws')
      this.conn.onclose = function (evt) {
        alert('connection closed')
      }
      this.conn.onmessage = function (evt) {
        self.messageHandler(evt.data)
      }
    } else {
      alert(`browser doesn't support websocket`)
    }
  }
}
</script>

<style>
.chatmsg-content {
  height: 300px !important;
  overflow-y: auto;
  overflow-x: hidden;
}

img {
  height: 30px;
  width: 30px;
  margin: 5px;
  padding: 1px;
  float: right;
}

.chatmsg {
  overflow: hidden;
}

.chatmsg .card {
  margin: 10px !important;
}

.chatmsg-header {
  background-color:  #807e7e;
  align-items: center;
}
.chatmsg-header span {
  font-size: 1.5em;

}

.chatmsg-other .chatmsg-me {
  overflow: hidden;
  margin: 5px !important;
}

.chatmsg-other .card__text {
  padding: 10px !important;
}

.chatmsg-me .card__text {
  padding: 10px !important;
  /* background: #7cfc00; */
  background:#77ea07;
}


.chatmsg-footer {
  padding: 10px;
  margin: 5px;
}
.chatmsg-footer .input-group {
  margin: 0px !important;
}

.chatmsg-footer .input-group__details {
  height: 1px !important;
  min-height: 1px !important;
}

.chatmsg-footer .input-group__details .input-group__messages {
  min-height: 0px !important;
  height: 0px !important;
}

#chatmsg-side-btn {
  position: fixed !important;
  top: 45%;
  right: 0px;
}
textarea {
  overflow: hidden;
  resize: none !important;
}
</style>
