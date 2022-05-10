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

    <div class="container fixed-top bg-black q-py-md search-container z-top">
      <q-input
        class="q-px-md text-center text-white search-music"
        outlined
        v-model="searchText"
        placeholder="Search Music"
        label-color="white"
        color="white"
        dark
        filled
        :input-style="{ textTransform: 'capitalize' }"
        @keyup.enter="fetchSongs"
      >
        <template v-slot:append>
          <q-icon name="search" />
        </template>
      </q-input>

      <div class="q-px-md">
        <q-btn
          color="primary"
          label="Search"
          class="lt-md full-width"
          @click="fetchSongs"
        />
      </div>
    </div>

    <div class="loader fixed-center" v-if="loading">
      <div>
        <q-spinner color="primary" size="63px" />
      </div>
    </div>

    <div class="illustration" v-if="illustration">
      <img srcset="undraw_compose_music_ovo2.svg 4x" alt="illustration" />
      <div class="text-center">
        Get music recommendations for your favorite songs at <b>sonicology</b>
      </div>
    </div>

    <div class="fetch-error" v-if="fetchError">
      <q-icon name="error" size="xl" color="primary" />
      <br />
      Something went wrong tho.
    </div>

    <q-list bordered class="text-white q-mt-xl list">
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
            color="primary"
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
      loading: false,
      illustration: true,
      fetchError: false,
    };
  },
  methods: {
    async fetchSongs() {
      const response = await fetch(
        `https://n3rd-last-fm-api.glitch.me/getSongs?songQuery=${this.searchText}`
      );
      this.fetchError = false;
      // const response = await fetch(`http://192.168.43.235:1987/getSongs.json`);
      if (response.ok) {
        this.fetchError = false;
        this.illustration = false;
        this.loading = true;
        const songs = await response.json();
        this.loading = false;
        this.songs = songs.tracks.items;
      } else {
        console.error("something went wrong :x");
        this.illustration = false;
        this.fetchError = true;
      }
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
      this.$router.push("/similarMusic");
    },
  },
};
</script>

<style lang="scss" scoped>
// set a search-container variable

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
.search-music {
  font-size: 1.2rem;
  color: #fff !important;
  // set the inner text to white
  & > input {
    color: #fff !important;
  }
}
.q-field__input {
  color: #fff !important;
  text-transform: capitalize;
}
.q-placeholder {
  text-align: center;
}
.list {
  // set an accurate top margin
  margin-top: 121px;
}
.illustration,
.fetch-error {
  // put the illustration on the center of the page
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 1.5rem;
  color: #fff;
}
// make destop styles
@media (min-width: 992px) {
  .illustration,
  .fetch-error {
    font-size: 2rem;
  }
}
</style>
