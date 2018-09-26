<template>
    <div class="parse">
        <b-container>
            <b-row>
                <b-col cols="4">
                    <div class="forms">
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

      //结果显示
      this.getResult();

      let player1, player2;
      eval(`
        player1 = ${this.fstOperObj};
        player2 = ${this.secOperObj};
      `);

      //初始！
      this.parseNotes.push(
        `${this.getStringInfo(player1, player2)[0]} == ${
          this.getStringInfo(player1, player2)[1]
        }`
      );

      //如果两个都是对象，直接游戏结束(注意排除都是null的情况)
      if (
        typeof player1 === "object" &&
        player1 &&
        (typeof player2 === "object" && player2)
      ) {
        this.parseNotes.push(
          `${this.getStringInfo(player1, player2)[0]} == ${
            this.getStringInfo(player1, player2)[1]
          } //两个都是引用值，直接false`
        );
        this.parseNotes.push(`false`);
        return;
      }
      //抽象等价性：存在object,ToPrimitive(object)
      if (typeof player1 === "object" && player1) {
        player1 = this.toPrimitive(player1);
        this.parseNotes.push(
          `${this.getStringInfo(player1, player2)[0]} == ${
            this.getStringInfo(player1, player2)[1]
          } //对象转化为简单值`
        );
      }
      if (typeof player2 === "object" && player2) {
        player2 = this.toPrimitive(player2);
        this.parseNotes.push(
          `${this.getStringInfo(player1, player2)[0]} == ${
            this.getStringInfo(player1, player2)[1]
          } //对象转化为简单值`
        );
      }

      //到这里，两个player均为简单值

      //抽象等价性：考虑 null 与 undefined
      let sum = this.sumNullUndefined(player1, player2);
      if (sum === 2) {
        this.parseNotes.push(
          `${JSON.stringify(player1)} == ${JSON.stringify(player2)} //true`
        );
        this.parseNotes.push(`true`);
        return;
      } else if (sum === 1) {
        this.parseNotes.push(
          `${this.getStringInfo(player1, player2)[0]} == ${
            this.getStringInfo(player1, player2)[1]
          } //false; 唯有null、undefined互相宽松等价相等`
        );
        this.parseNotes.push(`false`);
        return;
      }

      //抽象等价性： 任何东西与boolean
      if (typeof player1 === "boolean" && typeof player2 === "boolean") {
        this.parseNotes.push(
          `${this.getStringInfo(player1, player2)[0]} == ${
            this.getStringInfo(player1, player2)[1]
          } //同类型比较，显而易见`
        );
        this.parseNotes.push(`${player1 == player2}`);
        return;
      } else if (typeof player1 === "boolean") {
        player1 = player1 === true ? 1 : 0;
        this.parseNotes.push(
          `${this.getStringInfo(player1, player2)[0]} == ${
            this.getStringInfo(player1, player2)[1]
          } //布尔值转化为数字`
        );
      } else if (typeof player2 === "boolean") {
        player2 = player2 === true ? 1 : 0;
        this.parseNotes.push(
          `${this.getStringInfo(player1, player2)[0]} == ${
            this.getStringInfo(player1, player2)[1]
          } //布尔值转化为数字`
        );
      }

      //抽象等价性：string与number
      if (typeof player1 === "string" && typeof player2 === "number") {
        player1 = +player1;
        this.parseNotes.push(
          `${this.getStringInfo(player1, player2)[0]} == ${
            this.getStringInfo(player1, player2)[1]
          } //字符串与数字比较，字符串转换为数字`
        );
      } else if (typeof player1 === "number" && typeof player2 === "string") {
        player2 = +player2;
        this.parseNotes.push(
          `${this.getStringInfo(player1, player2)[0]} == ${
            this.getStringInfo(player1, player2)[1]
          } //字符串与数字比较，字符串转换为数字`
        );
      } else {
        this.parseNotes.push(
          `${this.getStringInfo(player1, player2)[0]} == ${
            this.getStringInfo(player1, player2)[1]
          } //同类型比较，显而易见`
        );
      }

      this.parseNotes.push(`${player1 == player2}`);

      return;
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
      contain.push(typeof player2 === "object" && !player2);
      contain.push(typeof player2 === "undefined");
      let sum = contain.reduce(
        (accumulator, currentValue) => accumulator + currentValue
      );
      return sum;
    },
    getStringInfo(player1, player2) {
      let p1 =
        typeof player1 === "number"
          ? player1.toString()
          : JSON.stringify(player1);
      let p2 =
        typeof player2 === "number"
          ? player2.toString()
          : JSON.stringify(player2);
      return [p1, p2];
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


