<template>
  <div class="header" @click="showDetails">
    <div class="content-wrapper">
      <div class="avatar">
        <img width="64" height="64" :src="seller.avatar">
      </div>
      <div class="content">
        <div class="title">
          <span class="brand"></span>
          <span class="name">{{ seller.name }}</span>
        </div>
        <div class="description">
          {{ seller.description }}/{{ seller.deliveryTime }}分钟送达
        </div>
        <div v-if="seller.supports" class="support">
          <support-ico :size=1 :type="seller.supports[0].type"></support-ico>
<!--          "text: 在线支付满减。。"-->
          <span class="text">{{ seller.supports[0].description }}</span>
        </div>
      </div>
      <div v-if="seller.supports" class="support-count">
        <span class="count">{{ seller.supports.length }}个</span>
        <i class="icon-keyboard_arrow_right"></i>
      </div>
    </div>
    <div class="bulletin-wrapper">
      <span class="bulletin-title"></span><span class="bulletin-text">{{ seller.bulletin }}</span>
      <i class="icon-keyboard_arrow_right"></i>
    </div>
    <div class="background">
      <img :src="seller.avatar" width="100%" height="100%">
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
import SupportIco from '@/components/support-ico/support-ico'

export default {
  name: 'v-header',
  props: {
    seller: {
      type: Object,
      default () {
        return {}
      }
    }
  },
  methods: {
    showDetails () {
      this.headerDetailComp = this.headerDetailComp || this.$createHeaderDetail({
        $props: {
          seller: 'seller'
        }
      })
      this.headerDetailComp.show()
    }
  },
  components: { SupportIco }
}
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "~common/stylus/mixin"
  @import "~common/stylus/variable"

  .header
    position: relative
    color: $color-white
    overflow hidden
    background-color $color-background-ss
    .content-wrapper
      position: relative
      display: flex
      align-items : center
      padding: 24px 12px 18px 24px
      .avatar
        display : inline-block
        // 和右边顶部对齐
        vertical-align top
        flex : 0 0 64px
        width : 64px
        margin-right : 16px
        img
          border-radius : 2px
      .content
        display: inline-block
        flex : 1
        .title
          margin: 2px 0 8px 0
          .brand
            // span 要设置inline block, 否则宽高显示不出
            display : inline-block
            // 和name水平对齐
            vertical-align top
            width : 30px
            height : 18px
            // bg-image： 在mixin里设置2x和3x
            bg-image('brand')
            background-size : 30px 18px
            // background-repeat CSS 属性定义背景图像的重复方式。背景图像可以沿着水平轴，垂直轴，两个轴重复，或者根本不重复
            background-repeat : no-repeat
          .name
            font-size: 16px
            font-weight bold
            line-height 18px
            margin-left 6px
        .description
          margin-bottom 10px
          font-size 12px
          line-height 12px
        .support
          display flex
          .support-ico
            display inline-block
            vertical-align top
            width 12px
            height 12px
            margin-right 4px
            backdrop-size 12px
            background-repeat no-repeat
          .text
            font-height 12px
            font-size $fontsize-small-s

      .support-count
        position absolute
        right 12px
        height 24px
        display flex
        bottom 14px
        padding 0 8px
        line-height 24px
        border-radius 14px
        background rgba(0,0,0,0.2)
        text-align center
        .count
          font-size 10px
        .icon-keyboard_arrow_right
          margin-left 2px
          line-height 24px
          font-size 10px
    .bulletin-wrapper
      position relative
      display flex
      align-items center
      height 28px
      line-height 28px
      padding 0 8px
      background-color $color-background-sss
      .bulletin-title
        flex 0 0 22px
        display inline-block
        width 22px
        height 12px
        margin-right 4px
        bg-image('bulletin')
        background-size 22px
        background-repeat no-repeat
      .bulletin-text
        flex 1
        white-space nowrap
        overflow hidden
        text-overflow ellipsis
        font-size 10px
      .icon-keyboard_arrow_right
        font-size 10px
        margin-left 4px
        width 10px
    .background
      position absolute
      top 0
      left 0
      width 100%
      height 100%
      z-index -1
      filter blur(10px)
</style>
