<template>
  <div>ログアウト中・・・</div>
</template>
<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import config from '@/const/const';
import axios from 'axios';

/**
 * ログアウトをする画面.
 *
 */
@Component
export default class LogoutAdmin extends Vue {
  /**
   * ログアウトする.
   *
   * @remarks
   * 本メソッドは非同期でWebAPIを呼び出しログアウトをするためasync/await axiosを利用しています。これらを利用する場合は明示的に戻り値にPromiseオブジェクト型を指定する必要があります。
   * @returns Promiseオブジェクト
   */
  async created(): Promise<void> {
    await axios.get(`${config.EMP_WEBAPI_URL}/logout`);
    // console.dir('response:' + JSON.stringify(response));
    // ログイン画面に遷移する
    this.$router.push('/loginAdmin');
    // 'false'だと文字列が含まれてtrueの状態のため、空文字にしている
    sessionStorage.setItem('userStatus', '');
    this['$store'].commit('changeUserStatus', {
      user: sessionStorage.getItem('userStatus'),
    });
  }
}
</script>
