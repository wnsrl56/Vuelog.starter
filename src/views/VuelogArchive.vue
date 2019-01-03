<template>
  <div class="archive">
    <div v-if="displayType === 'archive-category'" class="archive-body">
      <i18n path="archive.inCategory" tag="h2">
        <q v-text="i18nify(archive.title)"></q>
      </i18n>
      <ul>
        <li v-for="(post, index) in archive.posts" :key="index">
          <router-link :to="{name: 'post', params: {category: post.category, slug: post.slug, year: post.year}}" v-text="i18nify(post.title)"></router-link>
          <span v-text="'- ' + convertDateFormat(post.date)"></span>
        </li>
        <li v-if="archive.posts.length === 0" v-text="$t('archive.empty')"></li>
      </ul>
    </div>

    <div v-if="displayType === 'archive-year'" class="archive-body">
      <h2 v-text="$t('archive.inYear', [archive.year])"></h2>
      <ul>
        <li v-for="(post, index) in archive.posts" :key="index">
          <router-link :to="{name: 'post', params: {category: post.category, slug: post.slug, year: archive.year}}" v-text="i18nify(post.title)"></router-link>
          <span class="tag-button" v-text="i18nify(post.categoryTitle)" />
        </li>
        <li v-if="archive.posts.length === 0" v-text="$t('archive.empty')"></li>
      </ul>
    </div>

    <div v-if="displayType === 'archive'" class="archive-body">
      <h2 v-text="$t('archive.byCategory').replace(':', '')"></h2>
      <ul>
        <li v-for="(category, index) in archive.postsByCategory" :key="index">
          <h4>
            <router-link :to="{name: 'archive-category', params: {category: category.slug}}" v-text="i18nify(category.title)"></router-link>
            <span class="posts-number" v-text="category.posts.length"></span>
          </h4>
          <ul>
            <li v-for="(post, index) in category.posts" :key="index">
              <router-link :to="{name: 'post', params: {category: post.category, slug: post.slug, year: post.year}}" v-text="i18nify(post.title)"></router-link>
              <span class="show-date" v-text="'- ' + convertDateFormat(post.date)"></span>
            </li>
            <li v-if="category.posts.length === 0" v-text="$t('archive.empty')"></li>
          </ul>
        </li>
      </ul>

      <h2 v-text="$t('archive.byYear').replace(':', '')"></h2>
      <ul>
        <li v-for="(year, index) in archive.postsByYear" :key="index">
          <h4>
            <router-link :to="{name: 'archive-year', params: {year: year.year}}" v-text="year.year"></router-link>
            <span class="posts-number" v-text="year.posts.length"></span>
          </h4>
          <ul>
            <li v-for="(post, index) in year.posts" :key="index">
              <router-link :to="{name: 'post', params: {category: post.category, slug: post.slug, year: post.year}}" v-text="i18nify(post.title)"></router-link>
              <span class="tag-button" v-text="i18nify(post.categoryTitle)" />
            </li>
          </ul>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
  import { retrieveByLanguage } from '../utils'

  export default {
    name: 'vuelog-archive',

    computed: {
      displayType () {
        return this.$route.name
      },

      active () {
        return this.$store.getters.lang
      },

      config () {
        return this.$store.getters.config
      },

      postsByCategory () {
        return this.$store.getters.postsByCategory
      },

      postsByYear () {
        return this.$store.getters.postsByYear
      },

      pages () {
        return this.$store.getters.pages
      },

      archive () {
        switch (this.displayType) {
          case 'archive-category':
            return this.getPostsInCategory(this.$route.params.category)
          case 'archive-year':
            return this.getPostsInYear(+this.$route.params.year)
          case 'archive':
            return this.getAllPostsAndPages()
        }
      },

      title () {
        var title = this.$t('archive.title')
        var brand = retrieveByLanguage(this.config.brand, this.active, this.config.defaultLang)
        if (this.displayType === 'archive-category') {
          title += ' | ' + retrieveByLanguage(this.archive.title, this.active, this.config.defaultLang)
        }
        if (this.displayType === 'archive-year') {
          title += ` | ${this.archive.year}`
        }
        if (this.config.brandTrailing) {
          return title + ' | ' + brand
        } else {
          return brand + ' | ' + title
        }
      }
    },

    methods: {
      oops () {
        this.$router.replace('/oops')
      },

      i18nify (content) {
        return retrieveByLanguage(content, this.active, this.config.defaultLang)
      },

      getPostsInCategory (slug) {
        for (var i = 0; i < this.postsByCategory.length; i++) {
          if (this.postsByCategory[i].slug === slug) {
            return this.postsByCategory[i]
          }
        }
        this.oops()
        return { posts: [], slug: '', title: '' }
      },

      getPostsInYear (year) {
        if (Number.isNaN(year)) {
          this.oops()
          return { posts: [], year }
        }
        for (var i = 0; i < this.postsByYear.length; i++) {
          if (this.postsByYear[i].year === year) {
            return this.postsByYear[i]
          }
        }
        return { posts: [], year }
      },

      getAllPostsAndPages () {
        return {
          postsByCategory: this.postsByCategory,
          postsByYear: this.postsByYear,
          pages: this.pages
        }
      },
      map (iter, fn) {
        var res = []
        var i = -1

        while (++i < iter.length) {
          var item = iter[i]

          res.push(fn(item))
        }

        return res.join('')
      },
      convertDateFormat (date) {
        return date && date.length ? this.map(date, function (v) { return v.replace('-', '/') }) : date
      },
    },

    created () {
      this.$store.dispatch('documentTitle', this.title)
    },

    watch: {
      $route (to, from) {
        if (to.query.lang !== from.query.lang) {
          this.$store.dispatch('documentTitle', this.title)
        }
      }
    }
  }
</script>

<style lang="stylus" scoped>
  .show-date
    font-size 13px
  .tag-button
    height 24px
    padding 2px 6px
    margin 2px
    font-size 11px
    color #ffffff
    background-color #209cee
    border-radius 4px
  .posts-number
    padding 0px 4px 1px 4px
    font-size 11px
    color #ffffff
    background-color #2B3D4F
    border-radius 4px

  .archive-enter-active
  .archive-leave-active
    transition opacity .3s ease

  .archive-enter
  .archive-leave-active
    opacity 0

  ul
    line-height 1.6em
    padding-left 1.6em

  h4
  span
    color #7f8c8d

  @media screen and (max-width: 999px)
    h4
      margin-bottom .5em

    ul
      padding-left 1em

      ul
        padding-left .75em
</style>
