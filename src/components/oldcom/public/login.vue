<template>
<div class='login'>
  <div class="login_cot">
    <div class="login_border" :class="{'login_trans':toLogin}">
      <div class="login_b_l login_b_p">
        <span class='l_span'>必赢出票平台</span>
        <el-input v-model="username" placeholder="请输入手机号" class='l_input' ref='lname' @keyup.native.enter=" handleInputPwd"></el-input>
        <el-input v-model="userpwd" placeholder="请输入密码" type='password' ref='lpwd' class='l_input' @keyup.native.enter=" _handleLogin"></el-input>
        <el-button type="text" class='l_elb l_ebtn_r' @click='toTran'>忘记密码</el-button>
        <el-button type="primary" class='l_elb l_ebtn' @click="_handleLogin">登 录</el-button>
      </div>
      <div class="login_b_r login_b_p">
        <span class='l_span'>忘记密码</span>
        <el-input v-model="f_name" placeholder="请输入手机号" class='l_input'>
          <label slot="prepend">手机号　</label>
        </el-input>
        <el-input v-model="f_yzm" placeholder="请输入验证码" class='l_input' style="width:60%;float:left">
          <label slot="prepend">验证码　</label>
        </el-input>
        <el-button type="primary" style="width:35%;float:right;padding:12px 10px;" class='l_input' @click="_handleGerCode">{{codeText}}</el-button>
        <el-input v-model="f_pwd" placeholder="请输入密码" type='password' class='l_input'>
          <label slot="prepend">设置密码</label>
        </el-input>
        <el-input v-model="f_pwd2" placeholder="请再次输入密码" type='password' class='l_input'>
          <label slot="prepend">确认密码</label>
        </el-input>
        <el-button class='l_elb l_ebtn l_input' type="primary" @click="_handleReset">确 定</el-button>
        <el-button class='l_elb l_ebtn l_input' @click='toBack'>返 回</el-button>
        <div class="l_backi" :class="{'l_backshow':toLogin}" @click='toBack'>
          <i class="icon iconfont icon-back"></i>
        </div>
      </div>
    </div>

  </div>
</div>
</template>

<script>
import {
  asyncRouter
} from '../../router'
import CryptoJS from 'crypto-js';
export default {
  data() {
    return {
      username: "",
      userpwd: "",
      f_name: '',
      f_yzm: '',
      f_pwd: '',
      f_pwd2: '',
      toLogin: false,
      rember: true,
      codeFalg: false,
      codeText: '获取验证码'
    };
  },
  methods: {
    toGo() {
      this.$router.push({
        path: "/index"
      });
    },
    toTran() {
      this.toLogin = true;
    },
    handleInputPwd() {
      this.$refs.lpwd.focus()
    },
    cRouter(newRouter) {
      this.$store.state.routecotant = newRouter
      this.$router.addRoutes(newRouter)
      setTimeout(() => {
        this.$router.push({
          path: newRouter[0].children[0].path
        })
      })

    },
    _handleLogin() {
      if (this.username && this.userpwd) {
        let data = {
          username: this.username,
          password: this.userpwd
        }
        this.http.post('/user/login', data).then(res => {
          if (res) {
            if (res.status == 200) {
              this.$store.state.islogin = true;
              this.$store.state.userInfo.userName = this.username;
              this.$store.state.userInfo.token = res.data.token;
              this.$store.state.userInfo.types = res.data.type;
              this.$store.state.rule = res.data.roleId;
              if (this.username != localStorage.cpt_username) {
                this.$store.state.worklist = [];
                localStorage.work_list = '';
              }
              localStorage.setItem("work_role", res.data.roleId);
              localStorage.setItem("cpt_username", data.username);
              localStorage.setItem("cpt_token", res.data.token);
              localStorage.setItem("cpt_type", res.data.type);
              sessionStorage.setItem("cpt_token", res.data.token);
              if (!this.rember) {
                localStorage.setItem("cpt_token", '');
              }
              const rule = this.aesEN(res.data.resources)
              localStorage.setItem("cpt_rule", rule);
              const route = this.rule(asyncRouter, res.data.resources)
              // console.log(route,rule)
              this.$store.state.routecotant = route
              this.cRouter(route);
            } else {
              this._message(res.msg, "error");
              this.userpwd = '';
            }
          }
        }).catch(error => {

        })

      } else {
        this._message('用户名或密码不能为空！', "error");
      }
    },
    toBack() {
      this.f_name = '';
      this.f_yzm = '';
      this.f_pwd = '';
      this.f_pwd2 = '';
      this.toLogin = false;
    },
    _handleGerCode: async function() {
      if (!this.f_name) {
        this._message('手机号不能为空！', "error");
        return
      }
      if (this.codeFalg) return
      let _self = this;
      var a = 0;
      _self.codeFalg = true;
      _self.codeText = "重新发送(" + (60 - a) + "s)";
      const res = await this.http.post('/user/code/get', {
        type: 1,
        phone: this.f_name.trim()
      })
      if (res) {
        if (res.status == 200) {
          this._message('验证码已发送到你的手机,请查收！', "success");
          setInterval(function() {
            if (a != 60) {
              _self.codeFalg = true;
              _self.codeText = "重新发送(" + (60 - a) + "s)";
            } else {
              _self.codeFalg = false;
              _self.codeText = "获取验证码";
              return false;
            }
            a++;
          }, 1000)
        } else {
          this._message(res.msg, "error");
          _self.codeFalg = false;
          _self.codeText = "获取验证码";
        }
      } else {
        _self.codeFalg = false;
        _self.codeText = "获取验证码";
      }
    },
    _handleReset() {
      if (!this.f_yzm || !this.f_name) {
        this._message('手机号或验证码不能为空！', "error");
        return
      }
      if (!this.f_pwd || !this.f_pwd2) {
        this._message('密码不能为空！', "error");
        return
      }
      if (this.f_pwd != this.f_pwd2) {
        this._message('输入密码不一致，请重新输入！', "error");
        return
      }
      let data = {
        code: this.f_yzm,
        password: this.f_pwd,
        phone: this.f_name,
      }
      this.http.post('/user/forget', data).then(res => {
        if (res) {
          if (res.status == 200) {
            this._message('密码重置成功，请重新登入！', "success");
            this.toBack()
          } else {
            this._message(res.msg, "error");
            this.f_pwd = this.f_pwd2 = '';
          }
        }
      }).catch(error => {})
    }
  },
  components: {},
  watch: {},
  destroyed() {

  },
  created() {
    //  this.http.INIT(this)

    this.username = localStorage.cpt_username || '';
    if (AR_ADDR_QQDZ.indexOf('192.168.1') > -1) {
      this.userpwd = '123456'
    }
  },
  mounted() {

    if (this.username) {
      this.$refs.lpwd.focus()
    } else this.$refs.lname.focus()

    const name = localStorage.cpt_username || '';
    const list = localStorage.work_list || '';
    localStorage.clear();
    localStorage.cpt_username = name || '';
    localStorage.work_list = list;

  }
};
</script>

<style lang='less' scoped>
.login {
  .login_cot {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 15%;
    margin: auto;
    width: 400px;
    height: 550px;
    // border: 1px solid #ddd;
    padding: 10px 40px;
    // background-color: #fff;
    overflow: hidden;
    .login_border {
      position: relative;
      width: 100%;
      height: 100%;
      transform: translate3d(0, 0, 0);
      transition: all 0.4s linear;
    }
    .login_trans {
      transform: translate3d(-400px, 0, 0);
    }
  }
  .login_b_r {
    position: absolute;
    right: -400px;
    top: 0;
    width: 100%;
    height: 100%;
  }
  .login_b_l {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
  }
  .login_b_p {
    .l_span {
      display: block;
      margin: 20px 0;
      font-size: 24px;
    }
    .l_input {
      margin: 10px 0;
    }
    .l_elb {
      margin: 5px 0;
    }
    .l_ebtn_r {
      float: right;
    }
    .l_ebtn {
      // margin-bottm:20px;
      width: 100%;
    }
  }
  .l_backi {
    opacity: 0;
    position: absolute;
    top: 10px;
    left: -20px;
    width: 30px;
    height: 30px;
    z-index: 555;
    line-height: 30px;
    transition: all .8s linear;
    i {
      font-size: 28px;
      color: #737373;
    }
    i:hover {
      font-size: 34px;
    }
  }
  .l_backshow {
    opacity: 1;
  }
}
</style>

