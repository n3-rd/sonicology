<template>
  <div>
    <!-- add an header -->

    <q-toolbar class="bg-primary fixed-top z-max">
      <q-toolbar-title>
        <!-- router-link to /index -->
        <router-link to="/">
          <q-icon name="arrow_back" class="q-mr-md" />
        </router-link>
        <span>Similar Music to {{ baseSongName }}</span>
      </q-toolbar-title>
    </q-toolbar>

    <div class="container gt-sm">
      <div class="row">
        <div
          class="col-md-4 q-pa-xl song-grid"
          v-for="song in similarSongs"
          :key="song.id"
        >
          <div>
            <div class="album-art">
              <img :src="song.album.images[0].url" alt="album art" />
            </div>

            <div class="song-name">
              {{ song.name }}
            </div>
            <div class="song-artist">
              {{ song.artists[0].name }}
              <div
                class="play-button"
                v-if="song.preview_url"
                @click="playSound(song.preview_url, song.name)"
              >
                <q-icon name="play_arrow" color="primary" />
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <swiper
      :effect="'cards'"
      :grabCursor="true"
      :modules="modules"
      class="mySwiper lt-md"
    >
      <swiper-slide v-for="song in similarSongs" :key="song.id">
        <div class="album-art">
          <img :src="song.album.images[0].url" />
        </div>

        <div class="song-info">
          <div class="song-name q-py-md">
            {{ song.name }}
          </div>
          <div class="song-artist q-py-md">
            {{ song.artists[0].name }}
          </div>
        </div>

        <q-btn
          @click="playSound(song.preview_url, song.name)"
          color="primary"
          label="Play"
          icon="play_arrow"
          class="q-mr-sm"
        ></q-btn>
      </swiper-slide>
    </swiper>

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

    <div class="preloader q-px-lg" v-if="preloader">
      <div class="text">Getting Similar songs to {{ baseSongName }}</div>
      <q-spinner-audio color="primary" size="2em" />
    </div>

    <div class="fetch-error" v-if="fetchError">
      <q-icon name="error" size="xl" color="primary" />
      <br />
      Something went wrong tho.
    </div>

    <!-- <q-list bordered class="text-white q-mt-xl list">
      <div v-for="song in similarSongs" :key="song.id">
        <q-item
          class="q-my-sm"
          clickable
          v-ripple
          v-if="song.album.images.length && song.preview_url"
        >
          <q-item-section avatar>
            <q-avatar color="primary" text-color="white">
              <q-img
                v-if="song.album.images.length"
                :src="song.album.images[0].url"
                spinner-color="primary"
                spinner-size="82px"
              />
            </q-avatar>
          </q-item-section>

          <q-item-section @click="storeSongInfo(song.name, song.id)">
            <q-item-label class="song-name">{{ song.name }}</q-item-label>
            <q-item-label caption lines="1" class="text-white song-artist">

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
      </div>
    </q-list> -->
  </div>
</template>
<script>
import { ref } from "vue";
import { Howl, Howler } from "howler";
import { Swiper, SwiperSlide } from "swiper/vue";

// Import Swiper styles
import "swiper/css";

import "swiper/css/effect-cards";

// import required modules
import { EffectCards } from "swiper";

export default {
  name: "similarMusic",
  components: {
    Swiper,
    SwiperSlide,
  },
  data() {
    return {
      baseSongId: "",
      baseSongName: "",
      similarSongs: [],
      slide: "style",
      preloader: true,
      fetchError: false,
      playing: false,
      currentSong: "",
      modules: [EffectCards],
    };
  },
  methods: {
    async fetchSimilarSongs() {
      const response = await fetch(
        `https://n3rd-last-fm-api.glitch.me/getSimilarTracks?seedTrack=${this.baseSongId}`
      );
      this.fetchError = false;

      // const response = await fetch(
      //   `http://localhost:1987/getsimilarTracks.json`
      // );
      if (response.ok) {
        this.fetchError = false;
        const songs = await response.json();
        this.similarSongs = songs.tracks;
      } else {
        this.fetchError = true;
        console.error("something went wrong :x");
      }
    },
    playSound(song, songName) {
      const sound = new Howl({
        src: [song],
        html5: true,
      });
      this.currentSong = songName;
      this.playing = true;
      sound.play();
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
.fetch-error {
  // put the error on the center of the page
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 1.5rem;
  color: #fff;
}
// style swipers
.swiper-container {
  width: 100%;
  height: 100%;
}
.swiper-slide {
  text-align: center;
  background-color: #fff;
  border-radius: 18px;
  font-size: 22px;
  font-weight: bold;
  color: #ffffff;
  background-color: #000;
  width: 100vw;
  /* width: 414px; */
  height: 100vh;
  // format desktop
  @media (min-width: 992px) {
    width: 414px;
    height: 414px;
  }
}
.swiper-slide {
}
.swiper-slide img {
  width: 100%;
  height: 100%;
}
.song-info {
  font-size: 1.5rem;
  color: #fff;
  .song-name {
    font-size: 1.5rem;
    color: #fff;
  }
}
.song-artist {
  font-size: 1rem;
  color: #fff;
}
// format responsiveness
@media (max-width: 768px) {
  .swiper-slide {
    width: 100%;
    height: 100%;
  }
}
@media (max-width: 414px) {
  .swiper-slide {
    width: 100%;
    height: 100%;
  }
}
.song-grid {
  // width: 100%;
  width: 33%;
  img {
    width: 100%;
  }

  // .play-button {
  //   position: absolute;
  //   top: 50%;
  //   left: 50%;
  //   transform: translate(-50%, -50%);
  //   font-size: 1.5rem;
  //   color: #fff;
  // }

  // height: 100%;
}
</style>
