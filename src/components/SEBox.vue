<template>
  <div class="se-box">
    <v-card>
      <v-layout class="se-box-title">
        <v-flex xs1>
          <v-icon @click="removeSE">clear</v-icon>
        </v-flex>
        <v-flex xs11>
          <v-card-title class="font-weight-bold">{{name}}</v-card-title>
        </v-flex>
      </v-layout>
      <div>メモ</div>
      <v-layout>
        <v-flex xs2>
          <v-btn @click="togglePlaying" small fab color="grey darken-2">
            <v-icon>play_arrow</v-icon>
          </v-btn>
        </v-flex>
        <v-flex xs10>
          <v-slider append-icon="volume_up" prepend-icon="volume_down" v-model="volume" @input="applyVolume"></v-slider>
        </v-flex>
      </v-layout>
      <v-progress-linear v-if="!isPlaying" :value="0"></v-progress-linear>
      <v-progress-linear v-else :indeterminate="true"></v-progress-linear>
    </v-card>
  </div>
</template>

<script>
const path = require('path')
const electron = require('electron')
const fs = electron.remote.require('fs')
export default {
  name: 'SEBox',
  props: {
    // Now, SE is indentified by file path.
    filepath: String
  },
  data () {
    return {
      context: new AudioContext(),
      source: null,
      gainNode: null,
      isPlaying: false,
      volume: 50
    }
  },
  created () {
    this.initializeContext()
  },
  methods: {
    removeSE () {
      this.$emit('remove-sound', this.filepath)
    },
    togglePlaying () {
      if (!this.source) {
        return
      }
      this.$emit('play-sound', this.filepath)
      this.playSound()
      this.isPlaying = true
    },
    playSound () {
      this.initializeContext()
      this.context.resume().then()
      this.source.start(0)
      this.isPlaying = true
      setTimeout(() => {
        this.isPlaying = false
      }, 2000)
    },
    initializeContext () {
      this.context.suspend().then()
      fs.readFile(this.filepath, (error, data) => {
        if (error) {
          console.error(error)
        }
        const arraySoundBuffer = data.buffer.slice(data.byteOffset, data.byteOffset + data.byteLength)
        this.context.decodeAudioData(arraySoundBuffer, (decodedSoundBuffer) => {
          this.source = this.createSource(this.context, decodedSoundBuffer)
          this.gainNode = this.createGainNode(this.context, this.volume)
          this.connectAll(this.context, this.source, this.gainNode)
        }).then()
      })
    },
    applyVolume () {
      if (!this.gainNode) {
        return
      }
      this.gainNode.gain.value = this.toRealVolume(this.volume)
    },
    createSource (context, buffer) {
      const source = context.createBufferSource()
      source.buffer = buffer
      source.loop = false
      return source
    },
    createGainNode (context, volume) {
      const gainNode = context.createGain()
      gainNode.gain.value = this.toRealVolume(volume)
      return gainNode
    },
    connectAll (context, source, gainNode) {
      source.connect(gainNode)
      gainNode.connect(context.destination)
    },
    toRealVolume (percentValue) {
      return percentValue * 0.01
    }
  },
  computed: {
    name () {
      return path.basename(this.filepath, '.mp3')
    },
    endTime () {
      if (!this.source) {
        return 0
      }
      return this.source.buffer.duration
    }
  }
}
</script>

<style>
.v-slider {
  margin-left: 8px;
  margin-right: 8px;
}
.v-input--slider {
  margin-left: 8px;
  margin-right: 8px;
}
.v-input--slider .v-input__slot {
  margin-bottom: 0;
}
.v-progress-linear {
  margin: 0;
}
.se-box-title {
  background-color:darkolivegreen;
  word-wrap: break-word;
}
</style>
