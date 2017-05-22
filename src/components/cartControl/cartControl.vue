<template>
  <div class="cartControl">
    <transition name="fade">
      <div class="cart-decrease inner icon-remove_circle_outline" v-show="food.count>0" @click.stop.prevent="decreaseCart($event)">
        <!--<transition name="inner">-->
          <!--<span class="inner icon-remove_circle_outline"></span>-->
        <!--</transition>-->
      </div>
    </transition>

    <div class="cart-count" v-show="food.count>0">{{food.count}}</div>
    <div class="cart-add icon-add_circle" @click.stop.prevent="addCart($event)"></div>
  </div>
</template>

<script type="text/ecmascript-6">
  import Vue from 'vue'

  export default {
      props: {
          food: {
              type: Object
          }
      },
      created() {
//          console.log(this.food)
      },
      methods: {
          addCart(event) {
              if(!event._constructed){//说明是自己开发的:// 去掉自带click事件的点击
                  return ;
              }

              if(!this.food.count){
                  //引入vue，设置this.food没有的属性，用set才能被观测到，通知dom发生改变
                Vue.set(this.food, 'count', 1);
                this.food.count = 1;
              }else{
                this.food.count++
              }
//              console.log(event.target)
            //开发事件，传入参数，传递给父组件 goods.vue
              this.$emit('cartAdd',event.target);
          },
          decreaseCart(event){
            if (!event._constructed) {
              return;
            }
            if (this.food.count) {
              this.food.count--;
            }
          },

      }
  }
</script>
<style lang="stylus" rel="stylesheet/stylus">
  @import "cartControl.styl";
</style>
