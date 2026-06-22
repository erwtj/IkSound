<script setup>
import {useRoute} from "vue-router";
import {computed, onMounted, ref} from "vue";
import {api} from "../api.js";
import Waveform from "../components/Waveform.vue";
import {download} from "../utils.js";
import AudioPlayer from "../components/AudioPlayer.vue";

const route = useRoute();

const track = ref();
const audioPlayer = ref();

function fetchTrack(trackId) {
  api.get(`json/track/${trackId}`).then((response) => {
    track.value = response.data;
  }).catch((error) => {
    console.error("Error fetching track:", error);
  });
}

const url = computed(() => {
  return track.value.isSfx ?
      `https://www.epidemicsound.com/sound-effects/tracks/${track.value.kosmosId}` :
      `https://www.epidemicsound.com/music/tracks/${track.value.kosmosId}`;
})

function copyToClipboard() {
  window.navigator.clipboard.writeText(window.location).catch((err) => {
    console.error("Could not copy text: ", err);
  });
}

onMounted(() => {
  const trackId = route.params.id;
  fetchTrack(trackId);
});
</script>

<template>
  <div v-if="track" class="fade-in flex flex-col gap-4">
    <div class="h-64 w-full bg-dark flex flex-row p-6 gap-4">
      <div class="flex flex-col gap-1 ml-4">
        <h4 class="text-xl text-secondary">{{track.isSfx ? "Sound Effect" : "Track"}}</h4>
        <h1 class="text-3xl">{{ track.title }}</h1>
        <p v-if="track.creatives.mainArtists.length > 0"><span class="text-secondary">By </span> {{ track.creatives.mainArtists.map((arist) => arist.name).join(', ') }}</p>

        <div class="mt-auto">
          <p class="text-secondary">
            <span v-for="genre in track.genres" :key="genre.slug">
              <RouterLink class="clickable-link" :to="`/search?genre=${genre.slug}&sfx=${track.isSfx}`">
                {{ genre.displayTag }}
              </RouterLink>{{ track.genres[track.genres.length - 1].slug !== genre.slug ? ', ' : '' }}
            </span>
          </p>
          <p class="text-secondary" v-if="track.moods.length > 0">
            <span v-for="mood in track.moods" :key="mood.slug">
              <RouterLink class="clickable-link" :to="`/search?mood=${mood.slug}&sfx=${track.isSfx}`">
                {{ mood.displayTag }}
              </RouterLink>{{ track.moods[track.moods.length - 1].slug !== mood.slug ? ', ' : '' }}
            </span>
          </p>
        </div>
      </div>

      <div class="h-full ms-auto flex flex-row">
        <div class="flex flex-col text-secondary">
          <a class="h-1/3 empty-btn" :href="url" target="_blank">
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
              <path stroke-linecap="round" stroke-linejoin="round" d="M13.5 6H5.25A2.25 2.25 0 0 0 3 8.25v10.5A2.25 2.25 0 0 0 5.25 21h10.5A2.25 2.25 0 0 0 18 18.75V10.5m-10.5 6L21 3m0 0h-5.25M21 3v5.25" />
            </svg>
          </a>
          <a class="h-1/3 empty-btn" :href="track.stems.full.lqMp3Url" :download="track.title" @click.prevent="download(track)">
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"
                 class="size-6">
              <path stroke-linecap="round" stroke-linejoin="round"
                    d="M3 16.5v2.25A2.25 2.25 0 0 0 5.25 21h13.5A2.25 2.25 0 0 0 21 18.75V16.5M16.5 12 12 16.5m0 0L7.5 12m4.5 4.5V3"/>
            </svg>
          </a>
          <button class="h-1/3 empty-btn" type="button" @click.prevent="copyToClipboard">
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
              <path stroke-linecap="round" stroke-linejoin="round" d="M13.19 8.688a4.5 4.5 0 0 1 1.242 7.244l-4.5 4.5a4.5 4.5 0 0 1-6.364-6.364l1.757-1.757m13.35-.622 1.757-1.757a4.5 4.5 0 0 0-6.364-6.364l-4.5 4.5a4.5 4.5 0 0 0 1.242 7.244" />
            </svg>
          </button>
        </div>
        
        <img :src="track.coverArt.baseUrl + track.coverArt.sizes.L" alt="Cover Art" class="aspect-square h-full"/>
      </div>
    </div>
    
    <div class="h-14 flex flex-row w-full bg-dark">
      <div class="inline size-14 cursor-pointer">
        <div class="audio-control" @click="audioPlayer.togglePlay">
          <svg v-if="!audioPlayer.playing" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5"
               stroke="currentColor" class="size-6">
            <path stroke-linecap="round" stroke-linejoin="round"
                  d="M5.25 5.653c0-.856.917-1.398 1.667-.986l11.54 6.347a1.125 1.125 0 0 1 0 1.972l-11.54 6.347a1.125 1.125 0 0 1-1.667-.986V5.653Z"/>
          </svg>
          <svg v-else xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5"
               stroke="currentColor" class="size-6">
            <path stroke-linecap="round" stroke-linejoin="round" d="M15.75 5.25v13.5m-7.5-13.5v13.5"/>
          </svg>
        </div>
      </div>

      <div class="inline size-14 text-center text-secondary content-center">
        {{ Math.floor(audioPlayer.progress * track.length / 60) }}:{{ Math.floor(audioPlayer.progress * track.length % 60) < 10 ? '0' : '' }}{{ Math.floor(audioPlayer.progress * track.length % 60) }}
      </div>

      <div class="h-full content-center relative cursor-pointer flex-grow flex" @click="audioPlayer.handleWaveformClick">
        <span v-if="audioPlayer.progress !== null" class="h-full absolute z-10 w-px bg-white top-0 left-0 smooth-move" :style="`left: ${audioPlayer.progress * 100}%;`"></span>
        <Waveform :waveform-url="track.stems.full.waveformUrl"/>
      </div>

      <div class="inline size-14 text-center text-secondary me-8 content-center">
        {{ Math.floor(track.length / 60) }}:{{track.length % 60 < 10 ? '0' : '' }}{{ track.length % 60 }}
      </div>
    </div>
  </div>

  <AudioPlayer :src="track?.stems?.full?.lqMp3Url" ref="audioPlayer"/>
</template>

<style scoped>
.empty-btn:hover {
  background: #181b1e;
}
</style>