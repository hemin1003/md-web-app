<template>
  <div>
    <group class="l-group l-user-info">
      <x-input class="l-input-arrow" title="支付宝账号" :value.sync="formData.account" placeholder="请填写" :show-clear="false" text-align="right"></x-input>
      <x-input class="l-input-arrow" title="真实姓名" :value.sync="formData.name" placeholder="请填写" :show-clear="false" text-align="right"></x-input>
      <x-input class="l-input-arrow" title="未提现余额" :value.sync="formData.webBalance" :disabled="true" placeholder="请填写" keyboard="number" :show-clear="false" text-align="right"></x-input>
      <x-input class="l-input-arrow" title="提现金额" :value.sync="formData.income" placeholder="请填写" keyboard="number" :show-clear="false" text-align="right"></x-input>
    </group>
    <div class="text">
      请仔细核对以上信息，最少提现为10元
    </div>
    <div class="l-btn-area">
      <x-button type="primary" @click="submit">提交</x-button>
      <!--<x-button type="primary" @click="submit">修改支付信息</x-button>-->
    </div>
    <div style="margin: 25px; font-size: 12px">
        <h3>结算说明</h3>
        <ol>
            <li>账户余额满10元即可申请提现。</li>
            <li>结算方式为每个月结算两次，每月15日24点之前的提现申请，于后一周的周一到周五打款。</li>
            <li>每月月底最后一天24点之前的提现申请，于后一周的周一到周五打款。</li>
            <li>提款申请可随时提交（一个月提交一次也没有问题），每半个月会自动处理一次，如无特殊必要，请集中一次提交，减少平台的审核工作量。</li>
            <li>平台支持支付宝转账，请按照要求完善个人提现信息。支付宝请提供支付宝账号，个人真实姓名。</li>
            <li>请完善个人提款账号信息，完善后方可提现，若由于提供错误提款信息导致的转账错误，由个人承担。</li>
        </ol>
    </div>
    <div class="l-flex-wrap l-grid">
      <div style="width: 100%;" class="l-flex-vc l-title-hd">
        <h3>账户情况</h3>
        <table style="margin-top: 10px; color: #808080;">
          <thead>
            <tr>
              <td>账户余额</td>
              <td>累计提现金额</td>
              <td>总收入</td>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td>{{userinfo.webBalance}}（不含审核中提现金额）</td>
              <td>{{userinfo.webWithdraw}}（含审核中提现金额）</td>
              <td>{{userinfo.webTotal}}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <div class="l-flex-wrap l-grid">
      <div style="width: 100%;" class="l-flex-vc l-title-hd">
        <h3>我的申请</h3>
        <table style="margin-top: 10px; color: #808080;">
          <thead>
            <tr>
              <td>申请时间</td>
              <td>申请金额</td>
              <td>实际提现金额</td>
              <td>状态</td>
            </tr>
          </thead>
          <tbody>
            <tr v-for="obj in withdrawHis.list">
              <td>{{ obj.withdrawTime }}</td>
              <td>{{ obj.income }}</td>
              <td>{{ obj.realIncome }}</td>
              <td>{{ obj.status }}</td>
            </tr>
            <tr>
              <td>总计</td>
              <td>{{ withdrawHisSum.incomeSum }}</td>
              <td>{{ withdrawHisSum.realIncomeSum }}</td>
              <td></td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <!--
    <div class="l-flex-wrap l-grid">
      <div class="l-flex-hc l-title-hd" v-link="{path: '/news/list'}">
        <h3 class="l-rest">账户情况</h3>
      </div>
    </div>
    <div class="l-flex-wrap l-grid">
      <div class="l-flex-hc l-title-hd" v-link="{path: '/news/list'}">
        <h3 class="l-rest">我的申请</h3>
      </div>
    </div>
    -->
  </div>
</template>

<script>
import { utils, storage } from 'assets/utils'
import { Group, XInput, Cell, XButton} from 'vux-components'
import name2value from 'vux/src/filters/name2value'
import value2name from 'vux/src/filters/value2name'
import { store, getters, actions } from '../vuex'
import config from '../config'
import server from '../server'

// 上一页
let prevPath = storage.session.get('_history').prevPath
export default {
  components: {
    Group, XInput, Cell, XButton
  },
  route: {
    data() {
      const self = this
      //获取最新个人信息
      let webPhoneNum = self.webPhoneNum;
      self.$http.post('owner/daka/getUserInfo', {
        webPhoneNum: webPhoneNum
      })
      .then(({ body }) => {
        if(body.success){
          storage.local.set('userinfo', body.data)
          self.acUpdateUserInfo()
          self.formData.webBalance = self.userinfo.webBalance
        }
      })

      //提现列表数据
      self.$vux.loading.show('数据加载中...')
      server.withdrawHis.getList(15, webPhoneNum).then( listEntity => {
        listEntity.callback = function(){
          self.$vux.loading.hide()
          self.withdrawHis.loading = !listEntity.isAjax
          self.withdrawHis.list = listEntity.alldata

          for (var i = self.withdrawHis.list.length - 1; i >= 0; i--) {
            self.withdrawHisSum.incomeSum += parseFloat(self.withdrawHis.list[i].income);
            if(self.withdrawHis.list[i].statusType == 2){
              self.withdrawHisSum.realIncomeSum += parseFloat(self.withdrawHis.list[i].realIncome);  
            }
          }
        }
        listEntity.init()
      })
    }
  },
  store,
  vuex: { getters, actions },
  data() {
    return {
      webPhoneNum: this.userinfo.webPhoneNum,
      defaultVal: config.defaultVal,
      selected: '',
      formData: {
        phoneNum: this.userinfo.webPhoneNum,
        account: this.userinfo.webAlipay,
        name: this.userinfo.webRealName,
        webBalance: this.userinfo.webBalance,
        income: '0'
      },
      withdrawHis: {
        list: [],
        loading: true
      },
      withdrawHisSum: {
        incomeSum: 0,
        realIncomeSum: 0
      }
    }
  },
  methods: {
    submit() {
      const self = this

      if(!self.formData.income){
        self.$vux.toptips.show('提现金额不能为空')
        return 
      }
      let incomeInt = parseFloat(self.formData.income);
      if(incomeInt < 10){
        self.$vux.toptips.show('提现金额不能低于10元')
        return 
      }
      let webBalanceInt = parseFloat(self.formData.webBalance);
      if(webBalanceInt - incomeInt < 0){
        self.$vux.toptips.show('余额不足')
        return 
      }

      self.$vux.loading.show('提交中')
      self.$http.post('owner/user/doWithdraw', self.formData)
        .then(({ body }) => {
          self.$vux.loading.hide()
          if(body.success){
            storage.local.set('userinfo', body.data)
            self.acUpdateUserInfo()
            self.$vux.toast.show({
              text: '申请成功',
              onHide(){
                self.$router.go('/home')
              }
            })
          }else{
            self.$vux.toptips.show(body.message || '申请失败')
          }
        }, (error) => {
          self.$vux.loading.hide()
          self.$vux.toptips.show('服务器繁忙，请稍后重试！')
        })
    }
  }
}
</script>
<style>
.l-user-info .weui_cell{
  padding-top:11px;
  padding-bottom:11px;
 }
.l-user-info .avatar{
  vertical-align: middle;
  border-radius: 2px;
  border:1px solid #ebebeb;
}

.l-input-arrow:after{
  content: " ";
  display: inline-block;
  transform: rotate(45deg);
  height: 6px;
  width: 6px;
  border-width: 2px 2px 0 0;
  border-color: #C8C8CD;
  border-style: solid;
  position: relative;
  top: -1px;
  margin-left: .6em;
}
.weui_cell_ft .vux-popup-picker-value+span{
  color: #aaa;
}
.vux-popup-picker-value{
  color: #333;
}
.text{
  text-align:center;
}

</style>
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
