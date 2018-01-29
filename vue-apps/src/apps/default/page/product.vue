<template>
  <div>
    <group class="l-group l-user-info">
      <x-input class="l-input-arrow" title="提现方式" :value.sync="" placeholder="请填写" :show-clear="false" text-align="right"></x-input>
      <x-input class="l-input-arrow" title="微信号/支付宝账号" :value.sync="" placeholder="请填写" :show-clear="false" text-align="right"></x-input>
      <x-input class="l-input-arrow" title="真实姓名" :value.sync="formData.qq1" placeholder="请填写" keyboard="number" :show-clear="false" text-align="right"></x-input>
      <x-input class="l-input-arrow" title="未提现余额" :value.sync="formData.qq2" placeholder="请填写" keyboard="number" :show-clear="false" text-align="right"></x-input>
      <x-input class="l-input-arrow" title="提现金额" :value.sync="formData.qq3" placeholder="请填写" keyboard="number" :show-clear="false" text-align="right"></x-input>
    </group>
    <div class="text">
      请仔细核对以上信息，最少提现为10元
    </div>
    <div class="l-btn-area">
      <x-button type="primary" @click="submit">提交</x-button>
      <x-button type="primary" @click="submit">修改支付信息</x-button>
    </div>
    <div>
      <div>
        结算说明：
        <div>1. 账户余额满10元即可申请提现</div>
        <div>2. 结算方式为周结，每周周五晚24点前提交的提款申请，平台将于下周周一到周五进行处理，陆续打款</div>
      </div>
    </div>
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
  </div>
</template>

<script>
import { utils, storage } from 'assets/utils'
import { Group, XInput, Cell, XButton, Address, AddressChinaData } from 'vux-components'
import name2value from 'vux/src/filters/name2value'
import value2name from 'vux/src/filters/value2name'
import { store, getters, actions } from '../vuex'
import config from '../config'

export default {
  components: {
    Group, XInput, Cell, XButton, Address, AddressChinaData
  },
  route: {
    data() {
      let valueArr = []
      this.userinfo.provinceId && (valueArr[0] = this.userinfo.provinceId + '')
      this.userinfo.cityId && (valueArr[1] = this.userinfo.cityId + '')
      this.userinfo.areaId && (valueArr[2] = this.userinfo.areaId + '')
      this.address.value = valueArr
    }
  },
  store,
  vuex: { getters, actions },
  data() {
    return {
      defaultVal: config.defaultVal,
      formData: {
        photo: this.userinfo.photo,
        realName: this.userinfo.realName,
        email: this.userinfo.email,
        qq: this.userinfo.qq,
        companyName: this.userinfo.companyName,
        address: this.userinfo.address
      },
      address: {
        data: AddressChinaData,
        value: [],
        name: []
      }
    }
  },
  methods: {
    submit() {
      const self = this
      self.formData.id = self.userinfo.id
      if(self.address.value){
        self.formData.provinceId = self.address.value[0] || ''
        self.formData.cityId = self.address.value[1] || ''
        self.formData.areaId = self.address.value[2] || ''

        self.address.name = value2name(self.address.value, AddressChinaData).split(' ')
        self.formData.province = self.address.name[0] || ''
        self.formData.city = self.address.name[1] || ''
        self.formData.area = self.address.name[2] || ''
      }

      self.$vux.loading.show('保存中')
      self.$http.post('owner/modifyMemPersonalInfo', self.formData)
        .then(({ body }) => {
          self.$vux.loading.hide()
          if(body.success){
            storage.local.set('userinfo', Object.assign(self.userinfo, self.formData))
            self.acUpdateUserInfo()
            self.$vux.toast.show({
              text: '保存成功',
              onHide(){
                self.$router.go('/user')
              }
            })
          }else{
            self.$vux.toptips.show(body.message || '保存失败')
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
