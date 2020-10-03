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
    <v-layout row wrap>
      <template v-for="theirStream in theirStreams">
        <v-flex
          v-if="theirStream && theirStream.active"
          :key="theirStream.id"
          xs12
          sm12
          md6
        >
          <v-card class="mx-2 my-2">
            <video
              :ref="`theirVideo${theirStream.id}`"
              class="video"
              autoplay
              playsinline
            />
          </v-card>
        </v-flex>
      </template>
      <v-flex v-if="localStream" xs12 sm12 md6>
        <v-card class="mx-2 my-2">
          <video
            ref="myVideo"
            class="video"
            muted="true"
            autoplay
            playsinline
          />
        </v-card>
      </v-flex>
    </v-layout>
  </div>
</template>

<script></script>

<style lang="scss" scoped>
.video {
  width: 100%;
  height: 100%;
}
</style>
