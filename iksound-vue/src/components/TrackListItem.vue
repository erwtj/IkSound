<script setup>
import {computed, watch} from "vue";
import AudioPlayer from "./AudioPlayer.vue";
import Waveform from "./Waveform.vue";
import corsProxy from "../corsSetting.js";
import router from "../router.js";
import {download} from "../utils.js";

const props = defineProps(
    {
      playProgress: Number | null,
      playing: Boolean,
      track: {
        id: Number,
        kosmosId: String,
        title: String,
        added: String,
        creatives: {
          composers: Array,
          mainArtists: Array,
          featuredArtists: Array,
          producers: Array
        },
        length: Number,
        durationMs: Number,
        bpm: Number,
        isSfx: Boolean,
        hasVocals: Boolean,
        hidden: Boolean,
        publicSlug: String,
        genres: Array.of({slug: String, tag: String, displayTag: String}),
        moods: Array,
        energyLevel: String,
        stems: {
          full: {
            stemType: String,
            s3TrackId: Number,
            lqMp3Url: String,
            waveformUrl: String
          }
        },
        oldTitle: [String, null],
        seriesId: [String, null],
        metadataTags: Array,
        isExplicit: Boolean,
        isCommercialRelease: Boolean,
        imageUrl: String,
        coverArt: {
          baseUrl: String,
          sizes: {
            XS: String,
            S: String,
            M: String,
            L: String
          }
        },
        releaseDate: [String, null],
        segmentGroups: Array
      }
    }
);

const emit = defineEmits(['play', 'pause', 'playAt']);

const url = computed(() => {
  if (!props.track.value) return '';
  
  return props.track.value.isSfx ?
      `https://www.epidemicsound.com/sound-effects/tracks/${props.track.value.kosmosId}` :
      `https://www.epidemicsound.com/track/${props.track.value.publicSlug}`;
})

const play = () => {
  if (props.playing) {
    emit('pause');
  } else {
    emit('play');
  }
};

const moods = computed(() =>
    props.track.moods.map(mood => mood.displayTag).join(', ')
);

const handleWaveformClick = (e) => {
  const at = e.offsetX / e.target.offsetWidth;
  emit('playAt', at);
}
</script>

<template>
  <div class="bg-gradient bg-gradient-hover h-14 w-auto flex flex-row fade-in lg:gap-2 group overflow-hidden">
    <div class="flex gap-2 items-center h-14 overflow-hidden">
      <div class="inline size-14 cursor-pointer">
        <div class="audio-control hidden group-hover:block" @click="play">
          <svg v-if="!playing" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5"
               stroke="currentColor" class="size-6">
            <path stroke-linecap="round" stroke-linejoin="round"
                  d="M5.25 5.653c0-.856.917-1.398 1.667-.986l11.54 6.347a1.125 1.125 0 0 1 0 1.972l-11.54 6.347a1.125 1.125 0 0 1-1.667-.986V5.653Z"/>
          </svg>
          <svg v-else xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5"
               stroke="currentColor" class="size-6">
            <path stroke-linecap="round" stroke-linejoin="round" d="M15.75 5.25v13.5m-7.5-13.5v13.5"/>
          </svg>
        </div>
        <img class="size-14 max-w-fit" :src="track.coverArt.baseUrl + track.coverArt.sizes.XS" :alt="track.title"/>
      </div>
      <div class="flex-col overflow-hidden 2xl:w-80 xl:w-64 w-32 max-sm:w-64 flex mr-2">
        <RouterLink class="clickable-link truncate" :to="`track/${track.publicSlug}`">{{ track.title }}</RouterLink>
        <a v-if="track.creatives.mainArtists && track.creatives.mainArtists.length > 0"
           class="clickable-link truncate text-sm text-secondary"
           target="_blank"
           :href="`https://www.epidemicsound.com/artists/${track.creatives.mainArtists[0].slug}`">{{
            track.creatives.mainArtists[0].name
          }}</a>
      </div>
    </div>

    <div class="content-center h-14 relative cursor-pointer flex-grow hidden min-[900px]:flex" @click="handleWaveformClick">
      <span v-if="playProgress !== null" class="h-full absolute z-10 w-px bg-white top-0 left-0 smooth-move" :style="`left: ${playProgress * 100}%;`"></span>
      <Waveform :waveform-url="track.stems.full.waveformUrl"/>
    </div>

    <div class="text-secondary content-center hidden lg:block">
      <span class="text-sm">
        {{ Math.floor(track.length / 60) }}:{{track.length % 60 < 10 ? '0' : '' }}{{ track.length % 60 }}
      </span>
    </div>

    <div class="content-center overflow-hidden w-32 text-secondary text-sm xl:ms-10 mx-4 hidden min-[801px]:block max-[900px]:ms-auto">
      <RouterLink v-if="track.genres[0]" class="clickable-link w-full block truncate" :to="`search?genre=${track.genres[0].slug}&sfx=${track.isSfx}`">{{ track.genres[0].displayTag }}</RouterLink>
      <span v-else class="w-full fs-7 d-block fst-italic">No genre</span>
      <span v-if="!track.isSfx" class="w-full fs-7 block truncate">{{ moods }}</span>
    </div>

    <div class="text-secondary content-center flex flex-row justify-between max-[800px]:ms-auto max-[800px]:me-4">
      <a class="h-full empty-btn hidden xl:block" :href="url" target="_blank">
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
          <path stroke-linecap="round" stroke-linejoin="round" d="M13.5 6H5.25A2.25 2.25 0 0 0 3 8.25v10.5A2.25 2.25 0 0 0 5.25 21h10.5A2.25 2.25 0 0 0 18 18.75V10.5m-10.5 6L21 3m0 0h-5.25M21 3v5.25" />
        </svg>
      </a>
      <a class="h-full empty-btn" :href="track.stems.full.lqMp3Url" :download="track.title" @click.prevent="download(props.track)">
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"
             class="size-6">
          <path stroke-linecap="round" stroke-linejoin="round"
                d="M3 16.5v2.25A2.25 2.25 0 0 0 5.25 21h13.5A2.25 2.25 0 0 0 21 18.75V16.5M16.5 12 12 16.5m0 0L7.5 12m4.5 4.5V3"/>
        </svg>
      </a>
      <button class="h-full empty-btn" type="button" data-bs-toggle="modal" data-bs-target="#modal-@Id">
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"
             class="size-6">
          <path stroke-linecap="round" stroke-linejoin="round"
                d="M12 6.75a.75.75 0 1 1 0-1.5.75.75 0 0 1 0 1.5ZM12 12.75a.75.75 0 1 1 0-1.5.75.75 0 0 1 0 1.5ZM12 18.75a.75.75 0 1 1 0-1.5.75.75 0 0 1 0 1.5Z"/>
        </svg>
      </button>
    </div>
  </div>
</template>