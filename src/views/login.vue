<template>
  <div class="login">
    <div class="logo-wrap">
      <img src="@/assets/images/loginLogo.png" />
    </div>
    <div class="moto-wrap">选择达昌 共创数智未来</div>
    <div class="login-form-wrap">
      <el-form ref="loginRef" :model="loginForm" :rules="loginRules" class="login-form">
        <h3 class="title">设备运维管理平台</h3>
        <el-form-item prop="username">
          <el-input v-model="loginForm.username" type="text" size="large" auto-complete="off" placeholder="账号">
            <template #prefix><svg-icon icon-class="user" class="el-input__icon input-icon" /></template>
          </el-input>
        </el-form-item>
        <el-form-item prop="password">
          <el-input v-model="loginForm.password" type="password" size="large" auto-complete="off" placeholder="密码"
            @keyup.enter="handleLogin">
            <template #prefix><svg-icon icon-class="password" class="el-input__icon input-icon" /></template>
          </el-input>
        </el-form-item>
        <el-form-item prop="code" v-if="captchaEnabled">
          <el-input v-model="loginForm.code" size="large" auto-complete="off" placeholder="验证码" style="width: 63%"
            @keyup.enter="handleLogin">
            <template #prefix><svg-icon icon-class="validCode" class="el-input__icon input-icon" /></template>
          </el-input>
          <div class="login-code">
            <img :src="codeUrl" @click="getCode" class="login-code-img" />
          </div>
        </el-form-item>
        <el-checkbox v-model="loginForm.rememberMe" style="margin:0px 0px 25px 0px;">记住密码</el-checkbox>
        <el-form-item style="width:100%;">
          <el-button class="login-btn" :loading="loading" size="large" type="primary" style="width:100%;"
            @click.prevent="handleLogin">
            <span v-if="!loading"></span>
            <span v-else></span>
          </el-button>
          <div style="float: right;" v-if="register">
            <router-link class="link-type" :to="'/register'">立即注册</router-link>
          </div>
        </el-form-item>
      </el-form>
    </div>

    <!--背景动画 start-->
    <video id="bg-video" autoplay loop muted width="100%" height="100%"
      style="object-fit: fill; min-height: 800px; min-width: 1400px">
      <source src="@/assets/video/login.mp4" type="video/mp4" />
    </video>
    <!--背景动画 end-->
    <!--  底部  -->
    <div class="el-login-footer">
      <span>Copyright © 2018-2024 cndachang.top All Rights Reserved.</span>
    </div>
  </div>
</template>

<script setup>
import { getCodeImg } from "@/api/login";
import Cookies from "js-cookie";
import { encrypt, decrypt } from "@/utils/jsencrypt";
import useUserStore from '@/store/modules/user'

const userStore = useUserStore()
const route = useRoute();
const router = useRouter();
const { proxy } = getCurrentInstance();

const loginForm = ref({
  username: "admin",
  password: "admin123",
  rememberMe: false,
  code: "",
  uuid: ""
});

const loginRules = {
  username: [{ required: true, trigger: "blur", message: "请输入您的账号" }],
  password: [{ required: true, trigger: "blur", message: "请输入您的密码" }],
  code: [{ required: true, trigger: "change", message: "请输入验证码" }]
};

const codeUrl = ref("");
const loading = ref(false);
// 验证码开关
const captchaEnabled = ref(true);
// 注册开关
const register = ref(false);
const redirect = ref(undefined);

watch(route, (newRoute) => {
  redirect.value = newRoute.query && newRoute.query.redirect;
}, { immediate: true });

function handleLogin() {
  proxy.$refs.loginRef.validate(valid => {
    if (valid) {
      loading.value = true;
      // 勾选了需要记住密码设置在 cookie 中设置记住用户名和密码
      if (loginForm.value.rememberMe) {
        Cookies.set("username", loginForm.value.username, { expires: 30 });
        Cookies.set("password", encrypt(loginForm.value.password), { expires: 30 });
        Cookies.set("rememberMe", loginForm.value.rememberMe, { expires: 30 });
      } else {
        // 否则移除
        Cookies.remove("username");
        Cookies.remove("password");
        Cookies.remove("rememberMe");
      }
      // 调用action的登录方法
      userStore.login(loginForm.value).then(() => {
        const query = route.query;
        const otherQueryParams = Object.keys(query).reduce((acc, cur) => {
          if (cur !== "redirect") {
            acc[cur] = query[cur];
          }
          return acc;
        }, {});
        router.push({ path: redirect.value || "/", query: otherQueryParams });
      }).catch(() => {
        loading.value = false;
        // 重新获取验证码
        if (captchaEnabled.value) {
          getCode();
        }
      });
    }
  });
}

function getCode() {
  getCodeImg().then(res => {

    // captchaEnabled.value = res.captchaOnOff === undefined ? true : res.captchaOnOff; // 根据后台设定来
    captchaEnabled.value = false; // 暂时指定，后面根据服务器返回数据设置
    if (captchaEnabled.value) {
      codeUrl.value = "data:image/gif;base64," + res.img;
      loginForm.value.uuid = res.uuid;
    }
  });
}

function getCookie() {
  const username = Cookies.get("username");
  const password = Cookies.get("password");
  const rememberMe = Cookies.get("rememberMe");
  loginForm.value = {
    username: username === undefined ? loginForm.value.username : username,
    password: password === undefined ? loginForm.value.password : decrypt(password),
    rememberMe: rememberMe === undefined ? false : Boolean(rememberMe)
  };
}

getCode();
getCookie();
</script>

<style lang='scss' scoped>
.login {
  display: flex;
  justify-content: flex-end;
  align-items: center;
  width: 100%;
  height: 100%;
  // background-image: url("../assets/images/login-background.jpg");
  min-width: 1600px;
  background-size: cover;
  position: relative;
}

#bg-video {
  width: 100%;
  height: 100%;
  z-index: 0;
}

.logo-wrap {
  position: fixed;
  top: 40px;
  left: 40px;
  width: 180px;
  height: 44px;

  img {
    display: block;
    width: 100%;
    height: 100%;
  }
}

.moto-wrap {
  font-size: 80px;
  color: #d3e9ff;
  left: 13%;
  position: absolute;
  top: 50%;
  margin-top: -60px;
  display: inline-block;
  font-family: "moyeti";
  text-shadow: 2px 2px 8px rgba(0, 0, 0, 0.2);
  z-index: 1;
  // animation: gradientAnimation 1s linear;
  // animation-delay: 12s;
  // opacity: 0;
  // animation-fill-mode: forwards;
}

// 登录表单容器
.login-form-wrap {
  position: absolute;
  margin-right: 170px;
  z-index: 1;
  // z-index: 10;
  // padding: 5px;
}

.title {
  font-size: 30px;
  font-family: "moyeti";
  font-weight: normal;
  color: #ffffff;
  line-height: 65px;
  text-shadow: 0px 2px 4px rgba(0, 0, 0, 0.5);
  margin: 0;
  margin-bottom: 20px;
}

.login-form {
  // position: absolute;
  border-radius: 6px;
  // width: 400px;
  // padding: 25px 25px 5px 25px;
  border-radius: 6px;
  padding: 47px 38px 56px 38px;
  width: 335px;
  height: 400px;

  background-image: url("../assets/images/loginBg.png");
  background-size: cover;
  background-repeat: no-repeat;

  .el-input {
    height: 40px;

    input {
      height: 40px;
    }
  }

  .input-icon {
    height: 39px;
    width: 14px;
    margin-left: 0px;
  }
}

.login-btn {
  background-image: url('@/assets/images/loginbtn.png');
  background-repeat: no-repeat;
  background-position: center center;
}

.login-tip {
  font-size: 13px;
  text-align: center;
  color: #bfbfbf;
}

.login-code {
  width: 33%;
  height: 40px;
  float: right;

  img {
    cursor: pointer;
    vertical-align: middle;
  }
}

.el-login-footer {
  height: 40px;
  line-height: 40px;
  position: fixed;
  bottom: 0;
  width: 100%;
  text-align: center;
  color: #fff;
  font-family: Arial;
  font-size: 12px;
  letter-spacing: 1px;
}

.login-code-img {
  height: 40px;
  padding-left: 12px;
}
</style>
