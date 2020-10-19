<script>
import moment from "moment";

import Sources from '~/components/Sources';
import Pagination from '~/components/Pagination';

let url = require('url');
let Parser = require('rss-parser');
let parser = new Parser({
  customFields: {
    item: ['description'],
  }
});
export default {
  components: {
    Sources,
    Pagination,
  },
  data() {
    return {
      sources: [
        'https://mos.ru/rss',
        'https://lenta.ru/rss/news',
      ],
      currentSource: this.$route.query['source'],
      feeds: [],
      itemsPerPage: 4,
      currentPage: parseInt(this.$route.query['page'], 10) || 1,
      searchQuery: '',
      imageVisibility: true,
      activeElement: undefined,
      isWithImageActive: true,
      isWithoutImageActive: false,
      isImageOff: false,
      areAllSourcesActive: true
    }
  },
  computed: {
    items: function () {

      // Если не находим источник по индексу, то возвращаем элементы из всех лент
      if (!this.sources.some(source => source.includes(this.currentSource))) {
        return this.feeds
          .map(feed => feed.items)
          .flat()
          .sort((a, b) => moment(a.pubDate) < moment(b.pubDate))
          .filter(item => item.title.toLowerCase().includes(this.searchQuery.toLowerCase()) || item.content.toLowerCase().includes(this.searchQuery.toLowerCase()));
      }

      // Находим нужную ленту сравнивая имя хоста у элемента rss>channel>link
      const feed = this.feeds.find(feed => feed.link.includes(this.currentSource));

      // Если не находим ленту, то возвращаем элементы из всех лент
      if (feed === undefined) {
        return this.feeds
          .map(feed => feed.items)
          .flat()
          .sort((a, b) => moment(a.pubDate) < moment(b.pubDate))
          .filter(item => item.title.toLowerCase().includes(this.searchQuery.toLowerCase()) || item.content.toLowerCase().includes(this.searchQuery.toLowerCase()));
      } else {
        return feed.items
          .sort((a, b) => moment(a.pubDate) < moment(b.pubDate))
          .filter(item => item.title.toLowerCase().includes(this.searchQuery.toLowerCase()) || item.content.toLowerCase().includes(this.searchQuery.toLowerCase()));
      }
    },
    slicedItems() {

      // Готовим индексы для обрезания массива: стартовый и конечный
      // Стартовы индекс - это наш отступ (offset), а конечный offset + количество элементов на странице
      const start = ((this.currentPage - 1) * this.itemsPerPage), end = start + this.itemsPerPage;

      // Обрезаем массив со start и до индекса меньше end, https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Array/slice
      return this.items.slice(start, end);
    },
    pagesCount: function () {
      return Math.round(this.items.length / this.itemsPerPage);
    },

  },
  methods: {

  },
  watch: {
    $route(route) {

      // Следим за маршрутизацией и меняем данные
      this.currentPage = parseInt(route.query['page'], 10) || 1;
      this.currentSource = route.query['source'];
    },
  },
  async fetch() {
    const CORS_PROXY = "https://cors-anywhere.herokuapp.com/"
    this.feeds = await Promise.all(this.sources.map(source => parser.parseURL(CORS_PROXY + source)))
  }
}
</script>

<template>
  <section class="container">
    <div id="newslist">
      <div class="header">
      <div class="header-title">
      <h1>Список новостей</h1>
      <button  class="header-title__refresh" id="refresh" @click="$fetch"></button>
      </div>
      <search v-bind:searchQuery.sync="searchQuery" />
    </div>
    </div>
    <div class="menu">

    <sources :currentSource="currentSource" :sources="sources" />

    <icons
    v-bind:imageVisibility.sync="imageVisibility"
v-bind:isWithImageActive.sync="isWithImageActive"
v-bind:isWithoutImageActive.sync="isWithoutImageActive"
v-bind:isImageOff.sync="isImageOff"
     />
</div>

    <items :isImageOff="isImageOff" :imageVisibility="imageVisibility" :items="slicedItems" />

    <pagination :currentPage="currentPage" :pagesCount="pagesCount" />

  </section>
</template>
