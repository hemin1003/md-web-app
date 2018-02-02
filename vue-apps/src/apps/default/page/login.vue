<template>
  <div>
    <div class="vux-center l-fsize-l" style="height: 3.2rem;">
      使用账号和密码登录
    </div>
    <group class="weui_cells_form">
      <x-input title="账号" placeholder="手机号码" keyboard="number" is-type="china-mobile" :value.sync="formData.webPhoneNum" ></x-input>
      <x-input type="password" title="密码" placeholder="请填写密码" :value.sync="formData.webUserPwd"  :required="true"></x-input>
    </group>
    <div class="l-btn-area">
      <x-button type="primary" @click="submit">登录</x-button> 
    </div>
  </div>
</template>
<script>
import { utils, storage } from 'assets/lib'
import { Group, XInput, XButton } from 'vux-components'
import { store, actions } from '../vuex'
import config from '../config'
import server from '../server'

// 上一页
let prevPath = storage.session.get('_history').prevPath
export default {
  components: {
    Group, XInput, XButton
  },
  store,
  vuex: {
    actions
  },
  data() {
    return {
      formData: {
        webPhoneNum: '',
        webUserPwd: ''
      }
    }
  },
  methods: {
    submit() {
      let self = this
      if(!self.formData.webPhoneNum){
        self.$vux.toptips.show('账号不能为空')
        return  
      }
      if(!self.formData.webUserPwd){
        self.$vux.toptips.show('密码不能为空')
        return  
      }

      self.$vux.loading.show('登录中')
      self.$http.post('owner/user/doLogin', self.formData)
        .then(({ body }) => {
          self.$vux.loading.hide()
          if(body.success){
            storage.local.set('userinfo', body.data)
            self.acUpdateUserInfo()
            self.$router.replace( prevPath ? prevPath.indexOf('/register') === 0 ? '/' : prevPath : '/' )
          }else{
            self.$vux.toptips.show(body.message || '登录失败')
          }
        }, (error) => {
          self.$vux.loading.hide()
          self.$vux.toptips.show('服务器繁忙，请稍后重试！')
        })
    }
  }
}
</script>
