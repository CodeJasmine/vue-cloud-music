<template>
  <v-app class="bg">
    <mu-container>
      <mu-form
        ref="form"
        :model="validateForm"
        class="login-form"
      >
        <mu-form-item
          label="用户名"
          prop="username"
          :rules="usernameRules"
        >
          <mu-text-field
            v-model="validateForm.username"
            prop="username"
          ></mu-text-field>
        </mu-form-item>
        <mu-form-item
          label="密码"
          prop="password"
          :rules="passwordRules"
        >
          <mu-text-field
            v-model="validateForm.password"
            prop="password"
          ></mu-text-field>
        </mu-form-item>
        <mu-auto-complete
          label="提示输入内容"
          v-model="verifyCode"
        ></mu-auto-complete>
        <mu-button
          color="info"
          @click="refresh"
          v-if="vailiable"
        >获取验证码</mu-button>
        <img
          ref="image"
          alt=""
          v-if="!vailiable"
          @click="refresh"
        />
        <mu-form-item
          prop="isAgree"
          :rules="argeeRules"
        >
          <mu-checkbox
            label="同意用户协议"
            v-model="validateForm.isAgree"
          ></mu-checkbox>
        </mu-form-item>
        <mu-form-item>
          <mu-button
            color="primary"
            @click="submit"
          >提交</mu-button>
          <mu-button @click="clear">重置</mu-button>
          <mu-button
            color="purple"
            @click="gitHub"
          >
            GitHub
          </mu-button>
        </mu-form-item>
      </mu-form>
      <mu-dialog
        title="Dialog"
        width="360"
        :open.sync="openSimple"
      >
        登录成功
      </mu-dialog>
    </mu-container>
    <!--遮罩-->
    <div
      class="mask"
      v-if="show"
    >
      <div class="dialog">
        <h3>请选择要进入系统的角色</h3>
        <div class="btn-wrapper">
          <mu-button
            v-for="(role, index) in roles"
            :key="index"
            color="primary"
            class="btn"
            @click="gotoIndex(role.roleId)"
          >
            {{ role.roleName }}
          </mu-button>
        </div>
      </div>
    </div>
  </v-app>
</template>

<script>
export default {
  name: 'Login',
  data() {
    return {
      usernameRules: [
        { validate: (val) => !!val, message: '必须填写用户名' },
        { validate: (val) => val.length >= 3, message: '用户名长度大于3' }
      ],
      passwordRules: [
        { validate: (val) => !!val, message: '必须填写密码' },
        { validate: (val) => val.length >= 3 && val.length <= 10, message: '密码长度大于3小于10' }
      ],
      argeeRules: [{ validate: (val) => !!val, message: '必须同意用户协议' }],
      validateForm: {
        username: '',
        password: '',
        isAgree: false
      },
      verifyCode: '',
      menuList: [],
      show: false,
      vailiable: true,
      roles: []
    }
  },
  created() {
    this.axios
      .get(this.GLOBAL.baseUrl + '/captcha?name=' + this.validateForm.username, { responseType: 'blob' })
      .then((res) => {
        let img = this.$refs.codeImg
        let url = window.URL.createObjectURL(res.data)
        img.src = url
      })
  },
  mounted() {},
  methods: {
    submit() {
      this.$refs.form.validate().then((result) => {
        console.log('form valid: ', result)
        this.axios({
          method: 'post',
          url: this.GLOBAL.baseUrl + '/sysAdmin/login',
          data: {
            name: this.validateForm.username,
            password: this.validateForm.password,
            verifyCode: this.verifyCode
          }
        }).then((res) => {
          if (res.data.code === 1) {
            //存token
            localStorage.setItem('token', res.data.data.token)
            this.$store.commit('setToken', res.data.data.token)
            //存admin信息
            localStorage.setItem('id', res.data.data.admin.id)
            localStorage.setItem('name', res.data.data.admin.name)
            localStorage.setItem('avatar', res.data.data.admin.avatar)
            this.$store.commit('setName', res.data.data.admin.name)
            this.$store.commit('setAvatar', res.data.data.admin.avatar)
            this.roles = res.data.data.admin.roles
            //角色数量超过1个
            if (this.roles.length > 1) {
              //弹出遮罩层选择
              alert('登录成功，你的角色不只一个，请选择')
              //显示遮罩层，遮罩层按钮具体点击事件 gotoDashboard(roleId)
              this.show = true
            } else {
              //只有一个角色
              const roleId = res.data.data.admin.roles[0].roleId
              //将roleId存入全局
              localStorage.setItem('roleId', roleId)
              this.$router.push('/')
            }
          } else {
            //登录失败
            alert(res.data.msg)
            //清空验证码文本框
            this.verifyCode = ''
          }
        })
      })
    },
    refresh() {
      this.vailiable = false
      this.axios({
        method: 'get',
        url: 'http://localhost:8023/captcha',
        // 2、将请求数据转换为form-data格式
        params: {
          name: this.validateForm.username
        },
        // 3、设置请求头Content-Type
        headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
        responseType: 'blob'
      }).then((res) => {
        let img = this.$refs.image
        let url = window.URL.createObjectURL(res.data)
        img.src = url
      })
    },
    clear() {
      this.$refs.form.clear()
      this.validateForm = {
        username: '',
        password: '',
        isAgree: false
      }
    },
    gotoIndex(roleId) {
      //将roleId存入本地存储
      localStorage.setItem('roleId', roleId)
      this.$router.push('/')
    },
    gitHub() {
      const authorize_uri = 'https://github.com/login/oauth/authorize'
      const client_id = 'd018e29a85862a55579f'
      const redirect_uri = 'http://localhost:8023/oauth2/code/github'
      window.location.href = `${authorize_uri}?client_id=${client_id}&redirect_uri=${redirect_uri}`
    }
  }
}
</script>

<style scoped lang="scss">
.bg {
  background-image: url('../../assets/images/24.jpg');
  background-size: 100% 100%;
  opacity: 0.8;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
}
.login-form {
  max-width: 500px;
  margin-left: 300px;
  background-color: #fff;
  border-radius: 10px;
  padding: 10px;
  margin-top: 150px;
}
.mask {
  z-index: 900;
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.9);
  display: flex;
  align-items: center;
  justify-content: center;
  .dialog {
    z-index: 1000;
    width: 400px;
    height: 300px;
    line-height: 100px;
    background-color: #fff;
    border-radius: 10px;
    text-align: center;
    padding-top: 50px;
    .btn-wrapper {
      margin-left: 60px;
      display: flex;
      .btn {
        margin: 20px;
      }
    }
  }
}
</style>
