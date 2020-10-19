<template>
  <div class="icons">
    <button
      class="icons__image"
      id="with-image"
      v-bind:class="{ active: isWithImageActive }"
      @click="showImages"
    ></button>
    <button
      class="icons__noimage"
      id="without-image"
      v-bind:class="{ active: isWithoutImageActive }"
      @click="hideImages"
    ></button>
  </div>
</template>
<script>
export default {
  props: [
    "imageVisibility",
    "isWithImageActive",
    "isWithoutImageActive",
    "isImageOff"
  ],
  methods: {
    hideImages() {
      this.$emit("update:imageVisibility", false)
      this.$emit("update:isWithImageActive", false)
      this.$emit("update:isWithoutImageActive", true)
      this.$emit("update:isImageOff", true)
    },
    showImages() {
      this.$emit("update:imageVisibility", true)
      this.$emit("update:isWithImageActive", true)
      this.$emit("update:isWithoutImageActive", false)
      this.$emit("update:isImageOff", false)
    },
  },
};
</script>
