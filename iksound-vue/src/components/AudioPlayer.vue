<script setup>
import {onBeforeUnmount, ref, watch} from "vue";

const props = defineProps({
  audioUrl: String,
});

let animationFrameId;
let pendingSeekRatio = null;

const playing = ref(false);
const progress = ref(null);
const audio = ref();

function play() {
  if (!audio.value) return;

  audio.value.play().catch(() => {});
}

function pause() {
  if (!audio.value) return;

  audio.value.pause();
}

function togglePlay() {
  if (!audio.value) return;

  if (audio.value.paused) {
    play();
  } else {
    pause();
  }
}

function handleWaveformClick(e) {
  if (!audio.value) return;

  const target = e.currentTarget || e.target;
  const rect = target.getBoundingClientRect();
  const at = Math.min(Math.max((e.clientX - rect.left) / rect.width, 0), 1);

  progress.value = at;

  if (!isNaN(audio.value.duration) && audio.value.duration > 0) {
    audio.value.currentTime = at * audio.value.duration;
  } else {
    pendingSeekRatio = at;
  }

  play();
}

function updateProgress() {
  if (!audio.value || audio.value.paused) return;

  if (!isNaN(audio.value.duration))
    progress.value = audio.value.currentTime / audio.value.duration || 0;

  animationFrameId = requestAnimationFrame(updateProgress);
}

function stopProgress() {
  cancelAnimationFrame(animationFrameId);
}

function onPlay() {
  playing.value = true;
  updateProgress();
}

function onPause() {
  playing.value = false;
  stopProgress();
}

function onLoadedMetadata() {
  if (pendingSeekRatio === null || !audio.value || !audio.value.duration) return;

  audio.value.currentTime = pendingSeekRatio * audio.value.duration;
  pendingSeekRatio = null;
}

function onEnded() {
  playing.value = false;
  progress.value = 0;
  stopProgress();
}

defineExpose({
  playing,
  progress,
  togglePlay,
  pause,
  handleWaveformClick,
});

watch(() => props.audioUrl, (newVal) => {
  stopProgress();
  progress.value = 0;
  pendingSeekRatio = null;

  if (!newVal || !audio.value) {
    playing.value = false;
    return;
  }

  play();
}, {flush: 'post'});

onBeforeUnmount(() => {
  stopProgress();
});
</script>

<template>
  <audio
      :src="audioUrl"
      type="audio/mpeg"
      preload="none"
      ref="audio"
      @ended="onEnded"
      @play="onPlay"
      @pause="onPause"
      @loadedmetadata="onLoadedMetadata"
  />
</template>

<style scoped>
</style>
