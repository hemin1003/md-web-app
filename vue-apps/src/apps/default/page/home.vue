<template>
  <div>
    <div class="l-flex-wrap l-grid">
      <div style="width: 100%;" class="l-flex-vc l-title-hd">
        <h3>今日收入</h3>
        <font size="20" style="color: #808080; text-align: center">￥{{ info.today.seeIncome }}</font>
        <hr style="FILTER: progid:DXImageTransform.Microsoft.Shadow(color:#987cb9,direction:145,strength:15)" width="100%" color=#DCDCDC SIZE=1>
        <label style="color: #808080; margin-top: 10px">新增徒弟：{{ info.today.newStus }}</label>
        <label style="color: #808080">有效徒弟：{{ info.today.seeStus }}</label>
      </div>
    </div>
    <div class="l-flex-wrap l-grid">
      <div style="width: 100%;" class="l-flex-vc l-title-hd">
        <h3>昨日收入</h3>
        <font size="20" style="color: #808080; text-align: center">￥{{ info.yesterday.seeIncome }}</font>
        <hr style="FILTER: progid:DXImageTransform.Microsoft.Shadow(color:#987cb9,direction:145,strength:15)" width="100%" color=#DCDCDC SIZE=1>
        <label style="color: #808080; margin-top: 10px">新增徒弟：{{ info.yesterday.newStus }}</label>
        <label style="color: #808080">有效徒弟：{{ info.yesterday.seeStus }}</label>
      </div>
    </div>
    <div class="l-flex-wrap l-grid">
      <div style="width: 100%;" class="l-flex-vc l-title-hd">
        <h3>账户余额</h3>
        <font size="20" style="color: #808080; text-align: center">￥{{ userinfo.webBalance }}</font>
        <hr style="FILTER: progid:DXImageTransform.Microsoft.Shadow(color:#987cb9,direction:145,strength:15)" width="100%" color=#DCDCDC SIZE=1>
        <label style="color: #808080; margin-top: 10px">总新增徒弟：{{ userinfo.webSumStus }}</label>
        <label style="color: #808080">总有效徒弟：{{ userinfo.webValidStusSee }}</label>
        <button type="button" align="right" style="width: 25%; height:30px; background-color: #32CD32; font-size: 16px; color: white; margin-top: 10px; border: 1px solid #1E90FF" v-link="{path:'/product'}">我要提现</button>
      </div>
    </div>
    <div class="l-flex-wrap l-grid">
      <div style="width: 100%;" class="l-flex-vc l-title-hd">
        <h3>我的邀请</h3>
        <table style="margin-top: 10px; color: #808080;">
          <thead>
            <tr>
              <td>邀请大咖</td>
              <td>今日奖励</td>
              <td>累计奖励</td>
              <td>邀请时间</td>
            </tr>
          </thead>
          <tbody>
            <tr v-for="invite in invites.list">
              <td>{{ invite.invitedPhoneNum }}</td>
              <td>{{ invite.todayAward }}</td>
              <td>{{ invite.totalAward }}</td>
              <td>{{ invite.createDate }}</td>
            </tr>
            <tr>
              <td>总计</td>
              <td>{{ invitesSum.todayIncomeSum }}</td>
              <td>{{ invitesSum.allIncomeSum }}</td>
              <td></td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <div class="l-flex-wrap l-grid">
      <div style="width: 100%;" class="l-flex-vc l-title-hd">
        <h3>分日数据</h3>
        <table style="margin-top: 10px; color: #808080;">
          <thead>
            <tr>
              <td>日期</td>
              <td>新增徒弟</td>
              <td>新增有效徒弟</td>
              <td>收入</td>
            </tr>
          </thead>
          <tbody>
            <tr v-for="studentDay in studentDays.list">
              <td>{{ studentDay.date }}</td>
              <td>{{ studentDay.newStus }}</td>
              <td>{{ studentDay.seeStus }}</td>
              <td>{{ studentDay.seeIncome }}</td>
            </tr>
            <tr>
              <td>总计</td>
              <td>{{ studentDaysSum.newStusSum }}</td>
              <td>{{ studentDaysSum.seeStusSum }}</td>
              <td>{{ studentDaysSum.seeIncomeSum }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <!--
    <div class="l-flex-wrap l-grid">
      <div style="width: 100%;" class="l-flex-vc l-title-hd">
        <h3>分日数据趋势</h3>
        <table style="margin-top: 10px; color: #808080;">
          <thead>
            <tr>
              <td>日期</td>
              <td>新增徒弟</td>
              <td>新增有效徒弟</td>
              <td>收入</td>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td>总计</td>
              <td>0</td>
              <td>0</td>
              <td>0</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    -->
  </div>
</template>
<script>
import { utils, storage } from 'assets/lib'
import { Swiper, Masker, Flexbox, FlexboxItem } from 'vux-components'
import NewsList from '../components/news-list'
import config from '../config'
import server from '../server'
import name2value from 'vux/src/filters/name2value'
import value2name from 'vux/src/filters/value2name'
import { store, getters, actions } from '../vuex'

export default {
  components: {
    Swiper, Masker, Flexbox, FlexboxItem, NewsList
  },
  route: {
    data() {
      const self = this
      let webPhoneNum = self.webPhoneNum;

      //邀请奖励数据
      server.invites.getList(30, webPhoneNum).then( listEntity => {
        listEntity.callback = function(){
          self.invites.loading = !listEntity.isAjax
          self.invites.list = listEntity.alldata

          for (var i = self.invites.list.length - 1; i >= 0; i--) {
            self.invitesSum.todayIncomeSum += parseFloat(self.invites.list[i].todayAward);
            self.invitesSum.allIncomeSum += parseFloat(self.invites.list[i].totalAward);
          }
        }
        listEntity.init()
      })

      //徒弟分日数据
      let currentDay = CurentTimeDay(0);
      let yesterDay = CurentTimeDay(-1);

      self.$vux.loading.show('数据加载中...')
      server.studentDays.getList(15, webPhoneNum).then( listEntity => {
        listEntity.callback = function(){
          self.$vux.loading.hide()
          self.studentDays.loading = !listEntity.isAjax
          self.studentDays.list = listEntity.alldata

          for (var i = self.studentDays.list.length - 1; i >= 0; i--) {
            self.studentDaysSum.newStusSum += parseFloat(self.studentDays.list[i].newStus);
            self.studentDaysSum.seeStusSum += parseFloat(self.studentDays.list[i].seeStus);
            self.studentDaysSum.seeIncomeSum += parseFloat(self.studentDays.list[i].seeIncome);
            //今日收入
            if(self.studentDays.list[i].date == currentDay){
              self.info.today.seeIncome = self.studentDays.list[i].seeIncome;
              self.info.today.newStus = self.studentDays.list[i].newStus;
              self.info.today.seeStus = self.studentDays.list[i].seeStus;
            }
            //昨日收入
            if(self.studentDays.list[i].date == yesterDay){
              self.info.yesterday.seeIncome = self.studentDays.list[i].seeIncome;
              self.info.yesterday.newStus = self.studentDays.list[i].newStus;
              self.info.yesterday.seeStus = self.studentDays.list[i].seeStus;
            }
          }
        }
        listEntity.init()
      })

      //获取最新个人信息
      self.$http.post('owner/daka/getUserInfo', {
        webPhoneNum: webPhoneNum
      })
      .then(({ body }) => {
        if(body.success){
          storage.local.set('userinfo', body.data)
          self.acUpdateUserInfo()
        }
      })
    }
  },
  store,
  vuex: { getters, actions },
  data() {
    return {
      webPhoneNum: this.userinfo.webPhoneNum,
      invites: {
        list: [],
        loading: true
      },
      invitesSum: {
        todayIncomeSum: 0,
        allIncomeSum: 0
      },
      studentDays: {
        list: [],
        loading: true
      },
      studentDaysSum: {
        newStusSum: 0,
        seeStusSum: 0,
        seeIncomeSum: 0
      },
      info: {
        today: {
          seeIncome: 0,
          newStus: 0,
          seeStus: 0
        },
        yesterday: {
          seeIncome: 0,
          newStus: 0,
          seeStus: 0
        }
      }
    }
  }
}


function CurentTimeDay(day){ 
  var now = new Date();
  var year = now.getFullYear();       //年
  var month = now.getMonth() + 1;     //月
  var day = now.getDate() + day;            //日
  var hh = now.getHours();            //时
  var mm = now.getMinutes();          //分
  var ss = now.getSeconds();           //秒
  var clock = year + "-";
  if(month < 10)
      clock += "0";
  
  clock += month + "-";
  
  if(day < 10)
      clock += "0";
      
  clock += day;
  return(clock); 
}

function CurentTime(){ 
  var now = new Date();
  var year = now.getFullYear();       //年
  var month = now.getMonth() + 1;     //月
  var day = now.getDate();            //日
  var hh = now.getHours();            //时
  var mm = now.getMinutes();          //分
  var ss = now.getSeconds();           //秒
  var clock = year + "-";
  if(month < 10)
      clock += "0";
  
  clock += month + "-";
  
  if(day < 10)
      clock += "0";
      
  clock += day + " ";
  
  if(hh < 10)
      clock += "0";
      
  clock += hh + ":";
  if (mm < 10) clock += '0'; 
  clock += mm + ":"; 
   
  if (ss < 10) clock += '0'; 
  clock += ss; 
  return(clock); 
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
