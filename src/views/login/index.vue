<template>
  <div class="login-container">
      <!-- 导航栏 -->
  <van-nav-bar class="page-nav-bar" title="登录"/>
      <!-- 表单 -->
<van-form ref="loginForm" @submit="onSubmit">
  <van-field
    v-model="user.mobile"
    name="mobile"
    placeholder="请输入手机号"
    type="number"
    maxlength="11"
    :rules="userFormRules.mobile"
  >
  <i slot="left-icon" class="toutiao toutiao-shouji"></i>
  </van-field>
  <van-field
    v-model="user.code"
    name="code"
    placeholder="请输入验证码"
    type="number"
    maxlength="6"
    :rules="userFormRules.code"
  >
  <i slot="left-icon" class="toutiao toutiao-yanzhengma"></i>
  <template #button>
      <van-count-down v-if="isCountDownShow" :time="1000 * 10" format="ss s" @finish="isCountDownShow = false"/>
    <van-button v-else class="send-sms-btn" round size="small" type="default" @click="onSendSms" native-type="button">发送验证码</van-button>
  </template>
  </van-field>
  <div class="login-btn-wrap">
    <van-button class="login-btn" block type="info" native-type="submit" :disabled="false">登录</van-button>
  </div>
</van-form>
  </div>
</template>

<script>
import { login } from '@/api/user'
export default {
  name: 'LoginIndex',
  data () {
    return {
      user: {
        mobile: '13911111111',
        code: '246810'
      },
      userFormRules: {
        mobile: [{ required: true, message: '请输入手机号' }, { pattern: /1[3|5|7|8]\d{9}/, message: '手机号格式错误' }],
        code: [{ required: true, message: '请输入验证码' }, { pattern: /\d{6}/, message: '验证码格式错误' }]
      },
      isCountDownShow: false
    }
  },
  methods: {
    async onSubmit () {
      const user = this.user
      this.$toast.loading({
        message: '登录中...',
        forbidClick: true,
        duration: 2000
      })
      try {
        const { data } = await login(user)
        this.$store.commit('setUser', data.data)
        this.$toast.success('登录成功')
      } catch (err) {
        if (err.response.status === 400) {
          this.$toast.fail('手机验证码错误')
        } else { this.$toast.fail('shibai', err) }
      }
    },
    async onSendSms () {
      try {
        await this.$refs.loginForm.validate('mobile')
        this.$toast.success('验证通过')
      } catch (err) {
        return this.$toast.fail('验证失败')
      }
      this.isCountDownShow = true
      try {
        await this.sendSms(this.user.mobile)
        this.$toast.success('发送成功')
      } catch (err) {
        this.isCountDownShow = false
        if (err.response.status === 429) {
          this.$toast.fail('发送失败')
        }
        this.$toast.fail('发送失败')
      }
    }
  }
}
</script>

<style scoped lang="less">
.login-container {
.toutiao{
    font-size: 37px;
}
.send-sms-btn {
    background-color: #ededed;
    width: 152px;
    height: 46px;
    line-height: 46px;
    font-size: 22px;
    color: #666;
}

.login-btn-wrap {
  padding: 53px 33px;
  .login-btn {
    background-color: #6db4fb;
    border:none
  }
}
}
</style>
