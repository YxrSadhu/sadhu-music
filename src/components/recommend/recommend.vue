<template>
  <div class="recommend-wrapper" ref="recommendWrapper">
    <scroll ref="scroll" class="recommend-content" :data="lists">
      <div class="test-two" v-show="lists.length">
        <div class="decorate"></div>
        <div v-if="sliders.length" class="slide-wrapper">
          <swiper :options="swiperOption">
              <swiper-slide v-for="item in sliders" :key="item.id">
                <a :href="item.linkUrl"><img :src="item.picUrl"></a>
              </swiper-slide>
              <div class="swiper-pagination" slot="pagination"></div>
            </swiper>
        </div>
        <div class="recommend-list">
          <h1 class="list-title">推荐歌单</h1>
          <ul>
            <li
              v-for="(item, index) in lists"
              :key="item.dissid"
              class="item-list"
              @click="selectItem(item)"
            >
              <div class="avatar">
                <img @load="loadImage(index)" class="image" v-lazy="item.imgurl">
              </div>
              <div class="text">
                <h2 class="name">{{item.dissname}}</h2>
                <p class="desc">{{item.creator.name}}</p>
              </div>
            </li>
          </ul>
        </div>
      </div>
      <div class="loading-container" v-if="!lists.length">
          <loading></loading>
      </div>
    </scroll>
    <router-view></router-view>
  </div>
</template>

<script>
import Scroll from 'base/scroll/scroll'
import Loading from 'base/loading/loading'
import { getSlider, getList } from 'api/recommend'
import { playlistMixin } from 'common/js/mixin'
import { mapMutations } from 'vuex'

// let TIMES = 0 测试 onload

export default {
  mixins: [playlistMixin],
  created () {
    this._getSlider()
    this._getList()
  },
  data () {
    return {
      sliders: [],
      lists: [],
      swiperOption: {
        pagination: {
          el: '.swiper-pagination',
          dynamicBullets: true
        },
        loop: true,
        autoplay: {
          delay: 3000,
          stopOnLastSlide: false,
          disableOnInteraction: false
        }
      }
    }
  },
  methods: {
    handlePlayList (playlist) {
      const bottom = playlist.length > 0 ? '60px' : ''
      this.$refs.recommendWrapper.style.bottom = bottom
      this.$refs.scroll.refresh()
    },
    selectItem (item) {
      this.$router.push({
        path: `/recommend/${item.dissid}`
      })
      this.setDisc(item)
    },
    _getList () {
      getList().then(res => {
        this.lists = res.data.list
      })
    },
    _getSlider () {
      getSlider().then(res => {
        this.sliders = res.data.slider
      })
    },
    loadImage (index) {
      // if (!this.checkloaded) {
      //   this.checkloaded = true
      //   setTimeout(() => {
      //     this.$refs.scroll.refresh()
      //   }, 20)
      // } // 第一张图片加载完后，下一轮tick(20ms后)，后面的图片竟然还没加载完。。。
      if (index === this.lists.length - 1) {
        setTimeout(() => {
          // TIMES++
          if (this.$refs.scroll) {
            this.$refs.scroll.refresh()
            // console.log(TIMES)
          }
        }, 20)
      }
    },
    ...mapMutations({
      setDisc: 'SET_DISC'
    })
  },
  components: {
    Scroll,
    Loading
  }
}
</script>

<style lang="stylus" scoped>
@import "~common/stylus/variable.styl"

.recommend-wrapper
  position fixed
  top 88px
  bottom 0
  left 0
  right 0
  // overflow: hidden
  .recommend-content
    position relative
    height 100% // 继承了 wrapper
    overflow: hidden
    .decorate
      position absolute
      top -34vh
      z-index -10
      background $color-theme
      width 100%
      height 50vh
      overflow hidden
    .slide-wrapper >>> .swiper-pagination-bullet
      background rgb(255, 255, 255)
      opacity 0.6
    .slide-wrapper >>> .swiper-pagination-bullet-active
      background rgb(255, 255, 255)
      opacity 0.9
    .slide-wrapper
      overflow hidden
      width 96%
      height 0
      padding-bottom 40%
      margin 0 auto
      border-radius 5px
      img
        width 100%
    .recommend-list
      width 100%
      .list-title
        height 65px
        line-height 65px
        text-align left
        padding-left 20px
        color $color-text
        font-weight bold
        font-size 14px
      .item-list
        display flex
        box-sizing: border-box
        align-items: center
        padding: 0 20px 20px 20px
        .avatar
          // height: 60px
          flex: 0 0 60px
          padding-right 20px
          .image
            width 100%
        .text
          display: flex
          flex-direction: column
          justify-content: center
          flex: 1
          line-height: 20px
          overflow: hidden
          font-size: $font-size-medium
          .name
            margin-bottom 10px
            font-size: $font-size-medium-x
            color: $color-text
          .desc
            color $color-text-g
    .loading-container
      position absolute
      width 100%
      top: 50%
      transform translateY(-50%)
</style>
