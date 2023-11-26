<template>
  <div>
    <div v-if="selectedTrackIndex != -1 && tracks[selectedTrackIndex].album_art">
      <img :src="`data:${tracks[selectedTrackIndex].album_art.format};base64,${tracks[selectedTrackIndex].album_art.data.toString('base64')}`" />
    </div>
    <div v-else>
      Album Art
    </div>
  </div>
  <div @drop.prevent="onDrop" @dragenter.prevent @dragover.prevent style="width: 100%; height: 100%; background-color: antiquewhite; position: absolute;">
    <div v-for="(track, index) of tracks" :key="track.path" class="flex">
      <div v-if="track.title" @click="selectTrack(index)" class="w-100" :class="{ selected: index == selectedTrackIndex}">
        {{ track.title }} - {{ track.artist }}
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { parseFile } from 'music-metadata'

export default {
  data() {
    return {
      tracks: [{
        path: '',
        type: '',
        name: '',
        title: '',
        artist: '',
        album: '',
        album_art: {
          data: undefined,
          format: ''
        }
      }],
      selectedTrackIndex: -1
    }
  },
  methods: {
    async onDrop(e : any) {
      if(e.dataTransfer?.files) {
        for(const file of e.dataTransfer.files) {
          const { name, type, path } = file
          const metadata = await parseFile(path)
          console.log(path)
          const { title, artist, album, picture } = metadata.common
          // console.log(metadata)
          // title, artist, album, album art
          this.tracks.push({
            name, type, path,
            title: title || 'Unknown',
            artist: artist || 'Unknown Artist',
            album: album || 'Unknown Album',
            album_art: {
              format: picture?.[0]?.format,
              data: picture?.[0]?.data,
            }
          })
        }
      }
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
.selected {
  background-color: aqua;
}
</style>
