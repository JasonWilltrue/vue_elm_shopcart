<template>
  <div id="app">
    <v-header></v-header>
    <div class="tab border-1px">
      <div class="tab-item">
        <router-link to="/goods">商品</router-link>
      </div>
      <div class="tab-item">
        <router-link to="/ratings">评论</router-link>
      </div>
      <div class="tab-item">
        <router-link to="/seller">商家</router-link>
      </div>
    </div>

    <router-view :seller="seller"></router-view>

  </div>
</template>
<script>
import qs from "query-string";
import axios from "axios";
import { getSeller } from "./api/index.js";
import VHeader from "./components/v-header/v-header";

const ERR_OK = 0;
const debug  = process.env.NODE_ENV !== "production";

export default {
  components: {
    VHeader
  },
  data() {
    console.log("为什么" + qs.parse(location.search).id);

    return {
      seller: {
        id: qs.parse(location.search).id
      }
    };
  },
  created() {
    this._getSeller();
  },

  methods: {
    _getSeller() {
      const url = debug
        ? "/api/seller"
        :  "http://ustbhuangyi.com/sell/api/seller";
      axios
        .get(url)
        .then(res => {
          const { errno, data } = res.data;
          if (errno === ERR_OK) {
            this.seller = Object.assign({}, this.seller, data);
          }
        })
        .catch(e => {});
    }
  }
};
</script>



<style lang="stylus" scoped>
@import './common/stylus/mixin.styl'
.tab
  display flex
  width 100%
  height 40px
  line-height 40px
  border-1px(rgba(7, 17, 27, 0.1))
  .tab-item
    flex 1
    text-align center
    & > a
      display block
      font-size 14px
      color rgb(77, 85, 93)
      &.active
        color rgb(240, 20, 20)
        border-1px(rgb(240, 20, 20))
        transform all 1.2s
</style>
