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
  </div>
</template>

<script>
import { Howl, Howler } from "howler";

export default {
  name: "Player",
  data() {
    return {
      playing: this.songPlaying,
    };
  },
  props: {
    songs: {
      type: Array,
      required: true,
    },
    currentSong: {
      type: String,
      required: true,
    },
    songPlaying: {
      type: Boolean,
      required: true,
    },
  },
  methods: {
    stopSound() {
      Howler.stop();
      this.playing = false;
    },
  },
};
</script>
