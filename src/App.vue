<template>
  <div class="player"
        @drop.prevent="onDrop"
        @dragover.stop.prevent
        @dragenter.stop.prevent
        @dragstart.stop.prevent>
    <div class="flex justify-center">
      <div class="album-art">
        <div v-if="selectedTrackIndex != -1 && tracks[selectedTrackIndex].album_art">
          <img :src="`data:${tracks[selectedTrackIndex].album_art.format};base64,${tracks[selectedTrackIndex].album_art.data.toString('base64')}`" />
        </div>
        <div v-else>
          Album Art
        </div>
      </div>
    </div>
    <div class="controls">
      <img src="icons/previous.svg" class="h-75 ma2" />
      <img src="icons/play-pause.svg" class="h-100 ma2" />
      <img src="icons/next.svg" class="h-75 ma2" />
      <img src="icons/repeat.svg" class="h-100 ma2" />
      <img src="icons/shuffle.svg" class="h-100 ma2" />
    </div>
    <!-- <div>
      Add Remove
oop    </div> -->
    <div style="width: 100%; height: 100%; background-color: antiquewhite; position: absolute;">
      <div v-for="(track, index) of tracks" :key="track.path" class="flex">
        <div v-if="track.title" @click="selectTrack(index)" class="w-100" :class="{ selected: index == selectedTrackIndex}">
          {{ track.title }} - {{ track.artist }}
        </div>
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
  type: string
}

export default {
  data() {
    return {
      tracks: [] as Track[],
      selectedTrackIndex: -1
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
    async processAddedTracks(pendingTracks: PendingTrack[]) {
      for(const pendingTrack of pendingTracks) {
        const { path, type } = pendingTrack
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
          type, path,
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
  },
  mounted() {

  }
}
</script>

<style>
.album-art {
  height: 400px;
  width: 400px;
  background-color: aquamarine;
}

.controls {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 30px;
  margin: 10px 0px 10px 0px;
}

.selected {
  background-color: aqua;
}
</style>
