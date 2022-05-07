<template>
  <div>
    <div>
      <q-dialog v-model="playing" position="bottom" persistent>
        <q-card style="width: 350px">
          <!-- <q-linear-progress :value="0.6" color="primary" /> -->

          <q-card-section class="row items-center no-wrap bg-primary">
            <div>
              <div class="text-weight-bold">{{ currentSong }}</div>
            </div>

            <q-space />

            <q-btn flat round icon="stop" v-close-popup @click="stopSound" />
          </q-card-section>
        </q-card>
      </q-dialog>
    </div>

    <div class="container">
      <div class="row q-mt-md">
        <div class="col-10">
          <q-input
            class="q-px-md"
            outlined
            v-model="searchText"
            label="Search Music"
            bg-color="primary"
            label-color="white"
          />
        </div>
        <div class="col-2">
          <q-btn
            round
            color="primary"
            size="md"
            icon="search"
            @click="fetchSongs"
            @keyup.enter="fetchSongs"
          />
        </div>
      </div>
    </div>

    <q-list bordered class="text-white">
      <q-item
        v-for="song in songs"
        :key="song.id"
        class="q-my-sm"
        clickable
        v-ripple
      >
        <q-item-section avatar>
          <q-avatar color="primary" text-color="white">
            <q-img
              :src="song.album.images[0].url"
              spinner-color="primary"
              spinner-size="82px"
            />
          </q-avatar>
        </q-item-section>

        <q-item-section @click="storeSongInfo(song.name, song.id)">
          <q-item-label class="song-name">{{ song.name }}</q-item-label>
          <q-item-label caption lines="1" class="text-white song-artist">
            <!-- list all the artists -->
            {{ song.artists.map((artist) => artist.name).join(", ") }}
          </q-item-label>
        </q-item-section>

        <q-item-section side v-if="song.preview_url">
          <q-icon
            name="play_arrow"
            color="green"
            @click="playSound(song.preview_url, song.name)"
          />
        </q-item-section>
      </q-item>
    </q-list>
  </div>
</template>

<script>
// import Player from "../components/Player.vue";
import { Howl, Howler } from "howler";

export default {
  name: "Index",
  components: {
    // Player,
  },
  data() {
    return {
      searchText: "",
      songs: [],
      currentSong: "",
      playing: false,
    };
  },
  methods: {
    async fetchSongs() {
      const response = await fetch(
        `http://n3rd-last-fm-api.glitch.me/getSongs?songQuery=${this.searchText}`
      );
      const songs = await response.json();
      this.songs = songs.tracks.items;
    },
    playSound(song, songName) {
      const sound = new Howl({
        src: [song],
        html5: true,
      });
      Howler.stop();
      sound.play();
      this.playing = true;
      this.currentSong = songName;
      sound.on("end", () => {
        this.stopSound();
        console.log("Finished!");
      });
    },
    stopSound() {
      Howler.stop();
      this.playing = false;
    },
    closePlayer() {
      this.playing = false;
    },
    storeSongInfo(songName, songId) {
      localStorage.setItem("app#2SongName", songName);
      localStorage.setItem("app#2SongId", songId);
    },
  },
};
</script>

<style lang="scss" scoped>
#q-app > div > label > div > div > div > div {
  color: #fff;
}
.song-name {
  font-size: 1.2rem;
  // format overflowing text
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.song-artist {
  font-size: 0.8rem;
}
</style>
