<template>
  <input class="header__search" id="search" type="text" v-model="input" />
</template>
<script>
export default {
  props: ["searchQuery"],
  computed: {
    input: {
      get: function () {
        return this.searchQuery;
      },
      set: function (newValue) {
        let query = Object.assign({}, this.$route.query);
        delete query.page;
        this.$router.replace({ query });
        this.$emit("update:searchQuery", newValue);
      },
    },
  },
};
</script>
