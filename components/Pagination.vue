<template>
  <ul id="navigation" v-if="pagesCount > 1">
    <li
      class="pages"
      :class="{
        current: currentPage == page,
        last: page == pagesCount && Math.abs(page - currentPage) > 4,
        first: page == 1 && Math.abs(page - currentPage) > 4,
      }"
      v-for="page in pagesCount"
      :key="page"
      v-if="Math.abs(page - currentPage) < 4 || page == pagesCount || page == 1"
    >
      <router-link
        :to="{ query: { ...$route.query, page: page } }"
        style="{ color: blue }"
        replace
        >{{ page }}</router-link
      >
    </li>
  </ul>
</template>

<script>
export default {
  props: [
    'currentPage',
    'pagesCount',
    'beautifySource'
  ]
}
</script>
<style>
li.first::after {
  content:'...';
  font-family: Arial;
font-weight: bold;
font-size: 18px;
line-height: 21px;
  margin-left: 20px;


color: black;
}

li.last::before {
  content:'...';
  margin-right: 20px;
  font-family: Arial;
font-weight: bold;
font-size: 18px;
line-height: 21px;


color: black;
;
}
</style>
