<template>
  <div>
    <sapn v-for="(cat, index) in categories" :key="index">
      <button @click="$emit('handleSelectCategory', cat.id, cat.title)">
        {{ cat.title }}
      </button>
    </sapn>
  </div>
</template>

<script>
export default {
  name: "categories-selector",
  props: {
    categories: {
      type: Array,
      required: true,
    },
  },
};
</script>
