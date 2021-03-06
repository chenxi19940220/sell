<template>
  <div>
    <div class="goods">
      <div class="menu-wrapper" ref="menuWrapper">
        <ul>
          <li v-for="(item, index) in goods"
              :key="item.id"
              class="menu-item"
              :class="{'current':currentIndex === index}"
              ref="menuList"
              @click="selectMenu(index, $event)"
          >
            <span class="text border-bottom">
              <span v-show="item.type > 0" class="icon" :class="classMap[item.type]"></span>{{item.name}}
            </span>
          </li>
        </ul>
      </div>
      <div class="foods-wrapper" ref="foodsWrapper">
        <ul>
          <li v-for="item in goods" :key="item.id" class="food-list" ref="foodList">
            <h1 class="title">{{item.name}}</h1>
            <ul>
              <li @click="selectFood(food, $event)" v-for="(food, index) in item.foods" :key="index" class="food-item border-bottom">
                <div class="icon">
                  <img width="57" height="57" :src="food.icon" alt="">
                </div>
                <div class="content">
                  <h2 class="name">{{food.name}}</h2>
                  <p class="desc">{{food.description}}</p>
                  <div class="extra">
                    <span class="count">月售{{food.sellCount}}</span><span>好评率{{food.rating}}%</span>
                  </div>
                  <div class="price">
                    <span class="now">￥{{food.price}}</span>
                    <span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
                  </div>
                  <div class="cartcontrol-wrapper">
                    <cartcontrol :food="food" @add="addFood"></cartcontrol>
                  </div>
                </div>
              </li>
            </ul>
          </li>
        </ul>
      </div>
      <shopcart ref='shopcart' :select-foods="selectFoods" :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice"></shopcart>
    </div>
    <food @add="addFood" :food="selectedFood" ref="food"></food>
  </div>
</template>

<script type="text/ecmascript-6">
import axios from 'axios'
import BScroll from 'better-scroll'
import Shopcart from 'components/shopcart/shopcart'
import Cartcontrol from 'components/cartcontrol/cartcontrol'
import Food from 'components/food/food'

export default {
  name: 'Goods',
  data () {
    return {
      goods: [],
      seller: {},
      listHeight: [],
      scrollY: 0,
      selectedFood: {}
    }
  },
  computed: {
    currentIndex () {
      for (let i = 0; i < this.listHeight.length; i++) {
        let height1 = this.listHeight[i]
        let height2 = this.listHeight[i + 1]
        if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
          this._followScroll(i)
          return i
        }
      }
      return 0
    },
    selectFoods () {
      let foods = []
      this.goods.forEach((good) => {
        good.foods.forEach((food) => {
          if (food.count) {
            foods.push(food)
          }
        })
      })
      return foods
    }
  },
  created () {
    this._getSeller()
    this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee']
  },
  methods: {
    selectMenu (index, event) {
      if (!event._constructed) {
        return
      }
      let foodList = this.$refs.foodList
      let el = foodList[index]
      this.foodsScroll.scrollToElement(el, 300)
    },
    _getSeller () {
      axios.get('https://www.easy-mock.com/mock/5b61b78f2205a5414ac526b7/sell/data')
        .then((res) => {
          // console.log(res)
          if (res.request.status === 200 && res.request.readyState === 4) {
            // console.log(res.data)
            this.seller = res.data.seller
            this.goods = res.data.goods
            // console.log(this.seller)
            this.$nextTick(() => {
              this._initScroll()
              this._calculateHeight()
            })
          }
        })
        .catch((err) => {
          console.log(err)
        })
    },
    selectFood (food, event) {
      if (!event._constructed) {
        return
      }
      this.selectedFood = food
      this.$refs.food.show()
    },
    addFood (target) {
      this._drop(target)
    },
    _drop (target) {
      // 体验优化，异步执行下落动画
      this.$nextTick(() => {
        this.$refs.shopcart.drop(target)
      })
    },
    _initScroll () {
      this.menuScroll = new BScroll(this.$refs.menuWrapper, {
        click: true
      })
      this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
        click: true,
        probeType: 3
      })

      this.foodsScroll.on('scroll', (pos) => {
        // 判断滑动方向，避免下拉时分类高亮错误（如第一分类商品数量为1时，下拉使得第二分类高亮）
        if (pos.y <= 0) {
          this.scrollY = Math.abs(Math.round(pos.y))
        }
      })
    },
    _calculateHeight () {
      let foodList = this.$refs.foodList
      let height = 0
      this.listHeight.push(height)
      // console.log(foodList)
      for (let i = 0; i < foodList.length; i++) {
        let item = foodList[i]
        height += item.clientHeight
        height = height - 1
        this.listHeight.push(height)
      }
    },
    _followScroll (index) {
      let menuList = this.$refs.menuList
      // console.log(menuList)
      let el = menuList[index]
      // console.log(menuList)
      this.menuScroll.scrollToElement(el, 300, 0, -100)
    }
  },
  components: {
    Shopcart,
    Cartcontrol,
    Food
  }
}
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
@import "~common/stylus/mixin"

.border-bottom
  &:before
    border-color: rgba(7, 17, 27, 0.1)
.goods
  display flex
  position absolute
  top 174px
  bottom 46px
  width 100%
  overflow hidden
  .menu-wrapper
    flex 0 0 80px
    width 80px
    background #f3f5f7
    .menu-item
      display table
      height 54px
      width 56px
      padding 0 12px
      line-height 14px
      &.current
        position relative
        z-index 10
        margin-top -1px
        background #fff
        font-weight 700
        .text
          &.border-bottom
            &:before
              border none
      .icon
        display inline-block
        vertical-align top
        width 12px
        height 12px
        margin-right 4px
        background-size 12px 12px
        background-repeat no-repeat
        &.decrease
          bg-image('images/decrease_3')
        &.discount
          bg-image('images/discount_3')
        &.guarantee
          bg-image('images/guarantee_3')
        &.invoice
          bg-image('images/invoice_3')
        &.special
          bg-image('images/special_3')
      .text
        display table-cell
        width 56px
        vertical-align middle
        font-size 12px
        text-align center
  .foods-wrapper
    flex 1
    .title
      padding-left 14px
      height 26px
      line-height 26px
      border-left: 2px solid #d9dde1
      font-size 12px
      color rgb(147, 153, 159)
      background #f3f5f7
    .food-item
      display flex
      margin 18px
      padding-bottom 18px
      &:last-child
        margin-bottom 0
        &.border-bottom
          &:before
            border none
      .icon
        flex 0 0 57px
        margin-right 10px
      .content
        flex 1
        .name
          margin 2px 0 8px 0
          height 14px
          line-height 14px
          font-size 14px
          color rgb(7, 17, 27)
        .desc, .extra
          line-height 10px
          font-size 10px
          color rgb(147, 153, 159)
        .desc
          line-height 12px
          margin-bottom 8px
        .extra
          .count
            margin-right 12px
        .price
          font-weight 700
          line-height 24px
          .now
            margin-right 18px
            font-size 14px
            color rgb(240, 20, 20)
          .old
            text-decoration line-through
            font-size 10
        .cartcontrol-wrapper
          position absolute
          right 0
          bottom 12px
</style>
