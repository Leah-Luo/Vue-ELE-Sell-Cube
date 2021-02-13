<template>
  <div id="app" @touchmove.prevent>
    <v-header :seller="seller"></v-header>
    <div class="tab-wrapper">
      <tab :tabs="tabs" :initialIndex=0></tab>
    </div>
  </div>
</template>

<script>
  import qs from 'query-string'
  import VHeader from '@/components/v-header/v-header'
  import Goods from 'components/goods/goods'
  import { getSeller } from 'api'
  import Seller from 'components/seller/seller'
  import Ratings from 'components/ratings/ratings'
  import Tab from 'components/tab/tab'

  export default {
    // 数据获取
    // 真实情况需要seller的ID，因为有很多商家
    data() {
      return {
        seller: {
          id: qs.parse(location.search).id
        }
      }
    },
    computed: {
      tabs() {
        return [
          {
            label: '商品',
            component: Goods,
            data: {
              seller: this.seller
            }
          },
          {
            label: '评论',
            component: Ratings,
            data: {
              seller: this.seller
            }
          },
          {
            label: '商家',
            component: Seller,
            data: {
              seller: this.seller
            }
          }
        ]
      }
    },
    created () {
      // 真实情况需要seller的ID，因为有很多商家 this._getSeller()
      // 把获取的seller传回给v-header
      getSeller().then((seller) => {
        this.seller = seller
      })
    },
    methods: {
      _getSeller() {
        getSeller({
          id: this.seller.id
        }).then((seller) => {
          this.seller = Object.assign({}, this.seller, seller)
        })
      }
    },
    components: {
      Tab,
      VHeader
    }
  }
</script>

<style lang="stylus" scoped>
#app
  .tab-wrapper
    position: fixed
    top: 136px
    left: 0
    right: 0
    bottom: 0
  //font-family Avenir, Helvetica, Arial, sans-serif
  //-webkit-font-smoothing antialiased
  //-moz-osx-font-smoothing grayscale
  //text-align center
  //color #2c3e50
  //margin-top 60px
</style>
