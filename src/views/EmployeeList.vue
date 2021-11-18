<template>
  <div class="container">
    <!-- パンくずリスト -->
    <nav>
      <div class="nav-wrapper">
        <div class="col s12 teal">
          <a class="breadcrumb">従業員リスト</a>
        </div>
      </div>
    </nav>
    <div v-for="(num, i) of getPageNum" :key="i">
      <button type="button" @click="onchangepage(num)">{{ num }}</button>
    </div>
    <div>
      <form class="search">
        <input type="text" v-model="inputName" />
        <br />
      </form>
      <button @click="searchEmployee(inputName)">従業員検索</button>
    </div>
    {{ searchedMessage }}

    <div>従業員数:{{ getEmployeeCount }}人</div>
    <div class="row">
      <table class="striped">
        <thead>
          <tr>
            <th>名前</th>
            <th>入社日</th>
            <th>扶養人数</th>
          </tr>
        </thead>

        <tbody>
          <tr v-for="employee of currentEmployeeList" v-bind:key="employee.id">
            <td>
              <router-link :to="'/employeeDetail/' + employee.id">{{
                employee.name
              }}</router-link>
            </td>
            <td>{{ employee.formatHireDate }}</td>
            <td>{{ employee.dependentsCount }}人</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import { Employee } from '@/types/employee';
/**
 * 従業員一覧を表示する画面.
 */
@Component
export default class EmployeeList extends Vue {
  // 従業員一覧
  private currentEmployeeList: Array<Employee> = [];
  // 従業員数
  private employeeCount = 0;

  // ページの初期値
  private pageNum = 1;

  //名前検索用の入力変数
  private inputName = '';
  private searchedMessage = '';

  /**
   * Vuexストアのアクション経由で非同期でWebAPIから従業員一覧を取得する.
   *
   * @remarks
   * Vueインスタンスが生成されたタイミングで
   * Vuexストア内のアクションを呼ぶ(ディスパッチする)。
   * ライフサイクルフックのcreatedイベント利用。
   *
   * 取得してからゲットするため、async awaitを利用している。
   */
  async created(): Promise<void> {
    // もしログインボタンを押して遷移していなければ従業員一覧画面を表示させない
    if (this.$store.state.userStatus === '') {
      this.$router.push('/loginadmin');
      return;
    }

    await this.$store.dispatch('getEmployeeList');

    // 従業員一覧情報をVuexストアから取得
    // 非同期で外部APIから取得しているので、async/await使わないとGetterで取得できない
    // ページング機能実装のため最初の10件に絞り込み
    this.currentEmployeeList = this.$store.getters.getAllEmployees;
  }
  /**
   * 現在表示されている従業員一覧の数を返す.
   *
   * @returns 現在表示されている従業員一覧の数
   */
  get getEmployeeCount(): number {
    this.employeeCount = this['$store'].getters.getAllEmployeeCount;
    return this.employeeCount;
  }

  /**
   * ページ数の取得する.
   */
  get getPageNum(): number {
    this.pageNum = Math.ceil(this.employeeCount / 10);
    return this.pageNum;
  }

  /**
   * 指定したページに表示する従業員情報を取得する.
   */
  onchangepage(selectedPageNum: number): void {
    this.currentEmployeeList = this['$store'].getters.getEmployeesByPageNum(
      selectedPageNum
    );
  }

  /**
   * 名前検索をする.
   *
   *
   */
  searchEmployee(name: string): void {
    if (this.inputName == '') {
      this.searchedMessage = '１件もありませんでしたので全件表示をします';
      this.currentEmployeeList = this['$store'].getters.getAllEmployees;
      return;
    }
    //
    this.currentEmployeeList = this['$store'].getters.getSearchEmployeeByName(
      name
    );
    // this.currentEmployeeList =this['$store'].getters.getAllEmployees;
  }
}
</script>

<style scoped>
.searchForm {
  margin-bottom: 20px;
  width: 450px;
  margin: 0 auto;
}

.searchBtn {
  display: block;
  width: 150px;
  margin: 0 auto;
}
.search {
  border: 1px solid black;
}
</style>
