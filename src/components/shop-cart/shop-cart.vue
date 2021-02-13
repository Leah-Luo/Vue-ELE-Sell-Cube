<!--Vue 提供了 transition 的封装组件，某些情形中可以给任何元素和组件添加进入/离开过渡-->
<template>
  <div>
    <div class="shopcart">
      <div class="content" @click="toggleList">
        <div class="content-left">
          <div class="logo-wrapper">
            <div class="logo" :class="{'highlight':totalCount>0}">
              <i class="icon-shopping_cart" :class="{'highlight':totalCount>0}"></i>
            </div>
            <div class="num" v-show="totalCount>0">
              <bubble :num="totalCount"></bubble>
            </div>
          </div>
          <div class="price" :class="{'highlight':totalPrice>0}">￥{{totalPrice}}</div>
          <div class="desc">另需配送费￥{{deliveryPrice}}元</div>
        </div>
        <div class="content-right" @click="pay">
          <div class="pay" :class="payClass">
            {{payDesc}}
          </div>
        </div>
      </div>
      <div class="ball-container">
        <div v-for="(ball,index) in balls" :key="index">
          <transition
            @before-enter="beforeDrop"
            @enter="dropping"
            @after-enter="afterDrop">
            <div class="ball" v-show="ball.show">
              <div class="inner inner-hook"></div>
            </div>
          </transition>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Bubble from 'components/bubble/bubble'

const BALL_LEN = 10
const innerClsHook = 'inner-hook'

// 小球动画思路：
// 这里需要重绘的原因是：小球本身的位置在购物框中，为了实现从加号位置抛物线下落的效果，先把小球移动到了加号位置，
// 然后再按照一定缓动的动画移回购物车原始位置，因为小球整个动画是先从购物车移走再回购物车，整个过程是在一个 tick 内完成的，如果中途不强制重绘，浏览器会认为这个动画是无效的，就不会产生动画
// 如果一个元素同时做横向和纵向动画，那么叠加出来的效果是出不来抛物线的
// 假设购物车为篮子，里面已有10个小球，状态为不显示(show false)
function createBalls() {
  let balls = []
  for (let i = 0; i < BALL_LEN; i++) {
    balls.push({ show: false })
  }
  return balls
}

export default {
  name: 'shop-cart',
  props: {
    selectFoods: {
      type: Array,
      default() {
        return []
      }
    },
    deliveryPrice: {
      type: Number,
      default: 0
    },
    minPrice: {
      type: Number,
      default: 0
    },
    sticky: {
      type: Boolean,
      default: false
    },
    fold: {
      type: Boolean,
      default: true
    }
  },
  data() {
    return {
      balls: createBalls(),
      listFold: this.fold
    }
  },
  created() {
    this.dropBalls = []
  },
  computed: {
    totalPrice() {
      let total = 0
      this.selectFoods.forEach((food) => {
        total += food.price * food.count
      })
      return total
    },
    totalCount() {
      let count = 0
      this.selectFoods.forEach((food) => {
        count += food.count
      })
      return count
    },
    payDesc() {
      if (this.totalPrice === 0) {
        return `￥${this.minPrice}元起送`
      } else if (this.totalPrice < this.minPrice) {
        let diff = this.minPrice - this.totalPrice
        return `还差￥${diff}元起送`
      } else {
        return '去结算'
      }
    },
    payClass() {
      if (!this.totalCount || this.totalPrice < this.minPrice) {
        return 'not-enough'
      } else {
        return 'enough'
      }
    }
  },
  // 列表收起 展开
  // listFold 默认为false （不展开）
  methods: {
    toggleList() {
      if (this.listFold) {
        if (!this.totalCount) {
          return
        }
        this.listFold = false
        this._showShopCartList()
        this._showShopCartSticky()
      } else {
        this.listFold = true
        this._hideShopCartList()
      }
    },
    pay(e) {
      if (this.totalPrice < this.minPrice) {
        return
      }
      this.$createDialog({
        title: '支付',
        content: `您需要支付${this.totalPrice}元`
      }).show()
      e.stopPropagation()
    },
    // 接收参数el: 按钮位置
    // drop 函数是从 balls 中找到第一个 show 为 false 的 ball，放到 dropBalls 里然后就 return 了，并没有显示所有球
    drop(el) {
      for (let i = 0; i < this.balls.length; i++) {
        const ball = this.balls[i]
        if (!ball.show) {
          ball.show = true
          ball.el = el
          this.dropBalls.push(ball)
          return
        }
      }
    },
    // 当前正在做动画的一个小球元素
    // beforeDrop 的时候，el.style.display 为 none，需要显示的把它设置为 ''
    beforeDrop(el) {
      const ball = this.dropBalls[this.dropBalls.length - 1]
      const rect = ball.el.getBoundingClientRect()
      const x = rect.left - 32
      const y = -(window.innerHeight - rect.top - 22)
      el.style.display = ''
      // beforeDrop 函数里设置 transform 就是把小球从原本购物车的位置移动到按钮位置
      el.style.transform = el.style.webkitTransform = `translate3d(0,${y}px,0)`
      const inner = el.getElementsByClassName(innerClsHook)[0]
      inner.style.transform = inner.style.webkitTransform = `translate3d(${x}px,0,0)`
    },
    // 执行完成后会执行done
    dropping(el, done) {
      // 页面重绘
      this._reflow = document.body.offsetHeight
      // 在 dropping 函数就是把小球从按钮位置移动到购物车位置
      el.style.transform = el.style.webkitTransform = 'translate3d(0,0,0)'
      const inner = el.getElementsByClassName(innerClsHook)[0]
      inner.style.transform = inner.style.webkitTransform = 'translate3d(0,0,0)'
      el.addEventListener('transitionend', done)
    },
    afterDrop(el) {
      const ball = this.dropBalls.shift()
      if (ball) {
        ball.show = false
        el.style.display = 'none'
      }
    },
    _showShopCartList() {
      this.shopCartListComp = this.shopCartListComp || this.$createShopCartList({
        $props: {
          selectFoods: 'selectFoods'
        },
        $events: {
          leave: () => {
            this._hideShopCartSticky()
          },
          hide: () => {
            this.listFold = true
          },
          add: (el) => {
            this.shopCartStickyComp.drop(el)
          }
        }
      })
      this.shopCartListComp.show()
    },
    _showShopCartSticky() {
      this.shopCartStickyComp = this.shopCartStickyComp || this.$createShopCartSticky({
        $props: {
          selectFoods: 'selectFoods',
          deliveryPrice: 'deliveryPrice',
          minPrice: 'minPrice',
          fold: 'listFold',
          list: this.shopCartListComp
        }
      })
      this.shopCartStickyComp.show()
    },
    _hideShopCartList() {
      const list = this.sticky ? this.$parent.list : this.shopCartListComp
      list.hide && list.hide()
    },
    _hideShopCartSticky() {
      this.shopCartStickyComp.hide()
    }
  },
  watch: {
    fold(newVal) {
      this.listFold = newVal
    },
    totalCount(count) {
      if (!this.fold && count === 0) {
        this._hideShopCartList()
      }
    }
  },
  components: {
    Bubble
  }
}
</script>

<style lang="stylus" scoped>
@import "~common/stylus/mixin"
@import "~common/stylus/variable"

.shopcart
  height: 100%
  .content
    display: flex
    background: $color-background
    font-size: 0
    color: $color-light-grey
    .content-left
      flex: 1
      .logo-wrapper
        display: inline-block
        vertical-align: top
        position: relative
        top: -10px
        margin: 0 12px
        padding: 6px
        width: 56px
        height: 56px
        box-sizing: border-box
        border-radius: 50%
        background: $color-background
        .logo
          width: 100%
          height: 100%
          border-radius: 50%
          text-align: center
          background: $color-dark-grey
          &.highlight
            background: $color-blue
          .icon-shopping_cart
            line-height: 44px
            font-size: $fontsize-large-xxx
            color: $color-light-grey
            &.highlight
              color: $color-white
        .num
          position: absolute
          top: 0
          right: 0
      .price
        display: inline-block
        vertical-align: top
        margin-top: 12px
        line-height: 24px
        padding-right: 12px
        box-sizing: border-box
        border-right: 1px solid rgba(255, 255, 255, 0.1)
        font-weight: 700
        font-size: $fontsize-large
        &.highlight
          color: $color-white
      .desc
        display: inline-block
        vertical-align: top
        margin: 12px 0 0 12px
        line-height: 24px
        font-size: $fontsize-small-s
    .content-right
      flex: 0 0 105px
      width: 105px
      .pay
        height: 48px
        line-height: 48px
        text-align: center
        font-weight: 700
        font-size: $fontsize-small
        &.not-enough
          background: $color-dark-grey
        &.enough
          background: $color-green
          color: $color-white
  .ball-container
    .ball
      position: fixed
      left: 32px
      bottom: 22px
      z-index: 200
      transition: all 0.4s cubic-bezier(0.49, -0.29, 0.75, 0.41)
      .inner
        width: 16px
        height: 16px
        border-radius: 50%
        background: $color-blue
        transition: all 0.4s linear
</style>
