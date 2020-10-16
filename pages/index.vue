<script>
import moment from "moment";
let url = require('url');
let Parser = require('rss-parser');
let parser = new Parser({
  customFields: {
    item: ['description'],
  }
});
export default {
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

    beautifySource(source) {
      return new URL(source).hostname;
    },
    makeOriginLink(link) {
      return new URL(link).origin;
    },
    ucFirst(str) {
      return str[0].toUpperCase() + str.slice(1)
    },
    parseDate(str) {
      const date = moment(str)
      return date.format('DD.MM.YYYY')
    },
    hideImages() {
      this.imageVisibility=false;
      this.isWithImageActive = false;
      this.isWithoutImageActive = true;
      this.isImageOff = true;
    },
     showImages() {
      this.imageVisibility=true;
      this.isWithImageActive = true;
      this.isWithoutImageActive = false;
      this.isImageOff = false;

    },
    onDone(el) {
        if (this.activeElement) {
            this.activeElement.classList.remove("activeItem")
        }

        this.activeElement = el;
        this.activeElement.classList.add("activeItem")
    }

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

<style scoped>

</style>

<template>
  <section class="container">
    <div id="newslist">
      <div class="header">
      <div class="header-title">
      <h1>Список новостей</h1>
      <button  class="header-title__refresh" id="refresh" @click="$fetch"></button>
      </div>
      <input class="header__search" id="search" type="text" v-model="searchQuery" />
    </div>
    </div>
    <div class="menu">
        <div class="sources-wrapper">
    <ul id="sources">
      <li>
        <router-link :class="{current: currentSource === undefined }" :to="{ query: {} }" replace>Все</router-link>
      </li>
      <li v-for="source in sources" :key="source">

        <router-link :class="{current: source.toLowerCase().includes((currentSource || ' ').toLowerCase()) }" :to="{ query: { source: beautifySource(source) } }" replace>{{ ucFirst(beautifySource(source)) }}</router-link>
      </li>
    </ul>
    </div>
    <div class="icons">
<button  class="icons__image" id="with-image" v-bind:class="{ active: isWithImageActive }" @click="showImages"></button>
<button class="icons__noimage" id="without-image" v-bind:class="{ active: isWithoutImageActive }" @click="hideImages"></button>
    </div>
</div>
    <ul id="items">
      <li class="items" v-bind:class="{ withoutImage: isImageOff }" v-for="item in slicedItems" :key="item.title">
        <div class="item-img-with-description">
        <p class="item-img" v-if="imageVisibility === true"><img  :src="item.enclosure.url" v-if="item.enclosure" ></p>
        <div class="item-description">
        <p class="item-description__title item-title"><a :href="item.link" target="_blank">{{ item.title }}</a></p>
        <p class="item-description__content item-content" v-if="item.content" v-html="item.content"></p>
        </div>
        </div>
        <div class="item-sources">
        <p class="item-sources__source source"><a :href="makeOriginLink(item.link)" target="_blank">{{ beautifySource(item.link) }}</a></p>
        <p class="item-sources__date date">{{ parseDate(item.pubDate) }}</p>
        </div>
      </li>
    </ul>
    <ul id="navigation" v-if="pagesCount > 1">
      <li class="pages" :class="{current: currentPage == page, last: (page == pagesCount && Math.abs(page - currentPage) > 4), first:(page == 1 && Math.abs(page - currentPage) > 4)}" v-for="page in pagesCount" :key="page" v-if="Math.abs(page - currentPage) < 4 || page == pagesCount || page == 1">
        <router-link :to="{ query: { ...$route.query, page: page } }" v-bind:style="{ color: blue }" replace>{{ page }}</router-link>
      </li>
    </ul>
  </section>
</template>
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
