<template>
  <div @drop.prevent="onDrop" @dragenter.prevent @dragover.prevent style="width: 100%; height: 100%; background-color: antiquewhite; position: absolute;">
    Hey Dude
  </div>
</template>

<script lang="ts">
// import { contextBridge, ipcRenderer } from 'electron'

export default {
  data() {
    return {
      tracks: []
    }
  },
  methods: {
    onDrop(e : any) {
      if(e.dataTransfer?.files) {
        for(const file of e.dataTransfer.files) {
          console.log(file)
        }
      }
    }
  },
  mounted() {

  }
}
</script>

<style>

</style>
