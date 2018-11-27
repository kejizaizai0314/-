<template>
  <div>
    <div class="goods">
      <div class="menu-wrapper" ref="menuWrapper">
        <ul>
          <li class="menu-list" v-for="(item,index) in goods" :class = "{'current': currentIndex === index}" @click="selectMenu(index)" ref="menuList" :key = 'item.num'>
            <span class="menu-name"><span class="icon" :class="classMap[item.type]" v-show="item.type>0"></span>{{item.name}}</span>
          </li>
        </ul>
      </div>
      <div class="foods-wrapper" ref="foodsWrapper">
        <ul>
          <li class="foods-list" v-for="item in goods" ref="foodsList" :key="item.num">
            <div class="title">{{item.name}}</div>
            <ul>
              <li class="foods-desc" v-for="food in item.foods" :key="food.num">
                <div class="icon">
                  <img :src="food.icon" width="57" height="57">
                </div>
                <div class="content">
                  <span class="name">{{food.name}}</span>
                  <div class="desc">{{food.description}}</div>
                  <div class="extra">
                    <span>月售{{food.sellCount}}份 好评率{{food.rating}}%</span>
                  </div>
                  <div class="price">
                    <span class="new">￥{{food.price}}</span>
                    <span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
                  </div>
                </div>
              </li>
            </ul>
          </li>
        </ul>
      </div>
      <shopcart :deliveryPrice="seller.deliveryPrice" :minPrice="seller.minPrice"></shopcart>
    </div>
  </div>
</template>
<script>
import BScroll from 'better-scroll'
import shopcart from '../shopcart/shopcart'
const ERR_OK = 0
export default {
  props: {
    seller: {
      type: Object
    }
  },
  data () {
    return {
      goods: [],
      listHeight: [],
      scrollY: 0
    }
  },
  computed: {
    currentIndex () {
      for (let i = 0; i < this.listHeight.length; i++) {
        let height1 = this.listHeight[ i ]
        let height2 = this.listHeight[i + 1]
        if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
          return i
        }
      }
    }
  },
  created () {
    this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee']
    this.$http.get('/api/goods').then((response) => {
      response = response.body
      if (response.errno === ERR_OK) {
        this.goods = response.data
        this._initScroll()
        this.$nextTick(() => {
          this._calculateHeight()
        })
      }
    })
  },
  methods: {
    _initScroll () {
      this.menuScroll = new BScroll(this.$refs.menuWrapper, {
        click: true
      })
      this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
        probeType: 3,
        click: true
      })
      this.foodsScroll.on('scroll', (pos) => {
        this.scrollY = Math.abs(Math.round(pos.y))
      })
    },
    _calculateHeight () {
      let foodsList = this.$refs.foodsList
      let height = 0
      this.listHeight.push(height)
      for (let i = 0; i < foodsList.length; i++) {
        let item = foodsList[i]
        height += item.clientHeight
        this.listHeight.push(height)
      }
    },
    selectMenu (index) {
      let foodsList = this.$refs.foodsList
      let element = foodsList[index]
      this.foodsScroll.scrollToElement(element, 300)
    }
  },
  components: {
    shopcart
  }
}
</script>
<style lang="stylus">
@import '../../common/stylus/mixin.styl'
  .goods
    display flex
    position absolute
    top 174px
    bottom 46px
    width 100%
    overflow hidden
    .menu-wrapper
      flex 0 0 80px
      background-color #f3f5f7
      .menu-list
        display table
        width 56px
        height 54px
        padding 0 12px
        line-height 14px
        &.current
          margin-top -1px
          font-weight 700
          background-color #fff
          .text
            border-none()
        .icon
          display inline-block
          width 12px
          height 12px
          vertical-align top
          margin-right 2px
          background-size 12px 12px
          background-repeat no-repeat
          &.decrease
            bg-image('decrease_3')
          &.discount
            bg-image('discount_3')
          &.guarantee
            bg-image('guarantee_3')
          &.invoice
            bg-image('invoice_3')
          &.special
            bg-image('special_3')
        .menu-name
          display table-cell
          vertical-align middle
          font-size 12px
          border-1px(rgba(7,17,27,0.1))
    .foods-wrapper
      flex 1
      .foods-list
        list-style none
        .title
          height 20px
          padding-left 18px
          padding-top 3px
          border-left 2px solid #d9dde1
          color rgb(147,153,159)
          background-color #f3f5f7
        .foods-desc
          display flex
          .icon
            flex 0 0 57px
            margin 18px 10px 0 18px
          .content
            flex 1
            margin-top 15px
            .price
              font-weight: 700
              line-height: 24px
              .new
                margin-right: 8px
                font-size: 17px
                color: rgb(240, 20, 20)
              .old
                text-decoration: line-through
                font-size: 1px
                color: rgb(147, 153, 159)
</style>
