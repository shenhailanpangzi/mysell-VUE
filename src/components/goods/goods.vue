<template>
  <!--参考博客https://www.cnblogs.com/yujihang/p/6886532.html-->
  <div class="goods">
    <!--左侧-->
    <div class="menu-wrapper" ref="menuWrapper">
      <ul>
<!--通过计算属性currentIndex,获取到food滚动区域对应的menu区域的子块的索引,然后通过设置一个class来做样式切换变化 :class="{'current':currentIndex === index} ,实现联动-->
<!--当点击menu 区域的时候,会触发selectMenu事件,也会根据点击到的menu子块的索引然后去触发food区域滚动到对应的高度区块区间-->
        <li v-for="(item,index) in goods" class="menu-item" :key="index"
            :class="{'current':currentIndex===index}"
            @click="selectMenu(index,$event)">
          <span class="text border-1px">
            <span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>{{item.name}}
          </span>
        </li>
      </ul>
    </div>
    <!--右侧-->
    <div class="foods-wrapper" ref="foodsWrapper">
      <ul>
        <ul>
          <li v-for="(item,index) in goods" class="food-list" :key="index" ref="foodList">
            <h1 class="title">{{item.name}}</h1>
            <ul>
              <li v-for="(food,index) in item.foods" class="food-item border-1px" :key="index">
                <div class="icon">
                  <img width="57" height="57" :src="food.icon">
                </div>
                <div class="content">
                  <h2 class="name">{{food.name}}</h2>
                  <p class="desc">{{food.description}}</p>
                  <div class="extra">
                    <span class="count">月售{{food.sellCount}}份</span><span>好评率{{food.rating}}%</span>
                  </div>
                  <div class="price">
                    <span class="now">￥{{food.price}}</span>
                    <span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
                  </div>
                  <div class="cartcontrol-wrapper">
                    <cartcontrol @add="addFood" :food="food"></cartcontrol>
                  </div>
                </div>
              </li>
            </ul>
          </li>
        </ul>
      </ul>
    </div>
    <!--购物车 父组件访问子组件-->
    <shopcart ref="shopcart" :selectFoods="selectFoods" :seller="seller"
              :deliveryPrice="seller.deliveryPrice" :minPrice="seller.minPrice"></shopcart>
  </div>
</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll';
  import shopcart from 'components/shopcart/shopcart';
  import cartcontrol from 'components/cartcontrol/cartcontrol';
  const ERR_OK = 0;
  export default {
    props: {
      seller: {
        type: Object
      }
    },
    data() {
      return {
        selectedFoods: {},
        goods: [],
        listHeight: [],
        scrollY: 0,
        selectedFood: {}
      };
    },
    // 计算class属性值
    computed: {
      // 计算到达哪个区域的区间的时候的对应的索引值
      currentIndex() {
        for (let i = 0; i < this.listHeight.length; i++) {
          // 当前menu子块的高度
          let height1 = this.listHeight[i];
          // 下一个menu子块的高度
          let height2 = this.listHeight[i + 1];
          // 滚动到底部的时候,height2为undefined,需要考虑这种情况
          // 需要确定是在两个menu子块的高度区间
          if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
            // 返回这个menu子块的索引
            return i;
          }
        }
        return 0;
      },
      // 自动将所有的goods.food添加一个count属性,方便做数量运算
      selectFoods() {
        let foods = [];
        this.goods.forEach((good) => {
          good.foods.forEach((food) => {
            if (food.count) {
              foods.push(food);
            }
          });
        });
        return foods;
      }
    },
    created() {
      this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];
      // es6新增了let命令，用来声明变量。它的用法类似于var，但是所声明的变量，只在let命令所在的代码块内有效。
      let selectedGoods = window.selectedGoods;
      selectedGoods = selectedGoods ? JSON.parse(selectedGoods) : [];
      this.$http.get('/api/goods').then((response) => {
        response = response.body;
        console.log('response:' + response);
        if (response.errno === ERR_OK) {
          selectedGoods.map(item => {
            response.data.map((food, index) => {
//              console.log(food);
              food.foods.map((foods, i) => {
                // console.log(foods, item);
                if (foods.id === item.id) {
                  food.foods.splice(i, 1, Object.assign(food.foods[i], {count: item.count}));
                  // food.foods[i].count = item.count;
                  response.data.splice(index, 1, food);
                }
              });
            });
          });
          this.goods = response.data;
//        console.log('hello world', this.goods);
//       可以用 $nextTick 來确保Dom变化后再执行一些事情 $nextTick，则可以在回调中获取更新后的 DOM，
//       官方解释：在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM。
          this.$nextTick(() => {
        // 在vue实例生命周期的开始created分别加载 _initScroll 和 _calculateHeight
            this._initScroll();
            this._calculateHeight();
          });
        }
      });
    },
    methods: {
      selectMenu(index, event) {
        // 去掉自带的click事件点击，即pc端直接返回
        if (!event._constructed) {
          return;
        }
        let foodList = this.$refs.foodList;
        let el = foodList[index];
        // scrollToElement() 是better-scroll中的方法，滚动到某个元素 el（必填）表示 dom 元素，time 表示动画时间，offsetX 和 offsetY 表示坐标偏移量，easing 表示缓动函数
        this.foodsScroll.scrollToElement(el, 300);
      },
      selectFood(food, event) {
        if (!event._constructed) {
          return;
        }
        this.selectedFood = food;
        // this.$refs.food.show();
      },
      addFood(target, food) {
        this._drop(target);
      },
      _drop(target) {
        // 体验优化,异步执行下落动画 异步过程
        this.$nextTick(() => {
          this.$refs.shopcart.drop(target);
        });
      },
      _initScroll() {
        // 初始化scroll区域  ref 被用来给元素或子组件注册引用信息。引用信息将会注册在父组件的 $refs 对象上
        // 如果在普通的 DOM 元素上使用，引用指向的就是 DOM 元素; 如果用在子组件上，引用就指向组件实例:
        this.meunScroll = new BScroll(this.$refs.menuWrapper, {
        // 结合BScroll的接口使用,是否将click事件传递,默认被拦截了
          click: true
        });

        this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
          click: true,
          // 结合BScroll的接口使用,3实时派发scroll事件，探针的作用
          probeType: 3
        });
        // 结合BScroll的接口使用,监听scroll事件(实时派发的),并获取鼠标坐标，当滚动时能实时暴露出scroll
        this.foodsScroll.on('scroll', (pos) => {
          // 事件的回调函数 //滚动坐标会出现负的,并且是小数,所以需要处理一下，实时取得scrollY
          this.scrollY = Math.abs(Math.round(pos.y));
        });
      },
      // 通过 _calculateHeight 计算foods内部每一个块的高度,组成一个数组listHeight
      _calculateHeight() {
        // 获取每一个food的dom对象
        let foodList = this.$refs.foodList;
        let height = 0;
        // 初始化第一个高度为0
        this.listHeight.push(height);
        for (let i = 0; i < foodList.length; i++) {
          // 每一个item都是刚才获取的food的每一个dom
          let item = foodList[i];
          // 主要是为了获取每一个foods内部块的高度
          height += item.clientHeight;
          this.listHeight.push(height);
        }
      }
    },
    components: {
      shopcart,
      cartcontrol
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin.styl"

  .goods
    //display 属性规定元素应该生成的框的类型。
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
        .icon
          display: inline-block
          vertical-align: top
          width: 12px
          height: 12px
          margin-right: 2px
          background-size: 12px 12px
          background-repeat: no-repeat
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
        .text
          display table-cell
          width 56px
          //垂直居中
          vertical-align middle
          border-1px(rgba(7, 17, 27, 0.1))
          font-size 12px
    .foods-wrapper
      //让所有弹性盒模型对象的子元素都有相同的长度，且忽略它们内部的内容：
      flex 1
      .title
        padding-left 14px
        height 26px
        line-height 26px
        border-left 2px solid #d9dde1
        font-size 12px
        color rgb(147, 153, 159)
        background #f3f5f7
      .food-item
        display flex
        margin 18px
        padding-bottom 18px
        border-1px(rgba(7, 17, 27, 0.1))
        //最后一个子元素 去掉1px效果 去掉margin-bottom
        &last-child
          border-none()
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
          // 相同的可以写在一起
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
              margin-right 8px
              font-size 14px
              color rgb(240, 20, 20)
            .old
              //横线
              text-decoration line-through
              font-size 10px
              color rgb(147, 153, 159)
          .cartcontrol-wrapper
            position absolute
            right 0
            bottom 12px
</style>
