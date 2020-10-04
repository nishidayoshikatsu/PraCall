<template>
  <div class="video-call">
    <v-layout row wrap>
      <v-flex xs12 sm12 md12>
        <h1 class="titile">ビデオ通話(多人数,SFU)</h1>
      </v-flex>
    </v-layout>
    <v-card class="mx-2 my-2">
      <v-form>
        <v-container>
          <v-layout row wrap>
            <v-flex xs12 sm12 md12>
              <v-text-field v-model="peerId" label="Peer ID" :readonly="true" />
            </v-flex>
            <v-flex xs12 sm12 md6>
              <v-select
                v-model="selectedAudio"
                :items="audios"
                label="オーディオ"
                @change="changeAudioVideo"
              />
            </v-flex>
            <v-flex xs12 sm12 md6>
              <v-select
                v-model="selectedVideo"
                :items="videos"
                label="ビデオ"
                @change="changeAudioVideo"
              />
            </v-flex>
          </v-layout>
        </v-container>
      </v-form>
    </v-card>
    <v-card class="mx-2 my-2">
      <v-form ref="form" @submit.prevent="makeRoom">
        <v-container>
          <v-layout row wrap>
            <v-flex xs12 sm12 md12>
              接続先のルーム名を入力して接続してください。
            </v-flex>
            <v-flex xs12 sm12 md12>
              <v-text-field v-model="roomName" label="ルーム名" />
            </v-flex>
          </v-layout>
        </v-container>
        <v-card-actions>
          <v-spacer />
          <v-btn color="primary" type="submit"> 接続 </v-btn>
          <v-btn @click="endRoom"> 切断 </v-btn>
        </v-card-actions>
      </v-form>
    </v-card>
    <v-layout row wrap class="video-layout">
      <template v-for="theirStream in theirStreams">
        <v-flex
          v-if="theirStream && theirStream.active"
          :key="theirStream.id"
          xs12
          sm12
          md6
        >
          <v-card class="mx-2 my-2 video-card">
            <video
              :ref="`theirVideo${theirStream.id}`"
              class="video"
              autoplay
              playsinline
            />
            <div class="info">
              <div>
                <h4>Templature: 24℃</h4>
              </div>
            </div>
          </v-card>
        </v-flex>
      </template>
      <v-flex v-if="localStream" xs12 sm12 md6>
        <v-card class="mx-2 my-2 video-card">
          <video
            ref="myVideo"
            class="video"
            muted="true"
            autoplay
            playsinline
          />
          <div class="info">
            <div>
              <h4>Templature: 24℃</h4>
            </div>
          </div>
        </v-card>
      </v-flex>
    </v-layout>
  </div>
</template>

<script>
import Peer from 'skyway-js'

export default {
  data() {
    return {
      peer: null,
      selectedAudio: '',
      selectedVideo: '',
      audios: [],
      videos: [],
      localStream: null,
      theirStreams: [],
      peerId: '',
      roomName: '',
      room: null,
    }
  },
  mounted() {
    // オーディオ・ビデオデバイスの準備
    this.prepareAudioVideoDevice()
    // ローカルカメラに接続
    this.connectLocalCamera()
    // Peerオブジェクトの作成
    this.peer = new Peer({ key: process.env.API_KEY, debug: 3 })
    // 接続成功・失敗・切断時のイベント設定
    this.peer.on('open', () => {
      console.log(`peer event open: peer.id=${this.peer.id}`)
      this.peerId = this.peer.id
    })
    this.peer.on('error', (err) => {
      console.error(`peer event error:`, err)
    })
    this.peer.on('disconnected', () => {
      console.log(`peer event disconnected`)
    })
  },
  methods: {
    // オーディオ・ビデオデバイスを準備
    prepareAudioVideoDevice() {
      navigator.mediaDevices.enumerateDevices().then((deviceInfos) => {
        const audios = [{ text: '指定なし', value: '' }]
        const videos = [{ text: '指定なし', value: '' }]
        for (let i = 0; i !== deviceInfos.length; ++i) {
          const deviceInfo = deviceInfos[i]
          if (deviceInfo.kind === 'audioinput') {
            audios.push({
              text: deviceInfo.label || `Microphone ${this.audios.length + 1}`,
              value: deviceInfo.deviceId,
            })
          } else if (deviceInfo.kind === 'videoinput') {
            videos.push({
              text: deviceInfo.label || `Camera  ${this.videos.length - 1}`,
              value: deviceInfo.deviceId,
            })
          }
        }
        this.audios = audios
        this.videos = videos
      })
    },
    // オーディオ・ビデオの変更
    changeAudioVideo() {
      this.$nextTick(() => {
        this.connectLocalCamera()
      })
    },
    // ローカルカメラに接続
    connectLocalCamera() {
      const constraints = {
        audio:
          this.selectedAudio !== ''
            ? { deviceId: { exact: this.selectedAudio } }
            : true,
        video:
          this.selectedVideo !== ''
            ? { deviceId: { exact: this.selectedVideo } }
            : true,
      }
      navigator.mediaDevices
        .getUserMedia(constraints)
        .then((stream) => {
          this.prepareAudioVideoDevice()
          this.localStream = stream
          this.$nextTick(() => {
            this.$refs.myVideo.srcObject = stream
            if (this.room) {
              this.room.replaceStream(stream)
            }
          })
        })
        .catch((err) => {
          console.error('mediaDevice.getUserMedia() error:', err)
        })
    },
    // 発信
    makeRoom() {
      if (this.roomName === '') {
        return
      }
      const room = this.peer.joinRoom(`sfu_video_${this.roomName}`, {
        mode: 'sfu',
        stream: this.localStream,
      })
      this.connectRoom(room)
    },
    // 接続
    connectRoom(room) {
      this.endRoom()
      this.room = room
      room.on('stream', (stream) => {
        this.theirStreams.push(stream)
        this.$nextTick(() => {
          this.$refs[`theirVideo${stream.id}`][0].srcObject = stream
          this.$refs[`theirVideo${stream.id}`][0].play()
        })
      })
      room.on('removeStream', (stream) => {
        console.log(stream)
      })
    },
    // 切断
    endRoom() {
      if (this.room) {
        this.room.close()
        this.room = null
      }
    },
  },
}
</script>

<style lang="scss" scoped>
.video-layout {
  margin: 30px;
}
.video-card {
  width: 35vw;
  height: 70vh;
  margin: 0 30px;
  .video {
    width: 100%;
    height: 35vh;
  }
  .info {
    margin: 20px 50px;
    div {
      margin: 15px;
    }
  }
}
</style>
