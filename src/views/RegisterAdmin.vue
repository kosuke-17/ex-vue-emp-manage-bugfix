<template>
  <div class="container">
    <div class="row register-page">
      <form class="col s12" id="reg-form">
        <div class="error-message">
          {{ uniqueErrorMessage }}
        </div>
        <div class="row">
          <div class="input-field col s6">
            <div class="error-message">
              {{ lastNameErrorMessage }}
            </div>
            <input
              id="last_name"
              type="text"
              class="validate"
              v-model="lastName"
              required
            />
            <label for="last_name">姓</label>
          </div>
          <div class="input-field col s6">
            <div class="error-message">
              {{ firstNameErrorMessage }}
            </div>
            <input
              id="first_name"
              type="text"
              class="validate"
              v-model="firstName"
              required
            />
            <label for="first_name">名</label>
          </div>
        </div>
        <div class="row">
          <div class="input-field col s12">
            <div class="error-message">
              {{ mailAddressErrorMessage }}
            </div>
            <input
              id="email"
              type="email"
              class="validate"
              v-model="mailAddress"
              required
            />
            <label for="email">メールアドレス</label>
          </div>
        </div>
        <div class="row">
          <div class="input-field col s12">
            <div class="error-message">
              {{ passwordErrorMessage }}
            </div>
            <input
              id="password"
              type="password"
              class="validate"
              minlength="8"
              v-model="password"
              required
            />
            <label for="password">パスワード</label>
          </div>
        </div>
        <div class="row">
          <div class="input-field col s12">
            <div class="error-message">
              {{ comfirmPasswordErrorMessage }}
            </div>
            <input
              id="comfirmPassword"
              type="password"
              class="validate"
              minlength="8"
              v-model="comfirmPassword"
              required
            />
            <label for="comfirmPassword">確認パスワード</label>
          </div>
          <div>
            <form class="search">
              <div class="error-message">
                {{ zipcodeErrorMessage }}
              </div>
              <br />
              郵便番号<input
                class="validate"
                type="text"
                v-model="inputZipcode"
              />
              <br />
            </form>
            <button type="button" @click="searchAddress(inputZipcode)">
              住所検索
            </button>
            <br />
            住所(ボタンまだない)<input
              id="inputAddress"
              type="text"
              v-model="inputAddress"
            />
            <label for="inputAddress">住所</label>
          </div>
        </div>
        <div class="row">
          <div class="input-field col s6">
            <button
              class="btn btn-large btn-register waves-effect waves-light"
              type="button"
              v-on:click="registerAdmin"
            >
              登録
              <i class="material-icons right">done</i>
            </button>
          </div>
        </div>
      </form>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import config from '@/const/const';
import axios from 'axios';

/**
 * 管理者登録をする画面.
 */
@Component
export default class RegisterAdmin extends Vue {
  // 姓
  private lastName = '';
  // 名
  private firstName = '';
  // メールアドレス
  private mailAddress = '';
  // パスワード
  private password = '';
  // パスワード
  private comfirmPassword = '';

  // エラーメッセージ
  private lastNameErrorMessage = '';
  private firstNameErrorMessage = '';
  private mailAddressErrorMessage = '';
  private passwordErrorMessage = '';
  private comfirmPasswordErrorMessage = '';
  private uniqueErrorMessage = '';
  private errorFlag = false;

  private inputZipcode = '';
  private inputAddress = '';
  private zipcodeErrorMessage = '';

  /**
   * 管理者情報を登録する.
   *
   * @remarks
   * 本メソッドは非同期でWebAPIを呼び出し管理者登録をするためasync/await axiosを利用しています。これらを利用する場合は明示的に戻り値にPromiseオブジェクト型を指定する必要があります。
   * @returns Promiseオブジェクト
   */
  async registerAdmin(): Promise<void> {
    // 管理者登録処理

    // 演習1-2 未入力の場合、処理終了
    if (this.lastName == '') {
      this.lastNameErrorMessage = '姓が未入力です。';
      this.errorFlag = true;
    }
    if (this.firstName == '') {
      this.firstNameErrorMessage = '名が未入力です。';
      this.errorFlag = true;
    }
    if (this.mailAddress == '') {
      this.mailAddressErrorMessage = 'メールアドレスが未入力です。';
      this.errorFlag = true;
    }
    if (this.password == '') {
      this.passwordErrorMessage = 'パスワードが未入力です。';
      this.errorFlag = true;
    }
    if (this.comfirmPassword == '') {
      this.comfirmPasswordErrorMessage = '確認パスワードが未入力です。';
      this.errorFlag = true;
    }
    if (this.password != this.comfirmPassword) {
      this.comfirmPasswordErrorMessage =
        'パスワードと確認パスワードが一致しません';
      this.password = '';
      this.comfirmPassword = '';
      this.errorFlag = true;
    }

    // エラーがあったら処理終了
    if (this.errorFlag) {
      this.errorFlag = false;
      return;
    }

    const response = await axios.post(`${config.EMP_WEBAPI_URL}/insert`, {
      name: this.lastName + ' ' + this.firstName,
      mailAddress: this.mailAddress,
      password: this.password,
      address: this.inputAddress,
    });
    console.dir('response:' + JSON.stringify(response));

    //演習1-3 メールの重複があったら登録できないエラーメッセージを出す。
    if (response.data.status === 'success') {
      //演習1-1 管理者登録成功後の遷移先をログイン画面に変更
      this.$router.push('/loginAdmin');
    } else if (response.data.status === 'error') {
      this.uniqueErrorMessage = '登録できませんでした';
      return;
    }
  }

  /**
   * 郵便番号から住所情報を取得する
   */
  async searchAddress(): Promise<void> {
    try {
      // eslint-disable-next-line @typescript-eslint/no-var-requires
      const axiosJsonpAdapter = require('axios-jsonp');
      const response = await axios.get('https://zipcoda.net/api', {
        adapter: axiosJsonpAdapter,
        params: {
          zipcode: this.inputZipcode,
        },
      });
      this.inputAddress = response.data.items[0].components.join('');
      console.log('成功');
    } catch (e) {
      this.zipcodeErrorMessage = '正しい値(7桁)を入力してください';
    }
  }
}
</script>

<style scoped>
.register-page {
  width: 600px;
}
.error-message {
  font-size: 5px;
  color: red;
}
</style>
