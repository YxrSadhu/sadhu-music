<template>
  <div class="tab">
    <router-link tag="div" class="item" to="/recommend">
      <span class="link">推荐</span>
    </router-link>
    <router-link tag="div" class="item" to="/singer">
      <span class="link">歌手</span>
    </router-link>
    <router-link tag="div" class="item" to="/rank">
      <span class="link">榜单</span>
    </router-link>
  </div>
</template>

<script>
export default {
}
</script>

<style lang="stylus" scoped>
@import "~common/stylus/variable"
  .tab
    display flex
    height 44px
    line-height 44px
    font-size $font-size-medium
    background $color-theme
    .item
      flex 1
      text-align center
      color $color-text-lm
      .link
        padding 5px 0
    .router-link-active
      color $color-text-l
      .link
        font-weight bold
        border-bottom 2px solid $color-text-l
</style>
