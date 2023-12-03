<template>
  <div class="player"
        @drop.prevent="onDrop"
        @dragover.stop.prevent
        @dragenter.stop.prevent
        @dragstart.stop.prevent>
    <div class="player-background" :style="{ backgroundImage: `url(${albumArtData})`}" />
    <div class="flex justify-center">
      <div class="album-art-container">
        <div v-if="selectedTrackIndex != -1 && tracks[selectedTrackIndex].album_art?.data">
          <img :src="albumArtData" class="album-art" />
        </div>
        <div v-else>
          <div class="album-art aa-placeholder">
            Album Art
          </div>
        </div>
      </div>
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
      <div v-for="(track, index) of tracks" :key="track.path" @click="selectTrack(index)" class="w-100 track" :class="{ selected: index == selectedTrackIndex}">
        {{ track.title }} - {{ track.artist }} ({{ track.ext }})
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { parseFile } from 'music-metadata'

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
      selectedTrackIndex: -1
    }
  },
  computed: {
    albumArtData() {
      if(this.tracks && this.tracks.length > 0 && this.selectedTrackIndex > 0) {
        return `data:${this.tracks[this.selectedTrackIndex].album_art.format};base64,${this.tracks[this.selectedTrackIndex].album_art.data.toString('base64')}`
      }
      return ''
    },
    albumArtDataWithUrl() {
      return `url('${this.albumArtData}'')`
    }
  },
  methods: {
    onDrop(e : any) {
      console.log('dropped')
      console.log(e.dataTransfer)
      console.log(e.dataTransfer?.files?.length)
      const pendingTracks : PendingTrack[] = []
      for(const file of e.dataTransfer.files) {
        console.log(file)
        const { type, path } = file
        pendingTracks.push({ type, path })
      }
      this.processAddedTracks(pendingTracks)
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
        console.log(path)
        const { title, artist, album, picture } = metadata.common
        const { format: aa_format, data: aa_data } = picture?.[0] as any
        // console.log(metadata)
        // title, artist, album, album art
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
          && this.tracks.length > 0) this.selectedTrackIndex = 0
    },
    selectTrack(index: number) {
      this.selectedTrackIndex = index
    }
  }
}
</script>

<style>
body {
  margin: 20px;
  overflow: hidden;
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
  margin: 50px;
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
  box-shadow: -10px -10px 50px #333;
}

.control {
  height: 100%;
  margin-right: 30px;
}

.big-control {
  height: 150%;
  margin-right: 30px;
}

.controls {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 30px;
  margin: 0px 0px 20px 0px;
  filter: invert(100%) brightness(1000%);
}

.track-list {
  color: white;
  background-blend-mode: hue;
  border-radius: 20px;
  padding: 20px;
}

.track {
  padding: 3px;
}

.selected {
  background-color: #555;
}
</style>
