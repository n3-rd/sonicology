<template>
  <div>
    <q-toolbar class="bg-primary fixed-top z-max">
      <q-toolbar-title>
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
              <q-img :src="song.album.images[0].url" alt="album art" :ratio="1">
                <div
                  class="play-button absolute-bottom-left all-pointer-events cursor-pointer"
                  v-if="song.preview_url"
                  @click="playSound(song.preview_url, song.name)"
                  style="width: 68px"
                >
                  <q-icon name="play_arrow" color="primary" size="xl" />
                </div>
                <a
                  :href="song.external_urls.spotify"
                  class="q-pa-md absolute-bottom-right all-pointer-events cursor-pointer"
                  target="_blank"
                  style="width: 68px"
                >
                  <img
                    alt=""
                    src="spotify-2.svg"
                    class="spotify-logo-desktop cursor-pointer"
                    width="100%"
                  />
                </a>
              </q-img>
            </div>

            <div class="song-name">
              {{ song.name }}
            </div>
            <div class="song-artist">
              {{ song.artists[0].name }}
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
          v-if="song.preview_url"
        ></q-btn>

        <a
          :href="song.external_urls.spotify"
          target="_blank"
          class="q-pa-md absolute-bottom-right all-pointer-events cursor-pointer"
        >
          <img
            alt=""
            src="spotify-2.svg"
            class="spotify-logo-mobile cursor-pointer"
          />
        </a>
      </swiper-slide>
    </swiper>

    <div>
      <q-dialog v-model="playing" position="bottom" persistent>
        <q-card style="width: 350px">
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
  </div>
</template>
<script>
import { ref } from "vue";
import { Howl, Howler } from "howler";
import { Swiper, SwiperSlide } from "swiper/vue";

import "swiper/css";

import "swiper/css/effect-cards";

import { EffectCards } from "swiper";
import notify from "@gabeotisbenson/notify";

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
    if (window.matchMedia("(max-width: 600px)").matches) {
      notify.log(`Swipe to go to next or previous song`);
    }

    setTimeout(() => {
      this.preloader = false;
    }, 3000);
  },
};
</script>

<style lang="scss" scoped>
.preloader {
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
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 1.5rem;
  color: #fff;
}
.swiper-container {
  width: 100%;
  height: 100%;
}
.swiper {
  width: 100%;
  height: 100vh;
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
  height: 100vh;
  @media (min-width: 992px) {
    width: 414px;
    height: 414px;
  }
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
  width: 33%;
  img {
    width: 100%;
  }
}
.q-img__content > div {
  background: transparent;
  cursor: pointer;
}
.spotify-logo-mobile {
  width: 33px !important;
  margin-bottom: 33px !important;
}
</style>
