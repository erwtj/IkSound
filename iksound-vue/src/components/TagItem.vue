<script setup>
const props = defineProps({
  tag: {
    displayTag: String,
    slug: String,
    numTracks: Number
  },
  type: String, 
  isSfx: Boolean
})
</script>

<template>
  <li v-if="tag.children.length > 0 || tag.numTracks > 0 || type === 'mood'">
    <RouterLink :to="`/search?${type}=${tag.slug}&sfx=${isSfx}`" class="clickable-link">{{ tag.displayTag }}</RouterLink>
    <ul v-if="tag.children.length">
      <TagItem v-for="child in tag.children" :key="child.id" :tag="child" class="sub-genre" :is-sfx="isSfx" :type="type"></TagItem>
    </ul>
  </li>
</template>

<style scoped>
.sub-genre {
  color: rgba(222, 226, 230, 0.5);
}
</style>