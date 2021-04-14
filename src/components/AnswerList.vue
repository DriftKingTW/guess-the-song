<template>
  <div>
    <div
      class="game-button-container"
      v-for="(track, index) in tracks"
      :key="index"
    >
      <a
        href="#"
        class="transparent"
        @click.prevent="$emit('handleUserAnswer', track.id)"
        :disabled="!isDisable"
      >
        <!-- <img :src="track.album.images[0].url" alt="" /> -->
        <p>
          <span class="bg"></span>
          <span class="base"></span>
          <span class="text"
            >{{ track.name }} | {{ track.album.artist.name }}</span
          >
        </p>
        <!-- <a :href="track.url" target="_blank" style="display: none"
          >Link to Page</a
        > -->
        <!-- <p>ID: {{ track.id }}</p> -->
        <!-- <track-player :trackID="track.id"></track-player> -->
      </a>
    </div>
  </div>
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
