<template>
  <ul id="news">
    <li
      class="items"
      v-bind:class="{ withoutImage: isImageOff }"
      v-for="item in items"
      :key="item.title"
    >
      <div class="item-img-with-description">
        <p class="item-img" v-if="imageVisibility === true">
          <img :src="item.enclosure.url" v-if="item.enclosure" />
        </p>
        <div class="item-description">
          <p class="item-description__title item-title">
            <a :href="item.link" target="_blank">{{ item.title }}</a>
          </p>
          <p
            class="item-description__content item-content"
            v-if="item.content"
            v-html="item.content"
          ></p>
        </div>
      </div>
      <div class="item-sources">
        <p class="item-sources__source source">
          <a :href="makeOriginLink(item.link)" target="_blank">{{
            beautifySource(item.link)
          }}</a>
        </p>
        <p class="item-sources__date date">{{ parseDate(item.pubDate) }}</p>
      </div>
    </li>
  </ul>
</template>
<script>
import moment from "moment";

export default {
  props: ["isImageOff", "imageVisibility", "items"],
  methods: {
    beautifySource(source) {
      return new URL(source).hostname;
    },
    makeOriginLink(link) {
      return new URL(link).origin;
    },
    parseDate(str) {
      const date = moment(str);
      return date.format("DD.MM.YYYY");
    },
  },
};
</script>
