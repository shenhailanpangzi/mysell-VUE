<template>
  <div>
    <v-header :seller="seller"></v-header>
    <div class="tab border-1px">
      <div class="tab-item">
        <!-- 使用 router-link 组件来导航. -->
        <!-- 通过传入 `to` 属性指定链接. -->
        <!-- <router-link> 默认会被渲染成一个 `<a>` 标签 -->
        <router-link to="/goods">商品</router-link>
      </div>
      <div class="tab-item">
        <router-link to="/ratings">评论</router-link>
      </div>
      <div class="tab-item">
        <router-link to="/seller">商家</router-link>
      </div>
    </div>
    <!-- 路由出口 -->
    <!-- 路由匹配到的组件将渲染在这里 -->
    <router-view :seller="seller"></router-view>
  </div>
</template>

<script type="text/ecmascript-6">
  import header from './components/header/header.vue';

  const ERR_OK = 0;
  export default {
    data() {
      return {
        seller: {}
        // showHeader: true
      };
    },
    created() {
      this.$http.get('/api/seller').then((response) => {
        // 成功的回调 将response转换成json对象
        response = response.body;
        // console.log(response);
        if (response.errno === ERR_OK) {
          this.seller = Object.assign({}, this.seller, response.data);
          // console.log(this.seller);
        }
      });
    },
    components: {
      'v-header': header
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "./common/stylus/mixin.styl"

  .tab
    display flex
    width 100%
    height 40px
    line-height 40px
    // border-bottom: 1px solid rgba(7, 17, 27, 0.1)
    border-1px(rgba(7, 17, 27, 0.1))
    .tab-item
      flex 1
      text-align center
      /*&代表父元素  a代表a标签*/
      & > a
        /*a元素自动撑满整个div*/
        display block
        font-size 14px
        color rgb(77, 85, 93)
        /*当父元素的a标签 是active的时候*/
        &.active
         color: rgb(240, 20, 20)
</style>
