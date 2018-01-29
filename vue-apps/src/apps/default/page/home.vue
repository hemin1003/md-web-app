<template>
  <div>
    <div class="l-flex-wrap l-grid">
      <div class="l-flex-hc l-title-hd" v-link="{path: '/news/list'}">
        <h3 class="l-rest">今日收入</h3>
      </div>
    </div>
    <div class="l-flex-wrap l-grid">
      <div class="l-flex-hc l-title-hd" v-link="{path: '/news/list'}">
        <h3 class="l-rest">昨日收入</h3>
      </div>
    </div>
    <div class="l-flex-wrap l-grid">
      <div class="l-flex-hc l-title-hd" v-link="{path: '/news/list'}">
        <h3 class="l-rest">账号余额</h3>
      </div>
    </div>
    <div class="l-flex-wrap l-grid">
      <div class="l-flex-hc l-title-hd" v-link="{path: '/news/list'}">
        <h3 class="l-rest">我的邀请</h3>
      </div>
    </div>
    <div class="l-flex-wrap l-grid">
      <div class="l-flex-hc l-title-hd" v-link="{path: '/news/list'}">
        <h3 class="l-rest">分日数据</h3>
      </div>
    </div>
    <div class="l-flex-wrap l-grid">
      <div class="l-flex-hc l-title-hd" v-link="{path: '/news/list'}">
        <h3 class="l-rest">分日数据趋势</h3>
      </div>
    </div>
  </div>
</template>
<script>
import { Swiper, Masker, Flexbox, FlexboxItem } from 'vux-components'
import NewsList from '../components/news-list'
import config from '../config'
import server from '../server'

export default {
  components: {
    Swiper, Masker, Flexbox, FlexboxItem, NewsList
  },
  route: {
    data() {
      const self = this
      server.news.getList(3).then( listEntity => {
        listEntity.callback = function(){
          self.news.loading = !listEntity.isAjax
          self.news.list = listEntity.alldata
        }
        listEntity.init()
      })
    }
  },
  data() {
    return {
      images: {
        img1: require('assets/imgs/booking.png'),
        img2: require('assets/imgs/fuli.png')
      },
      banner: [{
        url: 'javascript:;',
        img: require('assets/imgs/temp-016.jpg'),
        title: ''
      }],
      news: {
        list: [],
        loading: true
      }
    }
  }
}
</script>
<style scoped lang="less">
.l-grid{
  margin: 0.75rem 0;
  background-color: #fff;
  font-size: 12px;
  padding: 0.375rem 0;
}
.l-grid-item{
  box-sizing: border-box;
  width: 25%;
  padding: 0.375rem;
  img{
    width: 80%;
  }
  p{
    margin-top: 5px;
  }
}
.l-masker-item{
  border-radius: 5px;
  .l-img{
    padding-bottom: 33%;
    display: block;
    position: relative;
    max-width: 100%;
    background-size: cover;
    background-position: center center;
    cursor: pointer;
    border-radius: 2px;
  }
  .l-title {
    color: #fff;
    text-align: center;
    text-shadow: 0 0 2px rgba(0,0,0,.5);
    font-weight: 500;
    font-size: 16px;
    position: absolute;
    left: 0;
    right: 0;
    width: 100%;
    text-align: center;
    top: 50%;
    transform: translateY(-50%);
  }
}
</style>
