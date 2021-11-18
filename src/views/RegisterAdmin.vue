<template>
  <div class="container">
    <div class="row register-page">
      <form class="col s12" id="reg-form">
        <div class="error-message">
          {{ errorMessage }}
          {{ uniqueErrorMessage }}
        </div>
        <div class="row">
          <div class="input-field col s6">
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
  private errorMessage = '';
  private uniqueErrorMessage = '';

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
      this.errorMessage = '姓の欄が未入力です。全ての入力欄を記入してください';
      return;
    }
    if (this.firstName == '') {
      this.errorMessage = '名が未入力です。全ての入力欄を記入してください';
      return;
    }
    if (this.mailAddress == '') {
      this.errorMessage =
        'メールアドレスが未入力です。全ての入力欄を記入してください';
      return;
    }
    if (this.password == '') {
      this.errorMessage =
        'パスワードが未入力です。全ての入力欄を記入してください';
      return;
    }
    if (this.password != this.comfirmPassword) {
      this.errorMessage = 'パスワードと確認パスワードが一致しません';
      this.password = '';
      this.comfirmPassword = '';
      return;
    }

    const response = await axios.post(`${config.EMP_WEBAPI_URL}/insert`, {
      name: this.lastName + ' ' + this.firstName,
      mailAddress: this.mailAddress,
      password: this.password,
    });
    console.dir('response:' + JSON.stringify(response));

    //演習1-3 メールの重複があったら登録できないエラーメッセージを出す。
    if (response.data.status === 'success') {
      //演習1-1 管理者登録成功後の遷移先をログイン画面に変更
      this.$router.push('/loginAdmin');
    } else if (response.data.status === 'error') {
      this.uniqueErrorMessage = '登録できませんでした';
    }
  }
}
</script>

<style scoped>
.register-page {
  width: 600px;
}
.error-message {
  color: red;
}
</style>
