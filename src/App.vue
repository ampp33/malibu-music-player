<template>
  <div class="player"
        @drop.prevent="onDrop"
        @dragover.stop.prevent
        @dragenter.stop.prevent
        @dragstart.stop.prevent
        draggable="true"
        @keypress="trackListKeyHandler">
    <div v-if="playingTrackIndex != -1" class="player-background" :style="{ backgroundImage: `url(${albumArtData})`}" />
    <div v-else class="player-background" style="background-color: darkgrey;" />
    <div class="flex justify-center">
      <div class="album-art-container">
        <div v-if="playingTrackIndex != -1 && tracks[playingTrackIndex].album_art?.data">
          <img :src="albumArtData" class="album-art" />
        </div>
        <div v-else>
          <div class="album-art aa-placeholder">
            Album Art
          </div>
        </div>
      </div>
    </div>
    <div v-if="playingTrackIndex != -1" class="track-info">
      <div class="song">{{ tracks[playingTrackIndex].title }}</div>
      <div class="album">{{ tracks[playingTrackIndex].album }}</div>
      <div class="artist">{{ tracks[playingTrackIndex].artist }}</div>
    </div>
    <div class="controls">
      <img src="icons/repeat.svg" class="control" />
      <img src="icons/previous.svg" class="control" />
      <img src="icons/play-pause.svg" class="big-control" />
      <img src="icons/next.svg" class="control" />
      <img src="icons/shuffle.svg" class="control" />
    </div>
    <!-- <div>
      Add Remove
oop    </div> -->
    <div class="track-list">
      <div v-if="tracks && tracks.length > 0" v-for="(track, index) of tracks" :key="track.path" @click="selectTrack(index)" @dblclick="playTrack(index)" class="w-100 track" :class="{ selected: index == selectedTrackIndex}">
        {{ track.title }} - {{ track.artist }} ({{ track.ext }})
      </div>
      <div v-else style="color: black">
        No Tracks
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { parseFile } from 'music-metadata'
const Vibrant = require('node-vibrant')

type PendingTrack = {
  path: string,
  type: string
}

type Track = {
  title: string,
  artist: string,
  album: string,
  album_art: {
    format: string,
    data: Buffer
  },
  path: string,
  type: string,
  ext: string
}

export default {
  data() {
    return {
      tracks: [] as Track[],
      selectedTrackIndex: -1,
      playingTrackIndex: -1,
      domanantColor: '#333333'
    }
  },
  computed: {
    albumArtData() {
      if(this.tracks && this.tracks.length > 0 && this.playingTrackIndex >= 0) {
        return `data:${this.tracks[this.playingTrackIndex].album_art.format};base64,${this.tracks[this.playingTrackIndex].album_art.data.toString('base64')}`
      }
      return ''
    },
    albumArtDataWithUrl() {
      return `url('${this.albumArtData}'')`
    }
  },
  methods: {
    onDrop(e : any) {
      const pendingTracks : PendingTrack[] = []
      for(const file of e.dataTransfer.files) {
        const { type, path } = file
        pendingTracks.push({ type, path })
      }
      this.processAddedTracks(pendingTracks)
    },
    trackListKeyHandler(e : KeyboardEvent) {
      console.log(e)
    },
    getFileExt(path: string) {
      const periodIndex = path.lastIndexOf('.')
      if(periodIndex > 0) return path.substring(periodIndex + 1)
      return ''
    },
    async processAddedTracks(pendingTracks: PendingTrack[]) {
      for(const pendingTrack of pendingTracks) {
        const { path, type } = pendingTrack
        const ext = this.getFileExt(path)
        let metadata = {} as any
        try {
          metadata = await parseFile(path)
        } catch (e) {
          continue
        }
        const { title, artist, album, picture } = metadata.common
        const { format: aa_format, data: aa_data } = picture?.[0] as any
        this.tracks.push({
          type, path, ext,
          title: title || 'Unknown',
          artist: artist || 'Unknown Artist',
          album: album || 'Unknown Album',
          album_art: {
            format: aa_format,
            data: aa_data,
          }
        })
      }
      if(this.selectedTrackIndex == -1
          && this.tracks.length > 0) this.selectTrack(0)
    },
    async setPlayerColors() {
      if(this.tracks && this.tracks.length > 0 && this.playingTrackIndex >= 0) {
        const aaData = this.tracks[this.playingTrackIndex].album_art.data.toString('base64')
        const buffer = Buffer.from(aaData, 'base64');

        const palette = await Vibrant.from(buffer).getPalette()
        const dominantColor = palette.Vibrant.getHex();
        console.log('determined dom color: ' + dominantColor)
        this.domanantColor = dominantColor
      } else {
        console.log('no dom color')
        this.domanantColor = '#333333'
      }
    },
    selectTrack(index: number) {
      this.selectedTrackIndex = index
    },
    playTrack(index: number) {
      this.playingTrackIndex = index
      this.setPlayerColors()
    }
  }
}
</script>

<style>
body {
  margin: 20px;
  overflow: hidden;
  -webkit-app-region: drag;
}

.player {
  /* filter: grayscale(1); */
}

.player-background {
  position: absolute;
  left: -50%;
  top: -50%;
  height: 200%;
  width: 200%;
  z-index: -1;

  background-repeat: no-repeat;
  background-size: cover;
  filter: blur(50px);
  -webkit-filter: blur(50px) brightness(50%);
}

.album-art-container {
  margin: 50px 50px 0px 50px;
}

.aa-placeholder {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 500px;
  width: 500px;
  background-color: #555;
}

.album-art {
  box-shadow: 0px 0px 120px -40px v-bind(domanantColor);
  max-height: 600px;
  margin-bottom: 20px;
}

.track-info {
  display: flex;
  flex-direction: column;
  align-items: center;
  color: white;
  margin-bottom: 25px;
}

.track-info .song {
  font-size: 25px;
  line-height: 30px;
  font-weight: 400;
}

.track-info .album {
  font-size: 20px;
  line-height: 25px;
  font-weight: 300;
}

.track-info .artist {
  font-size: 20px;
  line-height: 25px;
  font-weight: 200;
}

.control {
  height: 50%;
  margin-right: 30px;
}

.big-control {
  height: 100%;
  margin-right: 30px;
}

.controls {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 50px;
  margin: 0px 0px 20px 0px;
  filter: invert(100%) brightness(1000%);
}

.track-list {
  color: white;
  padding: 20px;

  -webkit-app-region: no-drag;
}

.track {
  padding: 3px;
}

.selected {
  background-color: black;
  color: white;
}
</style>
