<template>
  <div class="main">
    <a-tooltip placement="bottom">
      <template slot="title">
        <span>点击登录</span>
      </template>
      <div class="title-container" style="text-align: center;cursor: pointer" @click="handleLogin">
        <a-icon type="github" theme="filled" :style="{ fontSize: '108px', color: '#08c' }" />
      </div>
    </a-tooltip>

  </div>
</template>

<script>
import { timeFix } from '@/utils/util'
import { getOauthLoginByGithub } from '@/api/user'
import openWindow from '@/utils/open-window'

export default {
  components: {
  },
  data () {
    return {
      isLoginError: false,
      state: {
        time: 60,
        loginBtn: false,
        // login type: 0 email, 1 username, 2 telephone
        loginType: 0,
        smsSendBtn: false
      }
    }
  },
  created () {
  },
  methods: {
    handleLogin () {
      getOauthLoginByGithub().then(res => {
        openWindow(res.model.authorizeUrl, '绑定GitHub', 540, 540)
        window.addEventListener('message', this.loginGithubHandel, false)
      })
    },
    loginGithubHandel (e) {
      const { socialId } = e.data
      if (socialId) {
        this.$store.dispatch('socialLogin', e.data).then(res => {
          this.loginSuccess(res)
        })
        window.removeEventListener('message', this.loginGithubHandel, false)
      }
    },
    loginSuccess (res) {
      // check res.homePage define, set $router.push name res.homePage
      // Why not enter onComplete
      /*
      this.$router.push({ name: 'analysis' }, () => {
        console.log('onComplete')
        this.$notification.success({
          message: '欢迎',
          description: `${timeFix()}，欢迎回来`
        })
      })
      */
      this.$router.push({ path: '/' })
      // 延迟 1 秒显示欢迎信息
      setTimeout(() => {
        this.$notification.success({
          message: '欢迎',
          description: `${timeFix()}，欢迎回来`
        })
      }, 1000)
      this.isLoginError = false
    },
    requestFailed (err) {
      this.isLoginError = true
      this.$notification['error']({
        message: '错误',
        description: ((err.response || {}).data || {}).message || '请求出现错误，请稍后再试',
        duration: 4
      })
    }
  }
}
</script>

<style lang="less" scoped>
.user-layout-login {
  label {
    font-size: 14px;
  }

  .getCaptcha {
    display: block;
    width: 100%;
    height: 40px;
  }

  .forge-password {
    font-size: 14px;
  }

  button.login-button {
    padding: 0 15px;
    font-size: 16px;
    height: 40px;
    width: 100%;
  }

  .user-login-other {
    text-align: left;
    margin-top: 24px;
    line-height: 22px;

    .item-icon {
      font-size: 24px;
      color: rgba(0, 0, 0, 0.2);
      margin-left: 16px;
      vertical-align: middle;
      cursor: pointer;
      transition: color 0.3s;

      &:hover {
        color: #1890ff;
      }
    }
    .title-container {
      position: relative;
    }
    .register {
      float: right;
    }
  }
}
</style>
