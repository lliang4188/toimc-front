<template>
  <div class="layui-container fly-marginTop">
    <div class="fly-panel fly-panel-user" pad20>
      <div class="layui-tab layui-tab-brief" lay-filter="user">
        <ul class="layui-tab-title">
          <li class="layui-this">登入</li>
          <li><router-link :to="{ name: 'reg' }">注册</router-link></li>
        </ul>
        <div class="layui-form layui-tab-content" style="padding: 20px 0">
          <validation-observer v-slot="{ validate }" ref="observer">
            <div class="layui-tab-item layui-show">
              <div class="layui-form layui-form-pane">
                <form method="post">
                  <div class="layui-form-item">
                    <label class="layui-form-label">用户名</label>
                    <validation-provider
                      name="email"
                      rules="required|email"
                      v-slot="{ errors }"
                    >
                      <div class="layui-input-inline">
                        <input
                          type="text"
                          name="username"
                          placeholder="请输入用户名"
                          autocomplete="off"
                          class="layui-input"
                          v-model.trim="username"
                        />
                      </div>
                      <div class="layui-form-mid">
                        <span style="color: #c00">{{ errors[0] }}</span>
                      </div>
                    </validation-provider>
                  </div>
                  <div class="layui-form-item">
                    <label for="L_pass" class="layui-form-label">密码</label>
                    <validation-provider
                      name="password"
                      rules="required|min:6"
                      v-slot="{ errors }"
                    >
                      <div class="layui-input-inline">
                        <input
                          type="password"
                          name="password"
                          placeholder="请输入密码"
                          autocomplete="off"
                          class="layui-input"
                          v-model.trim="password"
                        />
                      </div>
                      <div class="layui-form-mid">
                        <span style="color: #c00">{{ errors[0] }}</span>
                      </div>
                    </validation-provider>
                  </div>

                  <div class="layui-form-item">
                    <validation-provider
                      name="code"
                      ref="codefield"
                      rules="required|length:4"
                      v-slot="{ errors }"
                    >
                      <div class="layui-row layui-clear">
                        <label class="layui-form-label">验证码</label>
                        <div class="layui-input-inline">
                          <input
                            type="text"
                            name="code"
                            placeholder="请输入图片验证码"
                            autocomplete="off"
                            class="layui-input"
                            v-model.trim="code"
                          />
                        </div>
                        <div v-html="svg" class="svg" @click="_getCode()"></div>
                      </div>
                      <div class="layui-form-mid">
                        <span style="color: #c00">{{ errors[0] }}</span>
                      </div>
                    </validation-provider>
                  </div>
                  <div class="layui-form-item">
                    <button class="layui-btn" type="button" @click=" validate().then(submit)">
                      立即登录
                    </button>
                    <span style="padding-left: 20px">
                      <router-link :to="{ name: 'forget' }"
                        >忘记密码？</router-link
                      >
                    </span>
                  </div>
                  <div class="layui-form-item fly-form-app">
                    <span>或者使用社交账号登入</span>
                    <a
                      href=""
                      onclick="layer.msg('正在通过QQ登入', {icon:16, shade: 0.1, time:0})"
                      class="iconfont icon-qq"
                      title="QQ登入"
                    ></a>
                    <a
                      href=""
                      onclick="layer.msg('正在通过微博登入', {icon:16, shade: 0.1, time:0})"
                      class="iconfont icon-weibo"
                      title="微博登入"
                    ></a>
                  </div>
                </form>
              </div>
            </div>
          </validation-observer>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import CodeMixin from '@/mixin/code'
import { login } from '@/api/login'
export default {
  mixins: [CodeMixin],
  name: 'login',

  data () {
    return {
      username: '',
      password: ''
    }
  },
  mounted () {
  },
  methods: {
    async submit () {
      const isValid = await this.$refs.observer.validate()
      if (!isValid) {
        return
      }
      login({
        username: this.username,
        password: this.password,
        code: this.code,
        sid: this.$store.state.sid
      }).then((res) => {
        if (res.code === 200) {
          // 存储用户的登录名
          res.data.username = this.username
          this.$store.commit('setUserInfo', res.data)
          this.$store.commit('setIsLogin', true)
          this.$store.commit('setToken', res.token)
          this.username = ''
          this.password = ''
          this.code = ''
          requestAnimationFrame(() => {
            this.$refs.observer && this.$refs.observer.reset()
          })
          this.$router.push({ name: 'index' })
        } else if (res.code === 401) {
          this.$refs.codefield.setErrors([res.msg])
        }
      })
        .catch((err) => {
          const data = err.response.data
          if (data.code === 500) {
            this.$alert('用户名密码校验失败，请检查！')
          } else {
            this.$alert('服务器错误')
          }
        })
    }
  }

}
</script>

<style lang="scss" scoped>

</style>
