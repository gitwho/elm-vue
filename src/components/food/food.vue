<template>
  <transition name="fade">
    <div v-show="showFlag" class="food">
      <div class="food-content">
        <div class="image-header">
          <img :src="food.image" alt="">
          <div class="back" @click="hide">
            <i class="icon-arrow_lift"></i>
          </div>
        </div>
        <div class="content">
          <h1 class="title">{{food.name}}</h1>
          <div class="detail">
            <span class="sell-count">月售{{food.sellCount}}份</span>
            <span class="rating"> 好评率{{food.rating}}%</span>
          </div>
          <div class="price">
            <span class="now">￥{{food.price}}</span>
            <span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
          </div>
          <div class="cartControl-wrapper">
            <cartControl :food="food"></cartControl>
          </div>
          <transition name="buy">
            <div class="buy" @click.stop.prevent="addFirst($event)" v-show="!food.count || food.count === 0">
              加入购物车
            </div>
          </transition>
        </div>
        <split></split>
        <div class="info" v-show="food.info">
          <h1 class="title">商品信息</h1>
          <p class="text">{{food.info}}</p>
        </div>
        <split></split>
        <div class="rating">
          <h1 class="title">商品评价</h1>
          <!-- 分类按钮-->
          <ratingselect @increment="incrementTotal" :select-type="selectType" :only-content="onlyContent" :desc="desc" :ratings="food.ratings"></ratingselect>

          <div class="rating-wrapper">
            <ul v-show="food.ratings && food.ratings.length">
              <li v-show="needShow(rating.rateType, rating.text)" class="rating-item border-1px"
                  v-for="rating in food.ratings">
                <div class="user">
                  <span class="name">{{rating.username}}</span>
                  <img width="12" height="12" :src=rating.avatar alt="" class="avatar">
                </div>
                <div class="time">{{rating.rateTime | formatDate}}</div>
                <p class="text">
                  <span :class="{'icon-thumb_up':rating.rateType===0,'icon-thumb_down':rating.rateType===1}"></span>{{rating.text}}
                </p>
              </li>
            </ul>
            <div class="no-rating" v-show="!food.ratings || !food.ratings.length">暂无评价</div>
          </div>
        </div>

      </div>
    </div>
  </transition>

</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll';
  import cartControl from '../cartControl/cartControl.vue';
  import split from '../split/split.vue';
  import ratingselect from '../ratingselect/ratingselect.vue';
  import Vue from 'vue';
  import {formatDate} from '../../common/js/date';

  const ALL = 2;

  export default {
    components: {
      cartControl,split,ratingselect
    },
    props: {
        food: {
            type: Object
        }
    },
    data() {
        return {
          showFlag: false,
          selectType: ALL,
          onlyContent: true,
          desc: {
            all: '全部',
            positive: '推荐',
            negative: '吐槽'
          }
        };
    },
    methods: {
        show() {
            this.showFlag = true;
            this.selectType = ALL;
            this.onlyContent = true;
            this.$nextTick(() => {
                if(!this.scroll) {
                    this.scroll = new BScroll(this.$el, {
                        click: true
                    })
                }else {
                    this.scroll.refresh();
                }
            })
        },
        hide() {
            this.showFlag = false;
        },
        addFirst(event) {
            if(!event._constructed){
                return ;
            }
          //开发事件，传入参数，传递给父组件 goods.vue
          this.$emit('cart.add',event.target);
          //引入vue，设置this.food没有的属性，用set才能被观测到，通知dom发生改变
          Vue.set(this.food, 'count', 1);
        },
        needShow(type, text) {
            if(this.onlyContent && !text){
                return false;
            }
            if(this.selectType === ALL){
                return true;
            }else{
                return type === this.selectType;
            }
        },
// ??????????
        incrementTotal(type, data) {
          this[type] = data;
          this.$nextTick(() => {
            this.scroll.refresh();
          });
        }
       /* ratingTypeSelect(data){
//            this[type]= data;
            this.selectType= data;//改变selectType的值，needShow会判断,控制li显示隐藏
            this.$nextTick(() => {
                this.scroll.refresh();
            })
        },
        contentToggle(bool){
            this.onlyContent = bool;
            this.$nextTick(() => {
              this.scroll.refresh();
            })
        }*/
    },
    filters: {
        formatDate(time) {
            let date = new Date(time);
            return formatDate(date, 'yyyy-MM-dd hh:mm');
        }
    }
  }
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "food.styl";

</style>
