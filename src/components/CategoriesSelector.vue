<template>
  <div id="category-container">
    <section class="accordion">
      <input type="checkbox" name="collapse" id="handle1" v-model="collapse" /> <!--  checked="checked"  -->
      <h2 class="handle">
        <label for="handle1">Category</label>
      </h2>
      <div class="content">
        <ul class="tags flex-container">
          <li v-for="(cat, index) in categories" :key="index">
            <a
              href="#"
              class="tag"
              @click="collapseSection(); $emit('handleSelectCategory', cat.id, cat.title); "
            >
              {{ cat.title }}
            </a>
          </li>
        </ul>
      </div>
    </section>
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
  data() {
    return {
      collapse: "checked"
    }
  },
  methods: {
    collapseSection() {
      this.collapse = ""
    }
  }
};
</script>
