<template>
  <div>
    <div class="l-flex-wrap l-grid">
      <div style="width: 100%;" class="l-flex-vc l-title-hd">
        <h3>今日收入</h3>
        <font size="20" style="color: #808080; text-align: center">￥{{ info.today.income }}</font>
        <hr style="FILTER: progid:DXImageTransform.Microsoft.Shadow(color:#987cb9,direction:145,strength:15)" width="100%" color=#DCDCDC SIZE=1>
        <label style="color: #808080; margin-top: 10px">新增徒弟：{{ info.today.allStudents }}</label>
        <label style="color: #808080">有效徒弟：{{ info.today.validStus }}</label>
      </div>
    </div>
    <div class="l-flex-wrap l-grid">
      <div style="width: 100%;" class="l-flex-vc l-title-hd">
        <h3>昨日收入</h3>
        <font size="20" style="color: #808080; text-align: center">￥{{ info.yesterday.income }}</font>
        <hr style="FILTER: progid:DXImageTransform.Microsoft.Shadow(color:#987cb9,direction:145,strength:15)" width="100%" color=#DCDCDC SIZE=1>
        <label style="color: #808080; margin-top: 10px">新增徒弟：{{ info.yesterday.allStudents }}</label>
        <label style="color: #808080">有效徒弟：{{ info.yesterday.validStus }}</label>
      </div>
    </div>
    <div class="l-flex-wrap l-grid">
      <div style="width: 100%;" class="l-flex-vc l-title-hd">
        <h3>账户余额</h3>
        <font size="20" style="color: #808080; text-align: center">￥{{ info.account.income }}</font>
        <hr style="FILTER: progid:DXImageTransform.Microsoft.Shadow(color:#987cb9,direction:145,strength:15)" width="100%" color=#DCDCDC SIZE=1>
        <label style="color: #808080; margin-top: 10px">共引进徒弟：{{ info.account.allStudents }}</label>
        <label style="color: #808080">有效徒弟：{{ info.account.validStus }}</label>
        <button type="button" align="right" style="width: 25%; height:30px; background-color: #32CD32; font-size: 16px; color: white; margin-top: 10px; border: 1px solid #1E90FF" v-link="{path:'/product'}">我要提现</button>
      </div>
    </div>
    <div class="l-flex-wrap l-grid">
      <div style="width: 100%;" class="l-flex-vc l-title-hd">
        <h3>我的邀请</h3>
        <table style="margin-top: 10px; color: #808080;">
          <thead>
            <tr>
              <td>邀请大咖名</td>
              <td>今日奖励</td>
              <td>累计奖励</td>
              <td>邀请时间</td>
            </tr>
          </thead>
          <tbody>
            <tr v-for="invite in invites.list">
              <td>{{ invite.userName }}</td>
              <td>{{ invite.gold }}</td>
              <td>{{ invite.gold }}</td>
              <td>{{ invite.registerDate }}</td>
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
            <tr v-for="dayIncome in info.dayIncomeList">
              <td>{{ dayIncome.day }}</td>
              <td>{{ dayIncome.allStudents }}</td>
              <td>{{ dayIncome.validStus }}</td>
              <td>{{ dayIncome.income }}</td>
            </tr>
            <tr>
              <td>总计</td>
              <td>{{ info.dayIncomeSum.allStudentsSum }}</td>
              <td>{{ info.dayIncomeSum.validStusSum }}</td>
              <td>{{ info.dayIncomeSum.allIncomeSum }}</td>
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
      server.invites.getList(10).then( listEntity => {
        listEntity.callback = function(){
          self.invites.loading = !listEntity.isAjax
          self.invites.list = listEntity.alldata

          for (var i = self.invites.list.length - 1; i >= 0; i--) {
            self.invitesSum.todayIncomeSum += parseInt(self.invites.list[i].gold);
            self.invitesSum.allIncomeSum += parseInt(self.invites.list[i].gold);
          }
        }
        listEntity.init()
      })
    }
  },
  data() {
    return {
      invites: {
        list: [],
        loading: true
      },
      invitesSum: {
        todayIncomeSum: 0,
        allIncomeSum: 0
      },
      info: {
        today: {
          income: 99,
          allStudents: 88,
          validStus: 77
        },
        yesterday: {
          income: 66,
          allStudents: 55,
          validStus: 44
        },
        account: {
          income: 33,
          allStudents: 22,
          validStus: 11
        },
        dayIncomeList: [
          { day: '2018-02-01', allStudents: 19, validStus: 20, income: 100 },
          { day: '2018-01-31', allStudents: 19, validStus: 20, income: 100 },
          { day: '2018-01-30', allStudents: 19, validStus: 20, income: 100 },
          { day: '2018-01-29', allStudents: 19, validStus: 20, income: 100 },
          { day: '2018-01-28', allStudents: 19, validStus: 20, income: 100 },
          { day: '2018-01-27', allStudents: 19, validStus: 20, income: 100 },
          { day: '2018-01-26', allStudents: 19, validStus: 20, income: 100 }
        ],
        dayIncomeSum: {
          allStudentsSum: 133,
          validStusSum: 140,
          allIncomeSum: 700
        }
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
