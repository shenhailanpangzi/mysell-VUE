<template>
  <div>
    <div class="shopcart">
      <div class="content" @click="toggleList">
        <div class="content-left" @click="listToggle">
          <div class="logo-wrapper">
              <!--当totalCount>0时 key：highlight属性将会被应用-->
            <div class="logo" :class="{'highlight':totalCount>0}">
              <i class="icon-shopping_cart" :class="{'highlight':totalCount>0}"></i>
            </div>
            <!--购物车数量小点-->
            <div class="num" v-show="totalCount>0">{{totalCount}}</div>
          </div>
          <div class="price" :class="{'highlight':totalPrice>0}">￥{{totalPrice}}</div>
          <div class="desc">另需配送费￥{{deliveryPrice}}元</div>
        </div>
        <!--控制底部右边内容随food的变化而变化，payDesc()控制显示内容，payClass()添加类调整显示样式-->
        <div class="content-right">
          <!--:class绑定计算属性变量payClass-->
          <div class="pay" :class="payClass" @click="pay">
            {{payDesc}}
          </div>
        </div>
      </div>
      <!--小球掉入购物车动画 通过两个层来控制小球，外层控制一个方向的变化，内层控制另外一个方向的变化（写两层才会有抛物线的效果），采用fixed布局（是相对于视口的动画）-->
      <div class="ball-container">
        <div v-for="(ball,item) in balls" :key="item">
          <!--因为小球是有去无回的动画过程，这里采用vue中提供的钩子-->
          <transition name="drop" @before-enter="beforeDrop" @enter="dropping" @after-enter="afterDrop">
            <div class="ball" v-show="ball.show">
              <div class="inner inner-hook"></div>
            </div>
          </transition>
        </div>
      </div>
      <!--购物车详情模块-->
      <transition name="fold">
        <div class="shopcart-list" v-show="listShow">
          <div class="list-header">
            <h1 class="title">购物车</h1>
            <span class="empty" @click="empty">清空</span>
          </div>
          <div class="list-content" ref="foodlist">
            <ul>
              <li class="food" v-for="(food,item) in selectFoods" :key="item">
                <span class="name">{{food.name}}</span>
                <div class="price">
                  <span>￥{{food.price*food.count}}</span>
                </div>
                <div class="cartcontrol-wrapper">
                  <cartcontrol @add="addFood" :food="food"></cartcontrol>
                </div>
              </li>
            </ul>
          </div>
        </div>
      </transition>
    </div>
    <!--<transition name="fade-backdrop">-->
      <!--<div class="backdrop" v-show="showBackdrop" @click="hideBackdrop"></div>-->
    <!--</transition>-->
  </div>
</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll';
  import cartcontrol from 'components/cartcontrol/cartcontrol';
  export default {
    props: {
      // 选择了多少商品：定义成数组，底栏其余部分的变化都基于这个对象的变化而变化
      selectFoods: {
        type: Array,
        default() {
          return [
            {
              price: 0,
              count: 0
            }
          ];
        }
      },
      seller: {
        type: Object
      },
      // 配送费
      deliveryPrice: {
        type: Number,
        default: 0
      },
      // 最少起送费
      minPrice: {
        type: Number,
        default: 0
      }
    },
    data() {
      return {
        balls: [
          {
            show: false
          },
          {
            show: false
          },
          {
            show: false
          },
          {
            show: false
          },
          {
            show: false
          }
        ],
        dropBalls: [],
        // 折叠
        fold: false,
        listShow: false
      };
    },
    // 计算部分（都基于selectFoods进行相应计算）computed中的函数可以直接在Tempplate中以指针的形式引用
    computed: {
      // 计算总价，超过起送额度后提示可付款
      totalPrice() {
        let total = 0;
        this.selectFoods.forEach((food) => {
          total += food.price * food.count;
        });
        return total;
      },
      // 计算选中的food数量，在购物车图标处显示，采用绝对定位，top：0；right：0；显示在购物车图标右上角
      totalCount() {
        let count = 0;
        this.selectFoods.forEach((food) => {
          count += food.count;
        });
        return count;
      },
      payDesc() {
        // 建议用全等号=== 不建议用== 因为==在类型不一致的情况下会转化
        if (this.totalPrice === 0) {
          return `￥${this.minPrice}元起送`;
        } else if (this.totalPrice < this.minPrice) {
          let diff = this.minPrice - this.totalPrice;
          return `还差￥${diff}元起送`;
        } else {
          return '去结算';
        }
      },
      payClass() {
        if (this.totalPrice < this.minPrice) {
          return 'not-enough';
        } else {
          return 'enough';
        }
      }
    },
    methods: {
      listToggle() {
        if (!this.selectFoods.length) {
          return;
        }
        this.listShow = !this.listShow;
        if (this.listShow) {
          this.$nextTick(() => {
            if (!this.foodlistScroll) {
              this._initScroll();
            } else {
              this.foodlistScroll.refresh();
            }
          });
        }
      },
      drop(el) {
        for (let i = 0; i < this.balls.length; i++) {
          let ball = this.balls[i];
          if (!ball.show) {
            // 触发动画
            ball.show = true;
            ball.el = el;
            this.dropBalls.push(ball);
            return;
          }
        }
      },
      toggleList() {
        if (!this.totalCount) {
          return;
        }
        this.fold = !this.fold;
      },
      empty() {
        this.selectFoods.forEach((food) => {
          food.count = 0;
        });
        this.fold = true;
        this.listShow = false;
      },
      pay() {
        alert('小破圆~');
        // if (this.totalPrice < this.minPrice) {
        //   return;
        // }
        // window.selectedGoods = JSON.stringify(this.selectFoods);
        // window.sellerPay = JSON.stringify(this.seller);
        // window.location.href = '#/payment';
      },
      addFood(target) {
        this.drop(target);
      },
      // 当前dom
      beforeDrop(el) {
        let count = this.balls.length;
        while (count--) {
          let ball = this.balls[count];
          if (ball.show) {
            let rect = ball.el.getBoundingClientRect();
            let x = rect.left - 32;
            let y = -(window.innerHeight - rect.top - 22);
            el.style.display = '';
            el.style.webkitTransform = `translate3d(0,${y}px,0)`;
            el.style.transform = `translate3d(0,${y}px,0)`;
            let inner = el.getElementsByClassName('inner-hook')[0];
            inner.style.webkitTransform = `translate3d(${x}px,0,0)`;
            inner.style.transform = `translate3d(${x}px,0,0)`;
          }
        }
      },
      dropping(el, done) {
        /* eslint-disable no-unused-vars */
        let rf = el.offsetHeight;
        this.$nextTick(() => {
          el.style.webkitTransform = 'translate3d(0,0,0)';
          el.style.transform = 'translate3d(0,0,0)';
          let inner = el.getElementsByClassName('inner-hook')[0];
          inner.style.webkitTransform = 'translate3d(0,0,0)';
          inner.style.transform = 'translate3d(0,0,0)';
          el.addEventListener('transitionend', done);
        });
      },
      afterDrop(el) {
        let ball = this.dropBalls.shift();
        if (ball) {
          ball.show = false;
          el.style.display = 'none';
        }
      },
      // 初始化Scroll滑动模块
      _initScroll() {
        this.foodlistScroll = new BScroll(this.$refs.foodlist, {
          // 结合BScroll的接口使用,是否将click事件传递,默认被拦截了
          click: true
        });
        this.$nextTick(() => {
          if (!this.scroll) {
            this.scroll = new BScroll(this.$refs.foodlist, {
              click: true
            });
          } else {
            this.scroll.refresh();
          }
        });
      }
    },
    watch: {
      // 如果 `selectFoods` 发生改变，这个函数就会运行
      selectFoods: function() {
        if (!this.selectFoods.length) {
          this.listShow = false;
          this.fold = true;
        }
      }
    },
    components: {
      cartcontrol
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin.styl"

  .shopcart
    position: fixed
    left: 0
    bottom: 0
    z-index: 50
    width: 100%
    height: 48px
    .content
      display: flex
      background: #141d27
      font-size: 0
      color: rgba(255, 255, 255, 0.4)
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
          background: #141d27
          .logo
            width: 100%
            height: 100%
            border-radius: 50%
            text-align: center
            background: #2b343c
            &.highlight
              background: rgb(0, 160, 220)
            .icon-shopping_cart
              line-height: 44px
              font-size: 24px
              color: #80858a
              &.highlight
                color: #fff
          .num
            position: absolute
            top: 0
            right: 0
            width: 24px
            height: 16px
            line-height: 16px
            text-align: center
            border-radius: 16px
            font-size: 9px
            font-weight: 700
            color: #fff
            background: rgb(240, 20, 20)
            box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.4)
        .price
          display: inline-block
          vertical-align: top
          margin-top: 12px
          line-height: 24px
          padding-right: 12px
          box-sizing: border-box
          border-right: 1px solid rgba(255, 255, 255, 0.1)
          font-size: 16px
          font-weight: 700
          &.highlight
            color: #fff
        .desc
          display: inline-block
          vertical-align: top
          margin: 12px 0 0 12px
          line-height: 24px
          font-size: 10px
      .content-right
        flex: 0 0 105px
        width: 105px
        .pay
          height: 48px
          line-height: 48px
          text-align: center
          font-size: 12px
          font-weight: 700
          &.not-enough
            background: #2b333b
          &.enough
            background: #00b43c
            color: #fff
    .ball-container
      .ball
        position: fixed
        left: 32px
        bottom: 22px
        z-index: 200
        // 关于cubic-bezier(0.49, -0.29, 0.75, 0.41),是动画抛物曲线(贝塞尔曲线)的配置，可以利用http://cubic-bezier.com/#.23,1.14,.83,.67 进行曲线调试
        transition: all 0.4s cubic-bezier(0.49, -0.29, 0.75, 0.41)
        .inner
          width: 16px
          height: 16px
          //圆形
          border-radius: 50%
          background: rgb(0, 160, 220)
          transition: all 0.4s linear
    .shopcart-list
      position: absolute
      left: 0
      top: 0
      z-index: -1
      width: 100%
      transform: translate3d(0, -100%, 0)
      &.fold-enter-active, &.fold-leave-active
        transition: all 0.5s
      &.fold-enter, &.fold-leave-active
        transform: translate3d(0, 0, 0)
      .list-header
        height: 40px
        line-height: 40px
        padding: 0 18px
        background: #f3f5f7
        border-bottom: 1px solid rgba(7, 17, 27, 0.1)
        .title
          float: left
          font-size: 14px
          color: rgb(7, 17, 27)
        .empty
          float: right
          font-size: 12px
          color: rgb(0, 160, 220)

      .list-content
        padding: 0 18px
        max-height: 217px
        overflow: hidden
        background: #fff
        .food
          position: relative
          padding: 12px 0
          box-sizing: border-box
          border-1px(rgba(7, 17, 27, 0.1))
          .name
            line-height: 24px
            font-size: 14px
            color: rgb(7, 17, 27)
          .price
            position: absolute
            right: 90px
            bottom: 12px
            line-height: 24px
            font-size: 14px
            font-weight: 700
            color: rgb(240, 20, 20)
          .cartcontrol-wrapper
            position: absolute
            right: 0
            bottom: 6px

  .list-mask
    position: fixed
    top: 0
    left: 0
    width: 100%
    height: 100%
    z-index: 40
    backdrop-filter: blur(10px)
    opacity: 1
    background: rgba(7, 17, 27, 0.6)
    &.fade-enter-active, &.fade-leave-active
      transition: all 0.5s
    &.fade-enter, &.fade-leave-active
      opacity: 0
      background: rgba(7, 17, 27, 0)
</style>
