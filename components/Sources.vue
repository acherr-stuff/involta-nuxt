<template>
  <div class="sources-wrapper">
    <ul id="sources">
      <li>
        <router-link
          :class="{ current: currentSource === undefined }"
          :to="{ query: {} }"
          replace
          >Все</router-link
        >
      </li>
      <li v-for="source in sources" :key="source">
        <router-link
          :class="{
            current: source
              .toLowerCase()
              .includes((currentSource || ' ').toLowerCase()),
          }"
          :to="{ query: { source: beautifySource(source) } }"
          replace
          >{{ ucFirst(beautifySource(source)) }}</router-link
        >
      </li>
    </ul>
  </div>
</template>
<script>
export default {
  props: ["currentSource", "sources"],
  methods: {
    beautifySource(source) {
      return new URL(source).hostname;
    },
    ucFirst(str) {
      return str[0].toUpperCase() + str.slice(1)
    },
  },
};
</script>
