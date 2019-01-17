<template>
  <footer>
    <div class="copy">
      <p>© 2019 <a href="/#/page/about">Jun</a> Powered By <a :href="system.website" target="_blank" rel="noopener noreferrer" v-text="system.brand"></a></p>
    </div>
    <div class="github">
      <a href="https://github.com/wnsrl56" target="_blank" rel="noopener noreferrer">Github</a>
    </div>
  </footer>
</template>

<script>
  export default {
    name: 'vuelog-footer',

    components: {
    },

    computed: {
      enableSwitch () {
        var switchLang = this.$store.getters.config.switchLang
        var count = Object.keys(this.$store.getters.languages).length
        return switchLang && (count > 1)
      },

      system () {
        return this.$store.getters.system
      }
    }
  }
</script>

<style lang="stylus" scoped>
  .github
    float right
    height 27px
    color #4fc08d
    padding 0px 4px
    &:hover
      color #5dc596

  .copy
    float left
    p
      display inline-block
      margin 0
  footer
    display inline-block
    border-top 1px solid #ddd
    color #7f8c8d
    font-size 13px
    font-weight 600
    width 900px
    padding 10px 0
    text-align center
    position relative
  span
    color #f66

  a:hover
    text-decoration none

  .lang
    position absolute
    right 10px
    top 20px

  @media screen and (max-width: 999px)
    footer
      padding 10px 20px
      width 100%

    div
      float left
    .lang
      position static
      float right
</style>
