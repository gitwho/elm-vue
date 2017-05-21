<template>
  <div class="goods">
    <!-- 左-->
    <div class="menu-wrapper" ref="menuWrapper">
      <ul>
        <li v-for="(item,index) in goods" class="menu-item border-1px" :class="{'current': currentIndex === index}"
            @click="selectMenu(index,$event)" >
          <span class="text border-1px">
            <span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>{{item.name}}
          </span>
        </li>
      </ul>
    </div>
    <!-- 右-->
    <div class="foods-wrapper" ref="foodsWrapper">
      <ul>
        <li v-for="item in goods" class="food-list food-list-hook">
          <h1 class="title">{{item.name}}</h1>
          <ul>
            <li @click="selectFood(food,$event)" v-for="food in item.foods" class="food-item border-1px">
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
                  <span class="now">￥{{food.price}}</span><span class="old"
                                                                v-show="food.oldPrice">￥{{food.oldPrice}}</span>
                </div>
                <div class="cartcontrol-wrapper">
                  <cartControl @cart.add="cartAdd" :food="food"></cartControl>
                </div>
              </div>
            </li>
          </ul>
        </li>
      </ul>
    </div>
    <!-- 底部组件-->
    <div>
      <shopCart ref="shopCart" :select-foods="selectFoods" :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice"></shopCart>
      <!-- 商品详情-->
      <food :food="selectedFood" ref="food"></food>
    </div>
  </div>

</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll'
  import shopCart from '../shopcart/shopCart.vue'
  import cartControl from '../cartControl/cartControl.vue'
  import food from '../food/food.vue';

  const ERR_OK = 0;
  export default {
    components: {
      shopCart,cartControl,food
    },
    props: {
      seller: {
        type: Object
      }
    },
    data() {
      return {
        goods: [],
        listHeight: [],
        scrollY: 0,
        selectedFood: {}
      }
    },
    created() {
      this.$http.get('/api/goods').then((response) => {
        response = response.body;
        if (response.errno === ERR_OK) {
          this.goods = response.data;
          //dom改变在这个方法后
          this.$nextTick(() => {
            this._initScroll();//初始化滚动
            this._calculateHeight();
          })

        }
      });
      this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];
    },
    computed: {
      currentIndex() {
        for (let i = 0; i < this.listHeight.length; i++) {
          let height = this.listHeight[i];
          let height2 = this.listHeight[i + 1];
          if (!height2 || (this.scrollY >= height && this.scrollY < height2)) {
            return i;
          }
        }
        return 0;
      },
      selectFoods() {
          let foods = [];
          this.goods.forEach((good) => {
              good.foods.forEach((food) => {
                  if(food.count){ //判断是否点击过
                      foods.push(food);
                  }
              })
          });
          return foods;
      }
    },
    methods: {
      _initScroll() {
        //console.log(this.$refs.menuWrapper)
  //          初始化滚动
        this.menuScroll = new BScroll(this.$refs.menuWrapper,{
          click: true//监听了点击事件
        });
        this.foodScroll = new BScroll(this.$refs.foodsWrapper,{
          probeType: 3,//表示显示当前位置
          click: true
        });

        this.foodScroll.on('scroll', (pos) => {
          this.scrollY = Math.abs(Math.round(pos.y));
          //console.log(pos)//回调函数返回当前x,y坐标

        })
      },
      _calculateHeight() {
        let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook');
        //console.log(foodList)

        //console.log(this.$el)//整个goods的html
        let height = 0;
        this.listHeight.push(height);
        for(let i = 0; i < foodList.length; i++){
          let item = foodList[i];
          height += item.clientHeight;
          this.listHeight.push(height);
        }
      },

      selectMenu(index, event) {
        if(!event._constructed){
          return ;
        }
        let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook');
        let el = foodList[index];
        this.foodScroll.scrollToElement(el,300);//BScroll方法
        console.log(index)
      },
      selectFood(food, event){
          if(!event._constructed){
              return ;
          }
          this.selectedFood = food;
          this.$refs.food.show();
      },
      cartAdd(target) {
        this.$refs.shopCart.drop(target);
      },
      //定义下落方法
      _drop(target){
          this.$nextTick(() => {//体验优化，异步执行
            //调用shopCart.vue drop方法
//        访问子组件： this.$refs.shopCart
            this.$refs.shopCart.drop(target);
          })

      }
    },
    events: {
//        接收子组件cartControl.vue的事件，并调用 _drop
        'cart.add'(target){
            this._drop(target);
        }
    }
  }
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "goods.styl";
</style>
