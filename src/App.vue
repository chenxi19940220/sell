<template>
  <div id="app">
    <m-header :seller="seller"></m-header>
    <div class="tab border-bottom">
      <router-link tag="div" class="tab-item" :to="{path:'/goods'}">
        <span class="tab-link">商品</span>
      </router-link>
      <router-link tag="div" class="tab-item" :to="{path:'/ratings'}">
        <span class="tab-link">评论</span>
      </router-link>
      <router-link tag="div" class="tab-item" :to="{path:'/seller'}">
        <span class="tab-link">商家</span>
      </router-link>
    </div>
    <router-view></router-view>
  </div>
</template>

<script type="text/ecmascript-6">
import axios from 'axios'
import MHeader from 'components/m-header/m-header'

export default {
  name: 'App',
  components: {
    MHeader
  },
  data () {
    return {
      seller: {}
    }
  },
  created () {
    this._getSeller()
  },
  methods: {
    _getSeller () {
      axios.get('https://www.easy-mock.com/mock/5b61b78f2205a5414ac526b7/sell/data')
        .then((res) => {
          // console.log(res)
          if (res.request.status === 200 && res.request.readyState === 4) {
            // console.log(res.data)
            this.seller = res.data.seller
          }
        })
        .catch((err) => {
          console.log(err)
        })
    }
  }
}
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
.tab
  display flex
  width 100%
  height 40px
  line-height 40px
  .tab-item
    flex 1
    text-align center
    & > span
      display block
      font-size 14px
      color rgb(77, 85, 93)
  .active
    & > span
      color rgb(240, 20, 20)
</style>
