<template>
  <div class="hello">
    <span class="uk-position-center" v-if="generated === 0" uk-spinner="ratio: 5.5"></span>
    <div v-else-if="generated === 1">
      <form v-on:submit.prevent="onSubmit" class="uk-position-center">
        <div class="uk-margin">
            <div class="uk-inline">
                <span class="uk-form-icon" uk-icon="icon: user"></span>
                <input v-model="username" class="uk-input" required type="text">
            </div>
        </div>
        <div class="uk-margin">
            <div class="uk-inline">
                <span class="uk-form-icon uk-form-icon-flip" uk-icon="icon: lock"></span>
                <input v-model="password" class="uk-input" required type="password">
            </div>
        </div>
        <div class="uk-margin">
          <button @click="login" class="uk-button uk-button-default">Submit</button>
        </div>
      </form>
    </div>
    <div v-else="generated === 2">
      <div class="uk-child-width-1-1@s uk-grid-match" uk-grid>
        <div class="uk-height-small" v-for="item in messages" :key="item.message">
          <div class="uk-card-body">
            {{ckeys.decrypt(item.user)}}<br>
            <p>{{ckeys.decrypt(item.content)}}</p>
          </div>
        </div>
      </div>
      <div class="uk-position-bottom uk-width-1-1">
          <div class="uk-inline uk-width-1-1">
              <a @click="send" 
              class="uk-form-icon uk-form-icon-flip" href="#" uk-icon="icon: chevron-double-right"></a>
              <input v-model="message" class="uk-input uk-width-1-1" type="text" maxlength="45">
          </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
const JSEncrypt = require('node-jsencrypt')
var keypair = require('keypair');


export default {
  name: 'RSA',
  data: function(){
    return {
      // инициализируем ключи клиента
      ckeys: new JSEncrypt(),
      username: "",
      password: "",
      serPubEnc: null,
      sesPubEnc: null,
      generated: false,
      tempLog: "",
      username: "",
      message: "",
      messages: null
    }
  },
  mounted: function() {
    var _this = this;
    // запрос на обмен ключами для старта сессии
    axios
    .post('http://localhost:5000/api/generate_keys', {
      public: _this.ckeys.getPublicKey()
    })
    .then(response => {
      // сохраняем публичный ключ сервера
      _this.serPubEnc = new JSEncrypt();
      _this.serPubEnc.setPublicKey(
        response.data.public_server
      );

      // сохраняем публичный ключ сессии
      _this.sesPubEnc = new JSEncrypt();
      _this.sesPubEnc.setPublicKey(
        response.data.public_session
      );

      // сохраняем временный идентификатор сессии
      _this.tempLog = response.data.temp_login;
      _this.generated = 1;
    })
  },
  updated: function() {
    var _this = this;
    if (_this.generated === 2) {
      // запрашиваем сообщения после логина
      setInterval(() => {
        axios
        .post('http://localhost:5000/api/messages', {
          username: _this.serPubEnc.encrypt(
            _this.ckeys.decrypt(
              _this.username
            )
          )
        })
        .then(response => {
          _this.messages = response.data.messages;
        })
      }, 45000);
    }
  },
  methods: {
    login: function (e) {
      var _this = this;
      // шифруем логин, пароль (публичным ключем сессии) 
      // и временный идентификатор (публичным ключем сервера)
      axios
      .post('http://localhost:5000/api/login', {
        username: _this.sesPubEnc.encrypt(
          _this.username
        ),
        passw: _this.sesPubEnc.encrypt(
          _this.password
        ),
        temp_login: _this.serPubEnc.encrypt(
          _this.ckeys.decrypt(
            _this.tempLog
          )
        )
      })
      .then(response => {
        _this.username = response.data.username;
        _this.generated = 2;
      })
    },
    send: function(e) {
      var _this = this;
      console.log(_this.message);
      // шифруем сообщение и отправляем на сервер
      axios
      .post('http://localhost:5000/api/message', {
        username: _this.serPubEnc.encrypt(
          _this.ckeys.decrypt(_this.username)
        ),
        message: _this.sesPubEnc.encrypt(
          _this.message
        )
      })
      .then(response => {
        _this.messages = response.data.messages;
        _this.message = "";
      })
    }
  }
}
</script>

<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
