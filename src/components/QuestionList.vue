<template>
  <ul>
    <li v-for="(track, index) in tracks" :key="index">
      <!-- <img :src="track.album.images[0].url" alt="" /> -->
      <p>{{ track.name }} | {{ track.album.artist.name }}</p>
      <a :href="track.url" target="_blank" style="display: none"
        >Link to Page</a
      >
      <!-- <p>ID: {{ track.id }}</p> -->
      <!-- <track-player :trackID="track.id"></track-player> -->
    </li>
  </ul>
</template>

<script>
export default {
  name: "question-list",
  props: {
    tracks: {
      type: Object,
      required: true,
    },
  },
};
</script>
