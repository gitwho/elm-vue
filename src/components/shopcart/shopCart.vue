<template>
  <div>
    <div class="shopcart">
      <div class="content" @click="toggleList">
        <div class="content-left">
          <div class="logo-wrapper">
            <div class="logo" :class="{'highlight':totalCount>0}">
              <span class="icon-shopping_cart" :class="{'highlight':totalCount>0}"></span>
            </div>
            <div class="num" v-show="totalCount>0">{{totalCount}}</div>
          </div>
          <div class="price">￥{{totalPrice}}</div>
          <div class="desc">另需配送费￥{{deliveryPrice}}元</div>
        </div>
        <div class="content-right">
          <div class="pay" :class="payClass">
            {{payDesc}}
          </div>
        </div>
      </div>
      <div class="ball-container">
        <div v-for="ball in balls">
          <transition name="drop">
            <div v-show="ball.show" class="ball">
              <div class="inner inner-hook"></div>
            </div>
          </transition>
        </div>
      </div>
      <transition name="fold">
        <div class="shopcart-list" v-show="listShow">
          <div class="list-header">
            <h1 class="title">购物车</h1>
            <span class="empty" @click="empty">清空</span>
          </div>
          <div class="list-content" ref="listContent">
            <ul>
              <li class="food" v-for="food in selectFoods">
                <span class="name">{{food.name}}</span>
                <div class="price">
                  <span>￥{{food.price * food.count}}</span>
                </div>
                <div class="cartControl-wrapper">
                  <cartControl :food="food"></cartControl>
                </div>
              </li>
            </ul>
          </div>
        </div>
      </transition>
    </div>
    <transition name="fade">
      <div class="list-mask" v-show="listShow" @click="hideList()"></div>
    </transition>
  </div>
</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll'
  import cartControl from '../cartControl/cartControl.vue';

  export default{
      components: {
        BScroll,cartControl
      },
      props: {
        deliveryPrice: {
            type: Number,
            default: 0
        },
        minPrice: {
          type: Number,
          default: 0
        },
        selectFoods: {
            type: Array,
            default() {//object array 写成函数
                return [];
            }
        }
      },
      data() {
          return {
              balls: [{show: false},{show:false},{show:false},{show:false},{show: false}],
              dropBall: [],
              fold: true
          }
      },
      computed: {
          totalPrice() {
              let total = 0;
              this.selectFoods.forEach((food) => {
                  total += food.price * food.count;
              })
              return total;
          },
          totalCount() {
              let count = 0;
              this.selectFoods.forEach((food) => {
                  count += food.count;
              })
              return count;
          },
          payDesc() {
              if(this.totalPrice === 0){
                  return `￥${this.minPrice}元起送`;
              }else if(this.totalPrice<this.minPrice){
                  let diff = this.minPrice - this.totalPrice;
                  return `还差${diff}元起送`;
              }else {
                  return '去结算';
              }
          },
          payClass() {
            if (this.totalPrice < this.minPrice) {
              return 'not-enough';
            } else {
              return 'enough';
            }
          },
          listShow() {
              console.log(this.totalCount)
              if(!this.totalCount){
                  this.fold = true;
                  return false;
              }
              let show = !this.fold;

              if(show){
                  if(!this.scroll){
                    this.$nextTick(() => {
                      this.scroll = new BScroll(this.$refs.listContent,{
                        click:true
                      });
                    })
                  }else{
                      this.scroll.refresh();
                  }
              }
              return show;
          }
      },
      methods: {
          toggleList() {
            if(!this.totalCount){
                return;
            }
            this.fold = !this.fold;
          },
          empty(){
              this.selectFoods.forEach((food) => {
                  food.count = 0;
              });
          },
          //drop方法
        drop(el){
//            console.log(el);
            for(let i=0;i<this.balls.length;i++){
                let ball = this.balls[i];
                if(!ball.show){
                    ball.show = true;//触发 动画滚动
                    ball.el = el;
                    this.dropBall.push(ball);
                    return;
                }
            }
        },
        beforeEnter(el){
            console.log('bf')
            let count = this.balls.length;
            while (count--) {
                let ball = this.balls[count];
                if(ball.show) {
                    let rect = ball.el.getBoundingClientRect();
                    console.log( ball.el )
                    let x = rect.left - 32;
                    let y = -(window.innerHeight - rect.top -22);
                    el.style.display = '';//显示
                    el.style.webkitTransform = `translate3d(0,${y}px,0)`;
                    el.style.transform = `translate3d(0,${y}px,0)`;
                    let inner = el.getElementsByClassName('inner-hook')[0];
                    inner.style.webkitTransform = `translate3d(${x}px,0,0)`;
                    inner.style.transform = `translate3d(${x}px,0,0)`;
                }
            }
        },
        enter(el){
          this.$nextTick(() => {
            el.style.webkitTransform = 'translate3d(0,0,0)';
            el.style.transform = 'translate3d(0,0,0)';
            let inner = el.getElementsByClassName('inner-hook')[0];
            inner.style.webkitTransform = 'translate3d(0,0,0)';
            inner.style.transform = 'translate3d(0,0,0)';
          });
        },
        afterEnter(el){
          let ball = this.dropBall.shift();
          if (ball) {
            ball.show = false;
            el.style.display = 'none';
          }
        }
      }
  }
</script>

<style lang="stylus" rel="stylesheet/stylus">
@import "shopCart.styl";
</style>
