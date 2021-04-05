<template>
  <ul>
    <li v-for="(track, index) in tracks" :key="index">
      <button
        @click.prevent="$emit('handleUserAnswer', track.id)"
        :disabled="!isDisable"
      >
        <!-- <img :src="track.album.images[0].url" alt="" /> -->
        <p>{{ track.name }} | {{ track.album.artist.name }}</p>
        <!-- <a :href="track.url" target="_blank" style="display: none"
          >Link to Page</a
        > -->
        <!-- <p>ID: {{ track.id }}</p> -->
        <!-- <track-player :trackID="track.id"></track-player> -->
      </button>
    </li>
  </ul>
</template>

<script>
export default {
  name: "answer-list",
  props: {
    tracks: {
      type: Object,
      required: true,
    },
    isDisable: {
      type: Boolean,
      required: true,
    },
  },
};
</script>
