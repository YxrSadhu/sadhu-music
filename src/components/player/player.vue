<template>
  <div class="player" v-show="playList.length > 0">
    <transition name="normal"
      @enter="enter"
      @after-enter="afterEnter"
      @leave="leave"
      @after-leave="afterLeave"
    >
      <div class="normal-player" v-show="fullScreen">
        <div class="background">
          <img width="100%" height="100%" :src="currentSong.image">
        </div>
        <div class="top">
          <div class="back" @click="back">
            <i class="icon-back"></i>
          </div>
          <h1 class="title" v-html="currentSong.name"></h1>
          <h2 class="subtitle" v-html="currentSong.singer"></h2>
        </div>
        <div class="middle">
          <div class="middle-l">
            <div class="cd-wrapper" ref="cdWrapper">
              <div class="cd" ref="imageWrapper">
                <img ref="image" :class="cdCls" class="image" :src="currentSong.image">
              </div>
            </div>
          </div>
        </div>
        <div class="bottom">
          <div class="operators">
            <div class="icon i-left">
              <i class="icon-sequence"></i>
            </div>
            <div class="icon i-left">
              <i class="icon-prev"></i>
            </div>
            <div class="icon i-center">
              <i :class="playIcon" @click="togglePlaying"></i>
            </div>
            <div class="icon i-right">
              <i class="icon-next"></i>
            </div>
            <div class="icon i-right">
              <i class="icon icon-not-favorite"></i>
            </div>
          </div>
        </div>
      </div>
    </transition>
    <transition name="mini">
      <div class="mini-player" @click="open" v-show="!fullScreen">
        <div class="icon">
          <div class="imgWrapper" ref="miniWrapper">
            <img ref="miniImage" :class="cdCls" width="40" height="40" :src="currentSong.image">
          </div>
        </div>
        <div class="text">
          <h2 class="name" v-html="currentSong.name"></h2>
          <p class="desc" v-html="currentSong.singer"></p>
        </div>
        <div class="control">
          <i @click.stop="togglePlaying" class="icon-mini" :class="miniIcon"></i>
        </div>
        <div class="control">
          <i class="icon-playlist"></i>
        </div>
      </div>
    </transition>
    <audio ref="audio" :src="currentSong.url"></audio>
  </div>
</template>

<script>
import { mapGetters, mapMutations } from 'vuex'
import animations from 'create-keyframe-animation' // 第三方库，用来让js创建css3动画
import { prefixStyle } from 'common/js/dom'

const transform = prefixStyle('transform')

export default {
  methods: {
    back () {
      this.setFullScreen(false)
    },
    open () {
      this.setFullScreen(true)
    },
    ...mapMutations({
      setFullScreen: 'SET_FULL_SCREEN',
      setPlayingState: 'SET_PLAYING_STATE'
    }),
    enter (el, done) { // 动画执行完了就执行 done 函数， done 回调函数执行的时候，进入到下一个钩子函数
      const { x, y, scale } = this._getPosAndScale()

      let animation = {
        0: { // 刚开始
          transform: `translate3d(${x}px, ${y}px, 0) scale(${scale})`
        },
        60: { // 60% 时 (位置到了 and 放大到了1.1)
          transform: `translate3d(0, 0, 0) scale(1.1)`
        },
        100: { // 大小回弹到 1
          transform: `translate3d(0, 0, 0) scale(1)`
        }
      }

      animations.registerAnimation({
        name: 'move', // 动画名称
        animation, // 动画
        presets: {
          duration: 400, // ms
          easing: 'linear' // 线性移动
        }
      })

      animations.runAnimation(this.$refs.cdWrapper, 'move', done) // 第一个参数是需要作用的 DOM
    },
    afterEnter () {
      animations.unregisterAnimation('move')
      this.$refs.cdWrapper.style.animation = ''
    },
    leave (el, done) {
      this.$refs.cdWrapper.style.transition = 'all 0.4s'
      const { x, y, scale } = this._getPosAndScale()
      this.$refs.cdWrapper.style[transform] = `translate3d(${x}px, ${y}px, 0) scale(${scale})`
      this.$refs.cdWrapper.addEventListener('transitionend', done) // 卧槽，transition结束了竟然还会触发个 transitionend 事件，涨知识了。。。
    },
    afterLeave () {
      this.$refs.cdWrapper.style.transition = ''
      this.$refs.cdWrapper.style[transform] = ''
    },
    _getPosAndScale () {
      const targetWidth = 40 // mini img 的宽度（左下角那个小小的cd）
      const paddingLeft = 40 // 其中心点离左边偏移40px高度
      const paddingBottom = 30
      const paddingTop = 80 // 这是 normal cd 举例顶部的宽度
      const width = window.innerWidth * 0.8 // 这是 normal cd 的宽度(圆形高宽一样都是半径)
      const scale = targetWidth / width // 初始缩放比例
      const x = -(window.innerWidth / 2 - paddingLeft) // mini 相对 normal cd的 x轴距离
      const y = window.innerHeight - paddingBottom - paddingTop - width / 2
      return {
        x,
        y,
        scale
      }
    },
    togglePlaying () {
      this.setPlayingState(!this.playing)
    },
    /**
     * 计算内层Image的transform，并同步到外层容器，这样在暂停的时候，cd 就会停在原本 rotate 的位置
     * @param wrapper
     * @param inner
     */
    syncWrapperTransform (wrapper, inner) {
      if (!this.$refs[wrapper]) {
        return
      }
      let imageWrapper = this.$refs[wrapper]
      let image = this.$refs[inner]
      let wTransform = getComputedStyle(imageWrapper)[transform]
      let iTransform = getComputedStyle(image)[transform]
      imageWrapper.style[transform] = wTransform === 'none' ? iTransform : iTransform.concat(' ', wTransform)
    }
  },
  computed: {
    playIcon () {
      return this.playing ? 'icon-pause' : 'icon-play'
    },
    miniIcon () {
      return this.playing ? 'icon-pause-mini' : 'icon-play-mini'
    },
    cdCls () {
      return this.playing ? 'play' : ''
    },
    ...mapGetters([
      'fullScreen',
      'playList',
      'currentSong',
      'playing'
    ])
  },
  watch: {
    currentSong () {
      this.$nextTick(() => {
        this.$refs.audio.play() // 不能发生变化了就马上调用 audio 的 play，要等 audio 加载完歌曲后，所以用个 nextTick 延时
      })
    },
    playing (newState) {
      const audio = this.$refs.audio
      this.$nextTick(() => {
        newState ? audio.play() : audio.pause()
      })

      if (!newState) {
        if (this.fullScreen) {
          this.syncWrapperTransform('imageWrapper', 'image')
        } else {
          this.syncWrapperTransform('miniWrapper', 'miniImage')
        }
      }
    }
  }
}
</script>

<style lang="stylus" scoped>
  @import '~common/stylus/variable'
  @import '~common/stylus/mixins'

  .player
    .normal-player
      position fixed
      left 0
      right 0
      top 0
      bottom 0
      z-index 150
      background $color-background
      .background
        position absolute
        left 0
        top 0
        width 100%
        height 100%
        z-index -1
        opacity 0.6
        filter blur(20px)
      .top
        position relative
        margin-bottom 25px
        .back
          position absolute
          top 0
          left 6px
          z-index 50
          .icon-back
            display block
            padding 9px
            font-size $font-size-large-x
            color $color-theme
            transform rotate(-90deg)
        .title
          width 70%
          margin 0 auto
          height 40px
          line-height 40px
          text-align center
          no-wrap()
          font-size $font-size-large
          color $color-text
        .subtitle
          height 20px
          line-height 20px
          text-align center
          font-size $font-size-medium
          color $color-text
      .middle
        position fixed
        width 100%
        top 80px
        bottom 170px
        white-space nowrap
        font-size 0
        .middle-l
          display inline-block
          vertical-align top
          position relative
          width 100%
          height 0
          padding-top 80%
          .cd-wrapper
            position absolute
            left 10%
            top 0
            width 80%
            box-sizing border-box
            height 100%
            .cd
              width 100%
              height 100%
              border-radius 50%
              .image
                position absolute
                left 0
                right 0
                width 100%
                height 100%
                box-sizing border-box
                border-radius 50%
                border 10px solid rgba(255, 255, 255, 0.1)
                &.play
                  animation rotate 10s linear infinite
      .bottom
        position absolute
        bottom 50px
        width 100%
        .operators
          display flex
          align-items center
          .icon
            flex 1
            color $color-theme
            i
              font-size 30px
          .i-left
            text-align right
          .i-center
            text-align center
            padding 0 20px
            i
              font-size 40px
          .i-right
            text-align left
      &.normal-enter-active, &.normal-leave-active
        transition all 0.4s
        .top, .bottom
          transition all 0.4s cubic-bezier(0.86, 0.18, 0.82, 1.32)
      &.normal-enter, &.normal-leave-to
        opacity 0
        .top
          transform translate3d(0, -100px, 0)
        .bottom
          transform translate3d(0, 100px, 0)
    .mini-player
      display flex
      align-items center
      position fixed
      left 0
      bottom 0
      z-index 180
      width 100%
      height 60px
      background $color-highlight-background
      &.mini-enter-active, &.mini-leave-active
        transition all 0.4s
      &.mini-enter, &.mini-leave-to
        opacity 0
      .icon
        flex 0 0 40px
        width 40px
        height 40px
        padding 0 10px 0 20px
        .imgWrapper
          width 100%
          height 100%
          img
            border-radius 50%
            &.play
              animation rotate 10s linear infinite
      .text
        display flex
        flex-direction column
        justify-content center
        flex 1
        line-height 20px
        overflow hidden
        .name
          margin-bottom 2px
          no-wrap()
          font-size $font-size-medium
          color $color-text
        .desc
          no-wrap()
          font-size $font-size-small
          color $color-text-d
      .control
        flex 0 0 30px
        width 30px
        padding 0 10px
        .icon-play-mini, .icon-pause-mini, .icon-playlist
          font-size: 30px
          color: $color-theme-d
        .icon-mini
          font-size: 32px
  @keyframes rotate
    0%
      transform rotate(0)
    100%
      transform rotate(360deg)
</style>