<template>
<div class="header">
  <!-- 头 -->
  <div class="content-wrapper">
    <div class="avatar">
      <img width="64" height="64" :src="seller.avatar">
    </div>
    <div class="content">
      <div class="title">
        <span class="brand"></span>
        <span class="name">{{seller.name}}</span>
      </div>
      <div class="description">
        {{seller.description}}/{{seller.deliveryTime}}分钟送达
      </div>
      <div v-if="seller.supports" class="support">
        <!-- v-if判断： 因为初始化时 seller 为空-->
        <span class="icon" :class="classMap[seller.supports[0].type]"></span>
        <span class="text">
          {{seller.supports[0].description}}
        </span>
      </div>
    </div>
    <!-- 5个 -->
    <div v-if="seller.supports" class="support-count" @click="showDetail">
      <span class="count">{{seller.supports.length}}个</span>
      <i class="icon-keyboard_arrow_right"></i>
    </div>
  </div>
  <!--公告-->
  <div class="bulletin-wrapper" @click="showDetail">
    <span class="bulletin-title"></span><span class="bulletin-text">{{seller.bulletin}}</span>
    <i class="icon-keyboard_arrow_right"></i>
  </div>
  <!-- 背景-->
  <div class="background">
    <img :src="seller.avatar" alt="" width="100%" height="100%">
  </div>
  <!-- 弹出框-->
  <transition name="fade">
    <div class="detail"  v-show="detailShow">
      <div class="detail-wrapper clearfix">
        <div class="detail-main">
          <h1 class="name">{{seller.name}}</h1>
          <!-- 星-->
          <div class="star-wrapper">
            <star :size="48" :score="seller.score"></star>
          </div>

          <div class="title">
            <div class="line"></div>
            <div class="text">优惠信息</div>
            <div class="line"></div>
          </div>
          <ul v-if="seller.supports" class="supports">
            <li class="support-item" v-for="(item,index) in seller.supports">
              <span class="icon" :class="classMap[seller.supports[index].type]"></span>
              <span class="text">{{seller.supports[index].description}}</span>
            </li>
          </ul>

          <!-- 公告-->
          <div class="title">
            <div class="line"></div>
            <div class="text">商家公告</div>
            <div class="line"></div>
          </div>
          <div class="bulletin">
            <p class="content">{{seller.bulletin}}</p>
          </div>
        </div>
      </div>
      <!-- ×-->
      <div class="detail-close" @click="hideDetail">
        <i class="icon-close"></i>
      </div>

    </div>
  </transition>

</div>
</template>

<script type="text/ecmascript-6">
  import star from '../star/star.vue';

  export default {
    props: {
      seller: {
        type: Object
      }
    },
    created() {
      this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];
    },
    data() {
        return {
            detailShow:false
        };
    },
    methods: {
      showDetail() {
          this.detailShow = true;
//          alert(this.detailShow)
      },
      hideDetail() {
        this.detailShow = false;
      }
    },
    components: {
        star
    }
  }
</script>

<style lang="stylus">
@import "header.styl";
</style>
