<script setup>
import { ref, onMounted } from "vue"
import axios from "axios";

const commits = ref([])
const loading = ref(true)
const error = ref(null)

async function fetchCommits() {
  axios.get(
      "https://api.github.com/repos/erwtj/IkSound/commits?per_page=3"
  ).then((res) => {
    commits.value = res.data
  }).catch((err) => {
    error.value = err.message
  }).finally(() => {
    loading.value = false
  })
}

onMounted(fetchCommits)
</script>

<template>
  <div class="px-4 text-center items-center">
    <div class="p-">
      <h1 class="text-8xl">Welcome to IkSound</h1>
      <p class="text-2xl">The best place to find music and sounds for your projects</p>
    </div>

    <div>
      <div class="m-auto mt-12 rounded-full overflow-hidden" style="width: 200px;">
        <a href="https://github.com/erwtj">
          <img class="size-full" src="https://avatars.githubusercontent.com/u/57986957" alt="Me"/>
        </a>
      </div>
      <label class="opacity-45">(this is me btw)</label>
    </div>

    <div class="shadow-lg mt-12 p-6 w-1/2 ml-auto mr-auto text-lg">
      <p class="mb-4">
        This website can be used to download the free previews of the songs and sound effects from the Epidemic Sound library.
      </p>
      <p class="mb-4">
        However, these preview files don't come with a license to use them in your projects. You can get the license by subscribing to Epidemic Sound.
      </p>

      <p class="mb-4">
        If you have any questions or suggestions, feel free to contact me at <a href="mailto:iknobox@gmail.com" class="text-blue-500 cursor-pointer underline">iknobox@gmail.com</a>!
      </p>
    </div>
  </div>

  <div class="shadow-lg mt-12 p-6 w-1/2 ml-auto mr-auto text-left">
    <h2 class="text-2xl font-semibold mb-4">Latest GitHub Updates</h2>

    <div v-if="loading">Loading updates...</div>
    <div v-else-if="error" class="text-red-500">{{ error }}</div>

    <ul v-else>
      <li
          v-for="commit in commits"
          :key="commit.sha"
          class="mb-3 border-b pb-2"
      >
        <a
            :href="commit.html_url"
            target="_blank"
            class="font-medium text-white hover:underline"
        >
          {{ commit.commit.message }}
        </a>
        <p class="text-sm opacity-60">
          {{ new Date(commit.commit.author.date).toLocaleDateString() }}
        </p>
      </li>
    </ul>
  </div>


</template>

<style scoped>

</style>