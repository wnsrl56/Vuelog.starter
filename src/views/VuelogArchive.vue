<template>
  <div class="archive">
    <div
      v-if="displayType === 'archive-category'"
      class="archive-body">
      <h2 v-text="`Post in category \u{201D} ${i18nify(archive.title)} \u{201D}`" />
      <ul>
        <li v-for="(post, index) in archive.posts" :key="index">
          <router-link
            :to="{name: 'post', params: {category: post.category, slug: post.slug, year: post.year}}"
            v-text="i18nify(post.title)"
          />
          <span
            v-text="`- ${convertDateFormat(post.date, v => v.replace('-', '/'))}`"
          />
        </li>
        <li
          v-if="archive.posts.length === 0"
          v-text="$t('archive.empty')"
        />
      </ul>
    </div>

    <div v-if="displayType === 'archive-year'" class="archive-body">
      <h2 v-text="`Posts in year ${archive.year}`" />
      <ul>
        <li v-for="(post, index) in archive.posts" :key="index">
          <router-link
            :to="{name: 'post', params: {category: post.category, slug: post.slug, year: archive.year}}"
            v-text="i18nify(post.title)"
          />
          <span
            class="tag-button"
            v-text="i18nify(post.categoryTitle)"
          />
        </li>
        <li
          v-if="archive.posts.length === 0"
          v-text="$t('archive.empty')"
        />
      </ul>
    </div>

    <div v-if="displayType === 'archive'" class="archive-body">
      <h2 v-text="$t('archive.byCategory')" />
      <ul>
        <li v-for="(category, index) in archive.postsByCategory" :key="index">
          <h4>
            <router-link
              :to="{name: 'archive-category', params: {category: category.slug}}"
              v-text="i18nify(category.title)"
            />
            <span
              class="posts-number"
              v-text="category.posts.length"
            />
          </h4>
          <ul>
            <li v-for="(post, index) in category.posts" :key="index">
              <router-link
                :to="{name: 'post', params: {category: post.category, slug: post.slug, year: post.year}}"
                v-text="i18nify(post.title)"
              />
              <span
                class="show-date"
                v-text="`- ${convertDateFormat(post.date, v => v.replace('-', '/'))}`"
              />
            </li>
            <li
              v-if="category.posts.length === 0"
              v-text="$t('archive.empty')"
            />
          </ul>
        </li>
      </ul>

      <h2 v-text="$t('archive.byYear')" />
      <ul>
        <li v-for="(year, index) in archive.postsByYear" :key="index">
          <h4>
            <router-link
              :to="{name: 'archive-year', params: {year: year.year}}"
              v-text="year.year"
            />
            <span
              class="posts-number"
              v-text="year.posts.length"
            />
          </h4>
          <ul>
            <li v-for="(post, index) in year.posts" :key="index">
              <router-link
                :to="{name: 'post', params: {category: post.category, slug: post.slug, year: post.year}}"
                v-text="i18nify(post.title)"
              />
              <span
                class="tag-button"
                v-text="i18nify(post.categoryTitle)"
              />
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
        let title = this.$t('archive.title')
        const brand = retrieveByLanguage(this.config.brand, this.active, this.config.defaultLang)
        if (this.displayType === 'archive-category') {
          title = `${title} | ${retrieveByLanguage(this.archive.title, this.active, this.config.defaultLang)}`
        }
        if (this.displayType === 'archive-year') {
          title = `${title} | ${this.archive.year}`
        }
        return this.config.brandTrailing ? `${title} | ${brand}` : `${brand} | ${title}`
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
        for (let i = 0; i < this.postsByCategory.length; i++) {
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
        for (let i = 0; i < this.postsByYear.length; i++) {
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
        let res = []
        let i = -1

        while (++i < iter.length) {
          let item = iter[i]

          res.push(fn(item))
        }

        return res
      },
      convertDateFormat (date, fn) {
        return date && date.length ? this.map(date, fn).join('') : date
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
