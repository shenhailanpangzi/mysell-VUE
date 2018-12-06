<template>
  <!--增减商品按钮组件-->
    <div class="cartcontrol">
      <!--减少按钮动画效果-->
      <transition name="move">
      <div class="cart-decrease icon-remove_circle_outline" v-show="food.count>0"
           @click.stop.prevent="decreaseCart">
        <span class="inner icon-remove_circle_outline"></span>
      </div>
      </transition>
      <div class="cart-count" v-show="food.count>0">
        {{food.count}}
      </div>
      <!--@click.stop.prevent解决点击事件冒泡问题 阻止冒泡-->
      <div class="cart-add icon-add_circle" @click.stop.prevent="addCart">
      </div>
    </div>
</template>

<script type="text/ecmascript-6">
  import Vue from 'vue';
  export default {
    props: {
      food: {
        type: Object
      }
    },
    created() {
    },
    methods: {
      addCart(event) {
        // 如果不是自己派生发点击事件 防止pc点击触发两次的问题
        console.log(event._constructed);
        if (!event._constructed) {
          return;
        }
        // console.log('click');
        // this.food.count不存在
        if (!this.food.count) {
          // 使用vue.set方法添加属性 这样这个属性才能被观测到 才能使dom发生变化
          Vue.set(this.food, 'count', 1);
          // this.food.count = 1;
        } else {
          this.food.count++;
        }
        // 这句话的意思是提交名为'add'的事件给父组件;
        this.$emit('add', event.target, this.food);
      },
      decreaseCart(event) {
        console.log('cartcontrol:decreaseCart');
        // 如果不是自己派生发点击事件 防止pc点击触发两次的问题
        if (!event._constructed) {
          return;
        }
        // console.log('click');
        if (this.food.count) {
          this.food.count--;
        }
      }
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  .cartcontrol
    font-size 0
    .cart-decrease
      display: inline-block
      padding: 6px
      opacity: 1
      transform: translate3d(0, 0, 0)
      .inner
        display: inline-block
        line-height: 24px
        font-size: 24px
        color: rgb(0, 160, 220)
        transition: all 0.4s linear
        transform: rotate(0)
      &.move-enter-active, &.move-leave-active
        transition: all 0.4s linear
      &.move-enter, &.move-leave-active
        opacity: 0
        transform: translate3d(24px, 0, 0)
        .inner
          transform: rotate(180deg)
    .cart-add
      display inline-block
      padding 6px
      line-height 24px
      font-size 24px
      color rgb(0,160,220)
    .cart-count
      display inline-block
      // vertical-align 属性设置元素的垂直对齐方式。 把元素的顶端与行中最高元素的顶端对齐
      vertical-align top
      width 12px
      padding-top 6px
      line-height 24px
      text-align center
      font-size 10px
      color rgb(147, 153, 159)
    .cart-add
      display inline-block
</style>
