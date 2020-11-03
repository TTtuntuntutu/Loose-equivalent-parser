# Loose-equivalent-parser

宽松等价在JavaScript中是个迷惑的话题， [JavaScript-Equality-Table](https://dorey.github.io/JavaScript-Equality-Table/) 可整体查看：

> When using two equals signs for JavaScript equality testing, some funky conversions take place. 		



项目 Loose-equivalent-parser 侧重点不同。允许用户输入比较的两个对象，获取结果：

![结果](/img/result.png)

也允许获得可视化，宽松等价过程的转化：

![解析](/img/process.png)



技术栈方面，这是一个Vue 2.x搭建的SPA，使用了bootstrap-vue 中几个简单的组件。

Ok~ Just for fun!!!

## 解析

在解析过程遇到的问题与解决：

1. 宽松等价的两个“参与选手”的值最初是以字符串形式存储的，比如对象`{}`，存储的就是`"{}"`。如何从字符串形式获取正确的值？

   1. 第一个想到的是`JSON.parse`解析一下嘛。Well, it works in most cases. 除了`JSON.parse("undefined")`和`JSON.parse("{a:1}")`，想解析出`undefined`和`{a:1}`，结果报 `SyntaxError`

   2. 第二个想到的不得已的方法是`eval`。我先准备好变量，比如`let player`，紧接着`player = eval("xxx")`这样去使得`player`获取正确值。Well, it also works in almost every case. 除了`eval("{}")`的结果是`undefined`，`eval("{a:1}")`结果是`1`，因为`eval`在执行的时候没有将`{}`当作对象看待，而是一个块儿

   3. 呼之欲出的是最后的解决方法，和 2 类似，替换`player = eval("xxx")`为：

      ```javascript
      //假设值存储在 value 变量
      const value = "{a:1}"
      
      let player;
      
      eval(`
      	player = ${value}
      `)
      ```

      这样能获取正确的值了（如果是表达式，取表达式的结果），但是`eval`，看过YDKJS知道是一个很邪恶的存在。要是有更好的办法要告诉我呀！

2. 每一步的解析我都是生成一条字符串文本，扔到`code`标签去显示。现在`player`存储着正确的值，比如`{a:1}`，那要怎么转换成字符串的`"{a:1}"`？

   1. 第一个想到的是`JSON.stringify`。Well, it works in most cases. 除了两种情况，第一种情况是在处理对象，比如`{a:1}`，得到的字符串结果是`"{"a":1}"`；第二种情况是在处理`NaN`、`Infinity`、`-Infinity`时，结果都是`"null"`。这都不是期望的结果。

   2. 第二个当然是`toString`。Well，它可以帮忙解决`NaN`、`Infinity`和`-Infinity`。但是它无法处理`null`和`undefined`，因为这两个基本值没有可封箱的对象，也就无法调用`toString`方法；也无法处理数组和对象，处理的结果和预期相去甚远。

   3. 所以`JSON.stringify`和`toString`都不能够单独作为解决问题的方法。我给出的方法是结合两者，如果`player`是`number`类型则使用`toString`，是其他类型则使用`JSON.stringify`：

      ```javascript
      function getStringInfo(player1, player2) {
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
      ```

3. 在宽松等价时，`null`、`undefined` 除了与自身宽松等价结果为`true`，仅有和对方宽松等价结果为`true`。换言之，这里存在四种结果为`true`情况（考虑`player1`和`player2`情况），即 `null==undefined`、`undefined == true`、`null == null`和`undefined == undefined`。因为不想`if..else`多种情况判断，耍了一个`Boolean`隐含强制转换为`Number`的小手段：

   ```javascript
   function sumNullUndefined(player1, player2) {
         let contain = [];
         contain.push(typeof player1 === "object" && !player1);
         contain.push(typeof player1 === "undefined");
         contain.push(typeof player2 === "object" && !player2);
         contain.push(typeof player2 === "undefined");
         let sum = contain.reduce(
           (accumulator, currentValue) => accumulator + currentValue
         );
         return sum;
       }
   ```

   如果 `sum` 值为 `2` 说明结果`true`，值为 `1` 说明只有`null`、`undefined`其中一个，结果为`false`。这里还有一个小坑是用`typeof`判断`null`是得复合一个条件，即本身为`false`

## 另外说明

[YDKJS 抽象等价性 here](https://github.com/getify/You-Dont-Know-JS/blob/1ed-zh-CN/types%20%26%20grammar/ch4.md#抽象等价性)

讲到的4条规则：

- 比较：`string`和`number`
- 比较：任何东西与`boolean`
- 比较：`null`与`undefined`
- 比较：`object`与非`object`

我是按照这样的顺序去处理的，中间如果确定获得结果值，就跳出解析：

- 比较：`object`与非`object`
- 比较：`null`与`undefined`
- 比较：任何东西与`boolean`
- 比较：`string`和`number`

这对于获取结果值正确与否是不影响的，但对于这个顺序是可以打上问号的。比如`[]==true`是先转为`""==true`还是`[]==1`。

Anyway, remember it's just for fun. 并且帮助获取正确的结果和了解解析所涉及的规则。