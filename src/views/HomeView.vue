<script setup>
import axios from 'axios'
// import { jwtDecode } from 'jwt-decode'
import { onMounted, ref } from 'vue'

const ishasAccount = ref(true)

const apiURL = 'https://todolist-api.hexschool.io'
const content = ref({})
const validate = ref({})
const hasAccount = ref({})
// const todos = ref({})

//init
onMounted(() => {
  const checkCookie = cookie.get('token')
  if (checkCookie) {
    hasAccount.value.token = checkCookie
    // hasAccount.value.name = checkCookie
    axiosCheck(checkCookie)
  }
})

//登入 && 註冊
const axiosRep = async (link, value, txt) => {
  try {
    const rep = await axios.post(`${apiURL + link}`, value)
    if (rep.data.status) {
      changText(txt, 'success')
      content.value = {}
      ishasAccount.value = true
    }
    if (rep.data.token) {
      hasAccount.value.token = rep.data.token
      hasAccount.value.name = rep.data.nickname
      cookie.set('token', hasAccount.value.token, rep.data.exp / 60 / 60 / 1000)
    }
  } catch (error) {
    changText(error.response.data.message, 'error')
    content.value = {}
  }
}
const validateSignInData = () => {
  const valueTxt = content.value
  validatefield.email(valueTxt.email)
  validatefield.password(valueTxt.password)
  if (!validate.value.email && !validate.value.password) {
    axiosRep('/users/sign_in', valueTxt, '登入成功')
  }
}
const validateRegisterData = () => {
  const valueTxt = content.value
  validatefield.email(valueTxt.email)
  validatefield.nickname(valueTxt.nickname)
  validatefield.password(valueTxt.password)
  validatefield.password2(valueTxt.password2)
  if (
    !validate.value.email &&
    !validate.value.nickname &&
    !validate.value.password &&
    !validate.value.password2
  ) {
    axiosRep('/users/sign_up', valueTxt, '註冊成功')
  }
}
const validatefield = {
  email: (email) => {
    if (!email) {
      validate.value.email = '此欄位不可為空'
    } else {
      const re =
        /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/
      if (!re.test(String(email).toLowerCase())) {
        validate.value.email = 'E-mail格式錯誤'
      } else {
        validate.value.email = ''
      }
    }
  },
  nickname: (nickname) => {
    if (!nickname) {
      validate.value.nickname = '此欄位不可為空'
    } else {
      validate.value.nickname = ''
    }
  },
  password: (password) => {
    if (!password) {
      validate.value.password = '此欄位不可為空'
    } else {
      if (password.length < 6) {
        validate.value.password = '長度不足 6 個字'
      } else {
        validate.value.password = ''
      }
    }
  },
  password2: (password2) => {
    if (!password2) {
      validate.value.password2 = '此欄位不可為空'
    } else {
      if (password2 !== content.value.password) {
        validate.value.password2 = '密碼不一致'
      } else {
        validate.value.password2 = ''
      }
    }
  },
}

//驗證 & 登出
const axiosCheck = async (txt) => {
  if (!txt) return (validate.value.token = '請輸入Token值')
  try {
    const rep = await axios.get(`${apiURL}/users/checkout`, {
      headers: {
        Authorization: txt,
      },
    })
    if (rep.data.status) {
      changText('驗證成功', 'success')
      content.value = {}
      validate.value = {}
    }
  } catch (error) {
    changText(error.response.data.message, 'error')
    content.value = {}
    validate.value = {}
  }
}
const axiosOut2 = async () => {
  if (!content.value.token) return (validate.value.token = '請輸入Token值')
  try {
    const rep = await axios.post(`${apiURL}/users/sign_out`, content.value.token, {
      headers: {
        Authorization: content.value.token,
      },
    })
    if (rep.data.status) {
      changText('登出成功', 'success')
      cookie.remove('token')
      hasAccount.value.token = ''
      content.value = {}
      validate.value = {}
    }
  } catch (error) {
    changText(error.response.data.message, 'error')
    content.value = {}
    validate.value = {}
  }
}
const axiosOut = async () => {
  try {
    const rep = await axios.post(`${apiURL}/users/sign_out`, hasAccount.value.token, {
      headers: {
        Authorization: hasAccount.value.token,
      },
    })
    if (rep.data.status) {
      changText('登出成功', 'success')
      cookie.remove('token')
      hasAccount.value.token = ''
    }
  } catch (error) {
    changText(error.response.data.message, 'error')
  }
}

// 建立message
const messages = ref([])
const changText = (msg, color) => {
  const data = {
    id: new Date().getTime(),
    msgTxt: msg,
    colorTxt: color,
  }
  messages.value.push(data)
  removeTxt(data.id)
}
const removeTxt = (id) => {
  setTimeout(() => {
    const index = messages.value.findIndex((message) => message.id == id)
    messages.value.splice(index, 1)
  }, 3000)
}

//cookie
const cookie = {
  set: (cname, cvalue, exhours) => {
    var d = new Date()
    d.setTime(d.getTime() + exhours * 60 * 60 * 1000)
    var expires = 'expires=' + d.toGMTString()
    document.cookie = cname + '=' + cvalue + '; ' + expires
  },
  get: (cname) => {
    var name = cname + '='
    var ca = document.cookie.split(';')
    for (var i = 0; i < ca.length; i++) {
      var c = ca[i].trim()
      if (c.indexOf(name) == 0) return c.substring(name.length, c.length)
    }
    return ''
  },
  remove: (cname) => {
    document.cookie = cname + '=; expires=0; path=/2025-vue-hexschool;'
  },
}
</script>

<template>
  <div class="sign" id="sign" v-if="!cookie.get('token')">
    <div class="img pc">
      <img src="@/assets/images/left.png" alt="" />
    </div>
    <div class="img mo">
      <img src="@/assets/images/logo_lg.png" alt="" />
    </div>
    <div class="form">
      <div class="signIn" v-if="ishasAccount">
        <h2>最實用的線上代辦事項服務</h2>
        <form action="">
          <div class="form-list">
            <label for="">Email</label>
            <input type="email" placeholder="請輸入Email" v-model="content.email" />
            <small>{{ validate.email }}</small>
          </div>
          <div class="form-list">
            <label for="">密碼</label>
            <input type="password" placeholder="請輸入密碼" v-model="content.password" />
            <small>{{ validate.password }}</small>
          </div>
        </form>
        <div class="btn">
          <button class="black" type="button" @click="validateSignInData">登入</button><br />
          <button type="button" @click="((ishasAccount = false), (validate = {}), (content = {}))">
            註冊帳號
          </button>
        </div>
      </div>
      <div class="register" v-else-if="ishasAccount == false">
        <h2>註冊帳號</h2>
        <form action="">
          <div class="form-list">
            <label for="">Email</label>
            <input type="email" placeholder="請輸入Email" v-model="content.email" />
            <small>{{ validate.email }}</small>
          </div>
          <div class="form-list">
            <label for="">您的暱稱</label>
            <input type="text" placeholder="請輸入您的暱稱" v-model="content.nickname" />
            <small>{{ validate.nickname }}</small>
          </div>
          <div class="form-list">
            <label for="">密碼</label>
            <input type="password" placeholder="請輸入密碼" v-model="content.password" />
            <small>{{ validate.password }}</small>
          </div>
          <div class="form-list">
            <label for="">再次輸入密碼</label>
            <input type="password" placeholder="請再次輸入密碼" v-model="content.password2" />
            <small>{{ validate.password2 }}</small>
          </div>
        </form>
        <div class="btn">
          <button class="black" type="button" @click="validateRegisterData">註冊帳號</button><br />
          <button type="button" @click="((ishasAccount = true), (validate = {}), (content = {}))">
            登入
          </button>
        </div>
      </div>
      <!-- <div class="verify" v-else>
        <h2>驗證 & 登出</h2>
        <form action="">
          <div class="form-list">
            <label for="">Token</label>
            <input type="text" placeholder="請輸入Token" v-model="content.token" />
            <small>{{ validate.token }}</small>
          </div>
        </form>
        <div class="btn">
          <button class="black" type="button" @click="axiosCheck(content.toke)">驗證</button><br />
          <button type="button" @click="axiosOut2">登出</button><br />
          <button
            class="black"
            type="button"
            style="margin-top: 12px"
            @click="ishasAccount = false"
          >
            進入Todo
          </button>
        </div>
      </div> -->
    </div>
  </div>
  <div class="msg">
    <template v-for="item in messages" :key="item.id">
      <div class="message" :class="item.colorTxt">{{ item.msgTxt }}</div>
    </template>
  </div>
  <div class="todoBox" v-if="cookie.get('token')">
    <header>
      <div class="logo">
        <img alt="logo" src="@/assets/images/logo_lg.png" />
      </div>
      <nav>
        <div class="name">{{ hasAccount.name }}的代辦</div>
        <div>
          <button type="button" @click="axiosOut">登出</button>
        </div>
      </nav>
    </header>
    <main>Todo 畫面</main>
  </div>
</template>

<style lang="scss">
.sign {
  margin: 0 auto;
  @media (min-width: 768px) {
    max-width: 90%;
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  @media (max-width: 767px) {
    max-width: 83.2%;
    margin-top: 48px;
  }
  .img {
    margin-right: 106px;
    @media (max-width: 1200px) {
      margin-right: 8.8vw;
    }
    @media (max-width: 991px) {
      margin-right: 5vw;
    }
    img {
      display: block;
      max-width: 100%;
      margin: 0 auto;
    }
    &.pc {
      @media (max-width: 767px) {
        display: none;
      }
    }
    &.mo {
      display: none;
      @media (max-width: 767px) {
        display: block;
        margin: 0 auto 16px;
      }
    }
  }
  .form {
    max-width: 304px;
    width: 100%;
    @media (max-width: 767px) {
      max-width: 100%;
    }
  }
  .form-list {
    margin-bottom: 16px;
  }
  label {
    display: block;
    font-size: 14px;
    font-weight: 700;
    margin-bottom: 4px;
    line-height: 1.42;
  }
  input {
    width: 100%;
    height: 47px;
    border-radius: 10px;
    background-color: #fff;
    border: none;
    padding: 0 16px;
    font-size: 16px;
    font-weight: 500;
    font-family: 'Noto Sans TC', sans-serif;
    &:focus,
    &:active {
      border: none;
      background-color: #fff;
      outline: none;
    }
  }
  small {
    display: inline-block;
    margin-top: 4px;
    font-size: 14px;
    font-weight: 700;
    color: #d87355;
  }
  h2 {
    font-size: 24px;
    font-weight: 700;
    margin-bottom: 24px;
    @media (max-width: 767px) {
      text-align: center;
      margin-bottom: 26px;
    }
  }
  .btn {
    text-align: center;
    margin-top: 24px;
    button {
      display: inline-block;
      width: 160px;
      height: 47px;
      text-align: center;
      line-height: 47px;
      border: 0;
      border-radius: 10px;
      padding: 0;
      font-size: 16px;
      font-weight: 700;
      cursor: pointer;
      color: #333;
      &.black {
        color: #fff;
        background-color: #333;
        margin-bottom: 12px;
      }
    }
  }
}
.msg {
  position: absolute;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  .message {
    padding: 10px 20px 12px;
    margin-bottom: 10px;
    border-radius: 4px;
    color: #fff;
    &.success {
      background-color: rgba(#28a745, 0.8);
    }
    &.error {
      background-color: rgba(#dc3545, 0.8);
    }
  }
}
.todoBox {
  min-height: 100vh;
  background: url('../assets/images/bg.png') center bottom / 100% no-repeat;
  @media (max-width: 767px) {
    padding: 0 32px;
    background: none;
  }
}
header {
  width: 100%;
  height: 70px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 34px;
  @media (max-width: 767px) {
    padding: 16px 0;
  }
  .logo {
    max-width: 242px;
    img {
      display: block;
      max-width: 100%;
    }
  }
  nav {
    display: flex;
    align-items: center;
    justify-content: center;
    .name {
      font-size: 16px;
      font-weight: 700;
      margin-left: 24px;
      @media (max-width: 767px) {
        display: none;
      }
    }
    button {
      cursor: pointer;
      color: #333;
      font-size: 16px;
      border: 0;
      padding: 0;
      margin: 0 0 0 24px;
      background-color: transparent;
      @media (max-width: 767px) {
        margin: 0;
        font-size: 14px;
      }
    }
  }
}
</style>
