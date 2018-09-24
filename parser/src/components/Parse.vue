<template>
    <div class="parse">
        <b-container>
            <b-row>
                <b-col cols="4">
                    <div class="forms">
                        <!-- <p>输入：</p> -->
                        <b-form-input 
                            v-model="fstOperObj"
                            type="text"></b-form-input>
                        <p>==</p>
                        <b-form-input 
                            v-model="secOperObj"
                            type="text"></b-form-input>
                    </div>
                </b-col>
                <b-col cols="8">
                    <div class="btns">
                        <b-button 
                            variant="primary"
                            @click="parse">
                            解析
                        </b-button>
                        <b-button 
                            variant="primary"
                            @click="getResult">
                            获取结果
                        </b-button>
                    </div>
                </b-col>
            </b-row>
            <b-row>
                <b-col cols="4">
                    <div class="forms">
                        <!-- <p>预测输出结果：</p> -->
                        <b-form-select v-model="preValue" :options="options" class="mb-3">
                        </b-form-select>
                    </div>
                </b-col>
                <b-col cols="8">
                    <div class="texts">
                        <p>{{resTag}}</p>
                        <p v-show="resNote">正确结果是: {{res}}</p>
                        <p v-show="!resNote">您的预测结果是正确的</p>
                    </div>
                </b-col>
            </b-row>
            <b-row>
                <b-col cols="12">
                    <b-card>
                        <b-list-group v-for="(parseNote,index) in parseNotes" :key="index">
                            <code>
                                <b-list-group-item>{{parseNote}}</b-list-group-item>
                            </code>
                        </b-list-group>
                    </b-card>
                </b-col>
            </b-row>
        </b-container>
    </div>
</template>

<script>
export default {
  name: "Parse",
  data() {
    return {
      fstOperObj: "",
      secOperObj: "",
      preValue: true,
      options: [
        { value: true, text: "Your answer is true" },
        { value: false, text: "Your answer is false" }
      ],
      resTag: "✔",
      resNote: false,
      res: true,
      parseNotes: []
    };
  },
  methods: {
    parse() {
      //信息清空
      while (this.parseNotes.length > 0) {
        this.parseNotes.pop();
      }
      let player1, player2;
      (player1 = eval(this.fstOperObj)), (player2 = eval(this.secOperObj));

      //如果两个都是对象，直接游戏结束(注意排除都是null的情况)
      if (
        typeof player1 === "object" &&
        player1 &&
        (typeof player2 === "object" && player2)
      ) {
        this.parseNotes.push(
          `${this.fstOperObj} == ${this.secOperObj} //false`
        );
        return;
      }
      //抽象等价性：存在object,ToPrimitive(object)
      if (typeof player1 === "object" && player1) {
        player1 = this.toPrimitive(player1);
      }
      if (typeof player2 === "object" && player2) {
        player2 = this.toPrimitive(player2);
      }

      //抽象等价性：考虑 null 与 undefined
      let sum = this.sumNullUndefined(player1, player2);
      if (sum === 2) {
        this.parseNotes.push(`${this.fstOperObj} == ${this.secOperObj} //true`);
        return;
      } else if (sum === 1) {
        this.parseNotes.push(
          `${this.fstOperObj} == ${
            this.secOperObj
          } //false 除本身以外，null只与undefined宽松等价相等`
        );
        return;
      }

      //抽象等价性： 任何东西与boolean
      if (typeof player1 === "boolean" && typeof player2 === "boolean") {
        this.parseNotes.push(
          `${this.fstOperObj} == ${this.secOperObj} //显而易见`
        );
        return;
      } else if (typeof player1 === "boolean") {
        player1 = player1 === true ? 1 : 0;
        this.parseNotes.push(`${player1} == ${this.secOperObj}`);
      } else if (typeof player2 === "boolean") {
        player2 = player2 === true ? 1 : 0;
        this.parseNotes.push(`${this.fstOperObj} == ${player2}`);
      }

      //抽象等价性：string与number
      if (typeof player1 === "string" && typeof player2 === "number") {
        let num = parseFloat(player1);
        if (JSON.stringify(num) === player1) {
          player1 = num;
        } else {
          player1 = NaN;
        }
        this.parseNotes.push(`${player1} == ${this.secOperObj} //字符串与数字比较，字符串转换为数字`);
      } else if (typeof player1 === "number" && typeof player2 === "string") {
        let num = parseFloat(player2);
        if (JSON.stringify(num) === player2) {
          player2 = num;
        } else {
          player2 = NaN;
        }
        this.parseNotes.push(`${this.fstOperObj} == ${player2} //字符串与数字比较，字符串转换为数字`);
      } else {
        this.parseNotes.push(`${this.fstOperObj} == ${this.secOperObj}`);
      }
      console.log(player1, player2);
    },
    getResult() {
      let player1, player2;
      (player1 = eval(this.fstOperObj)), (player2 = eval(this.secOperObj));
      let res = player1 == player2;
      if (res === this.preValue) {
        this.resTag = "✔";
        this.resNote = false;
        this.res = true;
      } else {
        this.resTag = "❌";
        this.resNote = true;
        this.res = false;
      }
    },
    toPrimitive(obj) {
      let value = obj.valueOf();
      let type = typeof value;
      if (type === "string" || type === "number" || type === "boolean") {
        return value;
      } else {
        return obj.toString();
      }
    },
    sumNullUndefined(player1, player2) {
      let contain = [];
      contain.push(typeof player1 === "object" && !player1);
      contain.push(typeof player1 === "undefined");
      contain.push(typeof player2 === "null");
      contain.push(typeof player2 === "undefined" && !player2);
      let sum = contain.reduce(
        (accumulator, currentValue) => accumulator + currentValue
      );
      return sum;
    }
  }
};
</script>

<style lang="less">
.parse {
  .row {
    margin: 20px auto;
    .forms {
      display: flex;
      p,
      input {
        margin-right: 5px;
      }
    }
    .btns > button {
      margin-right: 15px;
    }
    .texts {
      p {
        display: inline-block;
        margin-right: 20px;
      }
    }
  }
}
</style>


