<template>
  <div>
    <div class="goods">
      <div
        class = "headtitle"
        v-if  = "fixedTitle"
        ref   = "fixed"
      >{{fixedTitle}}</div>
      <div
        class = "menu-wrapper"
        ref   = "menuWrapper"
      >
        <ul>
          <li
              v-for  = "(item, index) in goods"
            :key     = "index"
              class  = "menu-item"
            :class   = "{'current':currentIndex===index}"
              @click = "selectMenu(index,$event)"
              ref    = "menuList"
          >

            <span
              class  = "num"
              v-show = "barTxts && barTxts[index].count >0"
            >{{barTxts[index].count}}</span>
            <span class="text border-1px">
              {{item.name}}
            </span>
          </li>
        </ul>
      </div>
      <div
        class = "foods-wrapper"
        ref   = "foodsWrapper"
      >
        <ul>
          <li
              v-for = "(item,index) in goods"
            :key    = "index"
              class = "food-list"
              ref   = "foodList"
          >
            <h1 class="title">{{item.name}}</h1>
            <ul>
              <li
                  v-for  = "(food,index) in item.foods"
                :key     = "index"
                  class  = "food-item border-1px"
                  @click = "selectFood(food,$event)"
              >
                <div class="icon">
                  <img
                      width  = "57"
                      height = "57"
                    :src     = "food.icon"
                  >
                </div>
                <div class="content">
                  <h2 class="name">{{food.name}}</h2>
                  <p class="desc">{{food.description}}</p>
                  <div class="extra">
                    <span class="count">月售{{food.sellCount}}份</span><span>好评率{{food.rating}}%</span>
                  </div>
                  <div class="price">
                    <span class="now">￥{{food.price}}</span><span
                      class  = "old"
                      v-show = "food.oldPrice"
                    >￥{{food.oldPrice}}</span>
                  </div>
                  <div class="cartcontrol-wrapper">
                    <cartcontrol
                        @add = "addFood"
                      :food  = "food"
                    ></cartcontrol>
                  </div>
                </div>
              </li>
            </ul>
          </li>
        </ul>
      </div>
      <shopcart
          ref          = "shopcart"
        :deliveryPrice = "seller.deliveryPrice"
        :minPrice      = "seller.minPrice"
        :selectFoods   = "selectFoods"
      ></shopcart>
    </div>
    <food
      :food = "selectedFood"
        ref = "food"
    ></food>
  </div>
</template>


<script>
import BScroll from "better-scroll";
import cartcontrol from "../cartcontrol/cartcontrol";
import shopcart from "../shopcart/shopcart";
import food from "../food/food";

import axios from "axios";
const ERR_OK = 0;
const debug  = process.env.NODE_ENV !== "production";

export default {
  name : "goods",
  props: {
    seller: Object
  },
  data() {
    return {
      itemIndex   : 0,
      goods       : [],
      selectedFood: {},
      listHeight  : [],
      scrollY     : -1,
      diff        : -1
    };
  },
  created() {
    this._getSeller();
  },
  computed: {
    /**
     * 计算当前下标是否与侧边栏下标一致
     */
    currentIndex() {
      for (let i = 0; i < this.listHeight.length; i++) {
        let height1 = this.listHeight[i];
        let height2 = this.listHeight[i + 1];
        // console.log(height1, height2);

        if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
          this._followScroll(i);
          // console.log("看看编号：" + i);
          this.itemIndex = i;
          return i;
        }
      }
      return 0;
    },
    /**获取食物 */
    selectFoods() {
      let foods = [];
      this.goods.forEach(good => {
        good.foods.forEach(food => {
          if (food.count) {
            foods.push(food);
          }
        });
      });
      // console.log(foods);
      return foods;
    },
    totalCount() {
      let total = 0;
      this.selectFoods.forEach(food => {
        total += food.price * food.count;
      });
      return total;
    },
    fixedTitle() {
      //当scrolly大于  -标题高度的时候不显示
      console.log("现在的y" + this.scrollY);
      if (this.scrollY <= 0) {
        return "";
      }

      return this.goods[this.itemIndex] ? this.goods[this.itemIndex].name: "";
    },
    barTxts() {
      let ret = [];  //初始化数组
      this.goods.forEach(good => {
        const { type, name, foods } = good;
        let   count                 = 0;
        foods.forEach(food => {
          count += food.count || 0;
        });
        ret.push({
          type,
          name,
          count
        });
      });
      console.log(ret);
      return ret;
    }
  },

  methods: {
    _getSeller() {
      const url = debug
        ? "/api/goods"
        :  "http://ustbhuangyi.com/sell/api/seller";
      axios
        .get(url)
        .then(res => {
          const { errno, data } = res.data;
          if (errno === ERR_OK) {
            this.goods = data;
            // console.log(data);
            this.$nextTick(() => {
              this._initScroll();
              this._calculateHeight();
            });
          }
        })
        .catch(e => {});
    },
    /**
     * 点击选中食品详情
     */
    selectFood(food, event) {
      if (!event._constructed) {
        return;
      }
      console.log("点击成功了", this.selectedFood);

      this.selectedFood = food;
      this.$refs.food.show();
    },
    /**
     * 左边点击事件
     */
    selectMenu(index, event) {
      if (!event._constructed) {
        return;
      }
      let foodList = this.$refs.foodList;
      let el       = foodList[index];
      this.foodsScroll.scrollToElement(el, 300);
    },
    _initScroll() {
      this.menuScroll = new BScroll(this.$refs.menuWrapper, {
        click: true
      });
      this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
        click    : true,
        probeType: 3
      });
      this.foodsScroll.on("scroll", pos => {
        if (pos.y <= 0) {
          this.scrollY = Math.abs(Math.round(pos.y));
          //判断是否碰到定时
          if (this.scrollY >= this.listHeight[1].offsetTop - 30)
            this.$refs.fixed.style.transform = `translate3d(0,${
              this.scrollY
            }px,0)`;
          // console.log(this.scrollY);
        }
      });
    },
    _calculateHeight() {
      let foodList = this.$refs.foodList;
      let height   = 0;
      this.listHeight.push(height);
      for (let i = 0; i < foodList.length; i++) {
        let item    = foodList[i];
            height += item.clientHeight;
        this.listHeight.push(height);
      }
      // console.log(this.listHeight);
    },
    //滑动右边栏  侧边栏也滚动到相应下标位置
    _followScroll(index) {
      let menuList = this.$refs.menuList;
      let el       = menuList[index];
      this.menuScroll.scrollToElement(el, 300, 0, -100);
    },
    //小球降落动画触发
    addFood(target) {
      // console.log(target); //返回目标元素<div class="add_circle"></div>

      this._drop(target); //拿到子组件派发出来事件的参数
    },
    _drop(target) {
      // 体验优化,异步执行下落动画
      this.$nextTick(() => {
        this.$refs.shopcart.drop(target);
      });
    }
  },
  watch: {
    scrollY(newY) {
      console.log(newY);
    },
    diff(newVal) {
      console.log(newVal);
    }
  },
  components: {
    cartcontrol,
    shopcart,
    food
  }
};
</script>

<style lang="stylus" scoped>
@import '../../common/stylus/mixin.styl'
.goods
  display flex
  position absolute
  top 175px
  bottom 46px
  width 100%
  overflow hidden
  .headtitle
    position absolute
    left 80px
    width 100%
    z-index 20
    padding-left 14px
    height 30px
    line-height 30px
    border-left 2px solid #d9dde1
    font-size 12px
    color rgb(147, 153, 159)
    background #f3f5f7
  .menu-wrapper
    flex 0 0 80px
    width 80px
    background #f3f5f7
    .menu-item
      display table
      position relative
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
          border-none()
      .text
        display table-cell
        width 56px
        vertical-align middle
        border-1px(rgba(7, 17, 27, 0.1))
        font-size 12px
      .num
        position absolute
        top 6px
        right 2px
        width 16px
        height 16px
        line-height 16px
        text-align center
        border-radius 100%
        font-size 9px
        font-weight 700
        color #fff
        background rgb(240, 20, 20)
        box-shadow 0 4px 8px 0 rgba(0, 0, 0, 0.4)
  .foods-wrapper
    flex 1
    .title
      padding-left 14px
      height 30px
      line-height 30px
      border-left 2px solid #d9dde1
      font-size 12px
      color rgb(147, 153, 159)
      background #f3f5f7
    .food-item
      display flex
      margin 18px
      padding-bottom 18px
      border-1px(rgba(7, 17, 27, 0.1))
      &:last-child
        border-nono()
        margin-bottom 0
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
          color rgb(147, 153, 159)
          line-height 10px
          font-size 10px
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
            margin-right 8px
            font-size 14px
            color rgb(240, 20, 28)
          .old
            text-decoration line-through
            font-size 10px
            color rgb(147, 153, 159)
        .cartcontrol-wrapper
          position absolute
          right 0
          bottom 12px
</style>
