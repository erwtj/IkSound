<script setup>

import {computed, nextTick, onBeforeUnmount, onMounted, ref} from "vue";
import {api} from "../../api.js";
import {useRoute} from "vue-router";
import TrackComponent from "../../components/TrackListItem.vue";
import AudioPlayer from "../../components/AudioPlayer.vue";

const route = useRoute();

const tracks = ref([]);
const selectedTrack = ref(null);
const audioPlayer = ref();
const page = ref(1);
const loading = ref(false);
const totalPages = ref(-1);
const totalHits = ref(-1);
const headGenre = ref('');
const subGenre = ref('');

const loadTracks = async (page) => {
  loading.value = true;
  
  const type = route.query.sfx.toLowerCase() === 'true' ? 'sfx' : 'tracks';
  const term = route.query.term;
  const genre = route.query.genre;
  const mood = route.query.mood;

  api.get(`json/search/${type}/?`, {
    params: {
      page: page,
      limit: 15,
      term: term,
      genres: genre,
      moods: mood,
      sort: 'relevance',
      order: 'desc'
    }
  })
  .then((response) => {
    const newTracks = Object.values(response.data.entities.tracks);
    tracks.value.push(...newTracks);
    loading.value = false;
    
    totalPages.value = response.data.meta.totalPages;
    totalHits.value = response.data.meta.totalHits;
    
    if (genre) {
      const genres = response.data.meta.aggregations.genres;
      
      headGenre.value = genres[0];
      subGenre.value = genres.find(g => g.key === genre);
    }
  });
}

const playTrack = (track) => {
  if (!audioPlayer.value) return;

  if (selectedTrack.value?.id === track.id) {
    audioPlayer.value.togglePlay();
    return;
  }

  selectedTrack.value = track;
}

const seekTrack = async (track, event) => {
  if (!audioPlayer.value) return;

  if (selectedTrack.value?.id !== track.id) {
    selectedTrack.value = track;
    await nextTick();
  }

  audioPlayer.value.handleWaveformClick(event);
}

const isTrackPlaying = (track) => {
  if (!audioPlayer.value) return false;

  return selectedTrack.value?.id === track.id && audioPlayer.value.playing;
}

const handleScroll = (e) => {
  const scrollPosition = document.documentElement.scrollTop || document.body.scrollTop;
  const scrollHeight = document.documentElement.scrollHeight || document.body.scrollHeight;
  const clientHeight = document.documentElement.clientHeight || document.body.clientHeight;
  const dist = scrollHeight - (scrollPosition + clientHeight)

  if(dist <= 0) {
    loadMore();
  }
}

const loadMore = () => {
  if (loading.value)
    return;
  
  page.value++;
  loadTracks(page.value)
}

const isSfx = computed(() => {
  return route.query.sfx.toLowerCase() === 'true';
});

onMounted(() => {
  loadTracks(page.value);
  document.onscroll = handleScroll;
});

onBeforeUnmount(() => {
  document.onscroll = null;
});
</script>

<template>
<AudioPlayer ref="audioPlayer" :audioUrl="selectedTrack?.stems?.full?.lqMp3Url"/>
<div class="2xl:px-16">

  <h1 v-if="route.query.genre && headGenre" class="fade-in text-secondary !text-lg my-2">
    <RouterLink class="clickable-link" :to="isSfx ? 'sounds' : 'music'">
      {{ isSfx ? 'Sound Effects' : 'Music' }}
    </RouterLink>
    /
    <RouterLink class="clickable-link" v-if="subGenre.key !== headGenre.key" :to="{ path: 'search', query: { ...route.query, genre: headGenre.key }, }">
      {{ headGenre.displayKey }}
    </RouterLink>
    <span v-else>
      {{ headGenre.displayKey }}
    </span>
    {{ subGenre.key !== headGenre.key ? `/ ${subGenre.displayKey}` : '' }}
  </h1>
  <h1 v-else-if="route.query.term && totalHits > 0" class="fade-in text-secondary !text-lg my-2">
    <span class="text-gray-300">{{totalHits}}</span> results for <i class="text-gray-300">"{{route.query.term}}"</i>
  </h1>

  <div v-if="totalPages === 0" class="fade-in w-full text-center">
    <strong>No results found...</strong>
    <p class="text-secondary">Try searching something else</p>
  </div>
  
  <ul class="flex flex-col gap-2">
    <li v-for="track in tracks" :key="track.id">
      <TrackComponent
          :track="track"
          :playing="isTrackPlaying(track)"
          :progress="selectedTrack?.id === track.id ? audioPlayer?.progress : null"
          :isSelectedTrack="selectedTrack?.id === track.id"
          :handleWaveformClick="(event) => seekTrack(track, event)"
          @togglePlay="playTrack(track)"
      />
    </li>
  </ul>
  
  <button v-if="tracks.length !== 0 && totalPages > page" @click="loadMore" class="fade-in" id="load-more">
    Load more
  </button>
</div>
</template>

<style scoped>
#load-more {
  @apply text-center w-full mt-4 p-1;
  background: #181b1e;
}

#load-more:hover {
  background: #1c1c1c;
}

#load-more:active {
  background: #3b3b3b;
}
</style>
