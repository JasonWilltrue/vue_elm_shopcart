<template>
  <transition name="move">
    <div
      v-show = "showFlag"
      class  = "food"
      ref    = "food"
    >
      <div class="food-content">
        <div class="image-header">
          <img
            :src  = "food.icon"
              alt = "meitu"
          >
          <div
            class  = "back"
            @click = "hide"
          >
            <i class="icon-arrow_lift"></i>
          </div>
        </div>
        <!-- 价格---购物车区域 -->
        <div class="content">
          <h1 class="title">{{food.name}}</h1>
          <div class="detail">
            <span class="sell-count">月销售{{food.sellCount}}</span>
            <span class="rating">好评率{{food.rating}}</span>
          </div>
          <div class="price">
            <span class="now">￥{{food.price}}</span>
            <span
              class  = "old"
              v-show = "food.oldPrice"
            >￥{{food.oldPrice}}</span>
          </div>
          <div class="cartcontrol-wrapper">
            <cartcontrol :food="food"></cartcontrol>
          </div>
          <transition name="fade">
            <div
              @click.stop.prevent = "addFirst($event)"
              class               = "buy"
              v-show              = "!food.count || food.count===0"
            >
              加入购物车
            </div>
          </transition>
        </div>
        <!-- 价格---购物车区域 -->
        <!-- 信息区域开始 -->
        <split v-show="food.info"></split>
        <div
          class  = "info"
          v-show = "food.info"
        >
          <h1 class="title">商品信息</h1>
          <p class="text">{{food.info}}</p>
        </div>
        <split></split>
        <!-- 信息区域结束 -->
        <div class="rating">
          <h1 class="title">商品评价</h1>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
import BScroll from "better-scroll";
import Vue from "vue";
import cartcontrol from "../cartcontrol/cartcontrol";
import split from "../split/split";
export default {
  name : "food",
  props: {
    food: {
      type: Object
    }
  },
  data() {
    return {
      showFlag: false
    };
  },
  methods: {
    show() {
      this.showFlag = true;
      this.$nextTick(() => {
        if (!this.scroll) {
          this.scroll = new BScroll(this.$refs.food, {
            click: true
          });
        } else {
          this.scroll.refresh(); //重绘
        }
      });
    },
    hide() {
      this.showFlag = false;
    },
    addFirst(event) {
      if (!event._constructed) {
        return;
      }
      this.$emit("add", event.target);
      Vue.set(this.food, "count", 1);
    }
  },
  components: {
    cartcontrol,
    split
  }
};
</script>

<style lang="stylus" scoped>
@import '../../common/stylus/mixin.styl'
.food
  position fixed
  left 0
  top 0
  bottom 48px
  z-index 30
  width 100%
  background #fff
  transform translate3d(0, 0, 0)
  &.move-enter-active, &.move-leave-active
    transition all 0.2s linear
  &.move-enter, &.move-leave-active
    transform translate3d(100%, 0, 0)
  .image-header
    position relative
    width 100%
    height 0
    padding-bottom 100%
    img
      position absolute
      top 0
      left 0
      width 100%
      height 100%
    .back
      position absolute
      left 0px
      top 10px
      .icon-arrow_lift
        padding 12px
        display block
        font-size 20px
        color #ffffff
  .content
    position relative
    padding 18px
    .title
      line-height 14px
      margin-bottom 8px
      font-size 14px
      font-weight 700
      color rgb(7, 17, 27)
    .detail
      margin-bottom 18px
      line-height 10px
      height 10px
      font-size 0
      .sell-count, .rating
        font-size 10px
        color rgb(147, 153, 159)
      .sell-count
        margin-right 12px
    .price
      font-weight 700
      line-height 24px
      .now
        margin-right 8px
        font-size 14px
        color rgb(240, 20, 20)
      .old
        text-decoration line-through
        font-size 10px
        color rgb(147, 153, 159)
    .cartcontrol-wrapper
      position absolute
      right 12px
      bottom 12px
    .buy
      position absolute
      right 18px
      bottom 18px
      z-index 10
      height 24px
      line-height 24px
      padding 0 12px
      box-sizing border-box
      border-radius 12px
      font-size 10px
      color #fff
      background rgb(0, 160, 220)
      opacity 1
      &.fade-enter-active, &.fade-leave-active
        transition all 0.2s
      &.fade-enter, &.fade-leave-active
        opacity 0
        z-index -1
  .info
    padding 18px
    .title
      line-height 14px
      margin-bottom 6px
      font-size 14px
      color rgb(7, 17, 27)
    .text
      line-height 24px
      padding 0 8px
      font-size 12px
      color rgb(77, 85, 93)
</style>
