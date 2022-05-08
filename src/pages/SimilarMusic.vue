<template>
  <div>
    <!-- add an header -->

    <q-toolbar class="bg-primary">
      <q-toolbar-title>
        <q-icon name="arrow_back" />
        <span>Similar Music to {{ baseSongName }}</span>
      </q-toolbar-title>
    </q-toolbar>

    <div class="preloader" v-if="preloader">
      <div class="text">Getting Similar songs to {{ baseSongName }}</div>
      <q-spinner-audio color="primary" size="2em" />
    </div>
  </div>
</template>
<script>
import { ref } from "vue";
export default {
  name: "similarMusic",
  data() {
    return {
      baseSongId: "",
      baseSongName: "",
      similarSongs: [],
      slide: "style",
      preloader: true,
    };
  },
  methods: {
    async fetchSimilarSongs() {
      const response = await fetch(
        `https://n3rd-last-fm-api.glitch.me/getSimilarTracks?seedTrack=${this.baseSongId}`
      );
      this.similarSongs = await response.json();
    },
  },
  mounted() {
    this.baseSongId = localStorage.getItem("app#2SongId");
    this.baseSongName = localStorage.getItem("app#2SongName");
    this.fetchSimilarSongs();
    setTimeout(() => {
      this.preloader = false;
    }, 3000);
  },
};
</script>

<style lang="scss" scoped>
.preloader {
  // set fullscreen
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: #000;
  z-index: 9999;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
