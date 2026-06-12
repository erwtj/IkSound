<script setup>
import {useRoute} from "vue-router";
import {computed, onMounted, ref} from "vue";
import {api} from "../api.js";
import Waveform from "../components/Waveform.vue";
import {download} from "../utils.js";

const route = useRoute();

const track = ref();

function fetchTrack(trackId) {
  api.get(`json/track/${trackId}`).then((response) => {
    track.value = response.data;
    console.log(response.data)
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
    <div class="h-64 w-full bg-gradient flex flex-row p-6 gap-4">
      <div class="flex flex-col gap-2 mx-4 mt-4 p-2">
        <h4 class="text-xl text-secondary">{{track.isSfx ? "Sound Effect" : "Track"}}</h4>
        <h1 class="text-3xl">{{ track.title }}</h1>
        <p v-if="track.creatives.mainArtists.length > 0"><span class="text-secondary">By </span> {{ track.creatives.mainArtists.map((arist) => arist.name).join(', ') }}</p>
        
        <p class="text-secondary">
          <span v-for="genre in track.genres" :key="genre.slug">
            <RouterLink class="clickable-link" :to="`/search?genre=${genre.slug}&sfx=${track.isSfx}`">
              {{ genre.displayTag }}
            </RouterLink>{{ track.genres[track.genres.length - 1].slug !== genre.slug ? ', ' : '' }}
          </span>
        </p>
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
        
        <img :src="track.coverArt.baseUrl + track.coverArt.sizes.L" alt="Cover Art" class="h-full ms-auto"/>
      </div>
    </div>
    
    <div class="h-14 w-full bg-gradient">
      <div class="content-center relative cursor-pointer flex-grow hidden min-[900px]:flex" @click="handleWaveformClick">
<!--        <span v-if="playProgress !== null" class="h-full absolute z-10 w-px bg-white top-0 left-0 smooth-move" :style="`left: ${playProgress * 100}%;`"></span>-->
        <Waveform :waveform-url="track.stems.full.waveformUrl"/>
      </div>
    </div>
  </div>
</template>

<style scoped>
@property --gradient-end {
  syntax: '<color>';
  initial-value: #212529;
  inherits: false;
}

.empty-btn {
  border: none;
  background: none;
  color: inherit;
  font: inherit;
  cursor: pointer;
  outline: inherit;
  align-content: center;
  text-align-last: center;
  place-items: center;
  aspect-ratio: 1;
  
  padding: 0.75rem;
}

.empty-btn:hover {
  background-color: #181b1e;
}

.empty-btn:active, .wide-btn:active {
  background-color: #3b3b3b;
}
</style>