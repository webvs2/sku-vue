<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png" />
    <h2>口红🩰💄</h2>
    <div class="box">
      <div
        class="item"
        v-for="(item, index) in vnodeData"
        :key="index"
        style="margin-top: 10px"
      >
        <div style="" class="titleBox">
          {{ item.label }}
        </div>
        <div style="display: inline-block" class="itemBox">
          <a-radio-group button-style="solid" :value="radioValue[index]">
            <template v-for="(j, i) in item.value">
              <a-radio-button
                :value="j"
                class="radio-s"
                :disabled="disabledFn(item.key + j, index, j)"
                :name="item.key"
                @click="stageFn(item.key, index, j)"
                :key="'a' + i"
                v-if="!!j"
              >
                {{ j }}
              </a-radio-button>
            </template>
          </a-radio-group>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// import {
//   getSpuDetail

// } from '@/request/apis.js'
import _ from "lodash";
export default {
  components: {},
  data() {
    return {
      // spu: '',
      radioValue: [],
      count: 1,
      vnodeData: [
        {
          //可以选择的选项
          label: "颜色",
          key: "a1",
          value: ["红色", "黄色", "蓝色"],
        },
        {
          label: "size",
          key: "a2",
          value: ["大", "中", "小"],
        },
        {
          label: "包装",
          key: "a3",
          value: ["无", "有", "精美"],
        },
      ],
      results: [
        {
          //可以选的sku
          skuCode: "sku1",
          saleAttrObj: {
            a1: "红色",
            a2: "中",
            a3: "精美",
          },
        },

        {
          skuCode: "sku2",
          saleAttrObj: {
            a1: "黄色",
            a2: "中",
            a3: "无",
          },
        },
        {
          skuCode: "sku3",
          saleAttrObj: {
            a1: "红色",
            a2: "中",
            a3: "无",
          },
        },
        {
          skuCode: "sku3",
          saleAttrObj: {
            a1: "蓝色",
            a2: "大",
            a3: "有",
          },
        },
      ], // 结果集
      // decision: [], // 决策集
      decision: {}, // 决策集
      matrix: [], // 矩阵
      matrixObj: {}, // 矩阵
      // matrixObj: {}, // 矩阵
      matrixGist: {}, // 矩阵判断依据
      // isAllCheck: false
    };
  },
  props: {
    spu: {
      type: String,
      default: "",
    },
  },
  watch: {
    spu: function () {
      this.init();
    },
  },
  computed: {
    isInitial() {
      return !!_.keys(this.matrixObj).length;
    },
    isAllCheck() {
      return _.keys(this.decision).length === this.vnodeData.length;
    },
  },
  created() {
    this.init();
  },
  methods: {
    disabledFn(key, index, value) {
      return !!this.isInitial && !this.matrixObj[key];
    },
    destroy() {
      return false;
    },
    init() {
      const spu = this.spu;
      // / ////////////清空操作
      this.matrix = [];
      this.decision = {};
      this.matrixGist = {};
      this.radioValue = [];
      this.matrixObj = {}; // 矩阵
    },

    // 阶段
    stageFn(key, index, con) {
      // key 当前的唯一标识
      // index 表明第几个阶段
      const { results, radioValue } = this;

      if (radioValue[index] === con) {
        this.radioValue[index] = "";
        this.$delete(this.decision, key);
      } else {
        this.radioValue[index] = con;
        this.$set(this.decision, key, con);
      }
      const decisionObj = results.map((item) => item.saleAttrObj);
      const rowState = {};
      console.log(this.results, "this.decision");
      _.keys(this.decision).map((item) => {
        const oldDecisionData = _.cloneDeep(this.decision);
        delete oldDecisionData[item];
        const data2 = _.filter(decisionObj, oldDecisionData);
        console.log("data2", data2);
        data2.map((c, i) => {
          rowState[[item] + c[item]] = true;
        });
      });

      let matrix = _.filter(decisionObj, this.decision);
      if (_.keys(this.decision).length === 0) {
        matrix = [];
      }
      this.$emit("change", matrix, this.isAllCheck);
      this.matrix = matrix;
      const ad = _.reduce(
        matrix,
        function (sum, n, key) {
          const newN = _.omit(n, ["index", "skuCode"]);
          const data = _.mapKeys(newN, function (v, k) {
            return k + v;
          });
          return {
            ...data,
            ...sum,
          };
        },
        {}
      );
      this.matrixObj = _.assign(ad, rowState);
    },
  },
};
</script>

<style lang="scss" scoped="scoped">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  text-align: center;
  width: 100%;
  height: 100%;
}
.box {
  width: 500px;
  border: solid red 1px;
  padding-left: 0px;
  margin-top: 10px;
  // .detail-option{
  // margin-top: 20px;
  position: relative;

  .item {
    // display: flex;
    // display: -webkit-flex;
    /* Safari */
    // border: solid red 1px;
    // border: solid red 1px;
    position: relative;
    padding-left: 100px;

    // flex-direction: row;
    // align-items: center;
    .titleBox {
      position: absolute;
      top: calc((100% - 28px) / 2);
      font-size: 14px;
      width: 100px;
      // padding-left: 16px;
      font-family: PingFangSC-Regular, PingFang SC;
      left: 0;
    }

    .itemBox {
      display: flex;
      display: -webkit-flex;
      flex-wrap: wrap;
      justify-content: center;
    }
  }

  // }
}

.radio-s {
  border-radius: 4px;
  margin-right: 10px;
  margin-bottom: 10px;
  font-family: PingFangSC-Regular, PingFang SC;

  // border: solid red 1px;
  border: 1px solid #d9d9d9 !important ;
  &::last-child {
    margin-bottom: 0px;
  }

  &::before {
    display: none !important;
  }
}
</style>


