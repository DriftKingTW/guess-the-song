<template>
  <iframe
    :src="getPlayerURL(trackID)"
    frameborder="0"
    allow="autoplay"
    style="display: none"
  ></iframe>
</template>

<script>
export default {
  name: "track-player",
  props: {
    trackID: {
      type: String,
      required: true,
    },
  },
  methods: {
    getPlayerURL(id) {
      return `https://widget.kkbox.com/v1/?id=${id}&type=song&terr=TW&lang=ZH&loop=true&autoplay=true`;
    },
  },
};
</script>
