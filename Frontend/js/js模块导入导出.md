# JS模块导入导出大全——module.exports、exports、export和export default的使用和区别



## **module.exports、exports、export**有什么区别？

这三个东西其实是**两个**概念的区分： 先来看一个表格：

| 规范名称 | 导出关键词               | 导入关键词 | 重命名变量 | 集体导出 |
| -------- | ------------------------ | ---------- | ---------- | -------- |
| commonjs | module.exports / exports | require    | {A ：B }   | 不支持   |
| esm      | export                   | import     | as         | * as xxx |

这样看其实还是挺清晰的，不同规范使用的关键词和特性不同而已，我们在使用这些关键词的时候区分自己是什么环境下写的代码。

- [nodejs](https://zhida.zhihu.com/search?content_id=195005095&content_type=Article&match_order=1&q=nodejs&zhida_source=entity)（webpack，babel）->commonjs,
- 浏览器（vue的script标签 或 html 中带type="module"的script标签中）-> esm。

1. `module.exports、exports`是一伙的，他们都是基于`commonjs`规范来的。
2. `export`是基于`es6`的esm(`**ECMA Script Modules**`)规范来的。

## commonjs规范:

> CommonJS 规范是为了解决 JavaScript 的作用域问题而定义的模块形式，可以使每个模块它自身的[命名空间](https://zhida.zhihu.com/search?content_id=195005095&content_type=Article&match_order=1&q=命名空间&zhida_source=entity)中执行。该规范的主要内容是，模块必须通过 `module.exports` 导出对外的变量或接口，通过 `require()` 来导入其他模块的输出到当前模块作用域中。

列子：

```text
// moduleA.js
module.exports.double = function( value ){
    return value * 2;
}
// moduleB.js
var { double } = require('./moduleA');
var res = double(4);
console.log(res) //8
```

为了方便，`**某些情况下**module.exports`可以简写为`exports`。其实一开始`exports`就是`module.exports`的引用。

```text
module.exports===exports //true
```

所以上面的`moduleA.js` ,可以这样省略掉`module`

```text
// moduleA.js
exports.double = function (value) {
    return value * 2;
}
// moduleB.js
var { double } = require('./moduleA');
var res = double(4);
console.log(res) //8
```

当 `module.exports`后面跟着赋值语句时，不能省略`module.`，为什么？我们接着看

一开始 `module.exports`全等于 `exports` 我们可以看成在文件`开头`有这样的代码：

```text
module.exports = {};//方便理解 一开始是空对象
let exports = module.exports;
console.log(module.exports===exports)//true
```

所以在`exports`对象`{}`上添加任何的新属性，其实就是在`module.exports`上添加属性，因为我们知道`node`中对象也是[引用类型](https://zhida.zhihu.com/search?content_id=195005095&content_type=Article&match_order=1&q=引用类型&zhida_source=entity)的。

如：

```text
exports.double = function (value) {
    return value * 2;
}

exports.PAI = 3.1415926535897932384626 //233，我可以背这么多位~
exports.add = function (a, b) {
    return a + b;
}
```

**危！给exports赋值就不行了**

```text
module.exports = {};//方便理解 一开始是空对象
let exports = module.exports;

//给exports 赋值会切断和 module.exports 的联系

//这样 不要这样做~！
exports = {
    name: 'zhangsan'
}
//或这样  不要这样做~！
exports = function name(a) {}
//或这样  不要这样做~！
exports = 3.1415926535897932384626
console.log(module.exports); // 打印：{} 回到初始{}值
console.log(exports);  // 打印：3.1415926535897932384626

console.log(module.exports === exports); //false 你走的独木桥 我走我的阳关道 已经莫得关系了
```

如果要使用赋值来导出，可以这样导出一个对象：

```text
// moduleA.js
module.exports = {
    name:'法外狂徒张三',
    age:'19',
    wife:'隔壁小红'
}

// moduleB.js
let { name, wife } = require('./moduleA');
console.log(name,age) //法外狂徒张三 隔壁小红
```

这样`module.exports`就能保证获取到值。

**如何重命名**

commonjs中重命名导出的变量用( **`：`**)符号：

```text
let { name, wife：girlfriend } = require('./moduleA');
```

## esm（**`ECMA Script Modules`**）规范

现在来说我们在编写vue或者react项目中熟悉的es6 module规范。

> esm 是将 javascript 程序拆分成多个单独模块，并能按需导入的标准。和`webpack`，`babel`不同的是，esm 是 javascript 的标准功能，在浏览器端和 nodejs 中都已得到实现。使用 [esm](https://zhida.zhihu.com/search?content_id=195005095&content_type=Article&match_order=7&q=esm&zhida_source=entity) 的好处是浏览器可以最优化加载模块，比使用库更有效率。通过`import`, `export`语法实现模块变量的导入和导出
> 划重点：`commonjs`中使用`require` 和 `modelue.exports`实现模块的[导入导出](https://zhida.zhihu.com/search?content_id=195005095&content_type=Article&match_order=1&q=导入导出&zhida_source=entity)，`esm` 使用 `import`, `export`导入导出。

`esm`的`export`支持：`let、var、const、function、class`等 以下都是正确的导出方式：

```text
var person = {
    name: '猫小白',
    text: a,
}
export {
    person,
}
export function showAge() {
    console.log(person.age);
}
export let city = '成都'
export const PAI = '3.141592653'
```

> 注意：`export`命令规定的是对外的接口，必须与模块内部的变量建立一一对应关系。

```text
// 错误1
export 3.14;

// 错误2
var api = 3.14;
export api;
```

> 这俩种写法都是错误的，因为export后面直接跟了一个具体的，外部无法通过一个特定的标识（变量）来获取这个值。

## export default 默认导出

和`commonjs`规范先进一点的是 `esm` 提供了一种默认导出的方式。`export defaul`t后面可以直接跟随：变量、具体值、`function、calss`。

```text
//正确
let a = '张三';

export default a
//正确
export default 3.14

export default function (a) {
    return a * a
}
//正确
let sum = function (a, b) {
    return a + b
}
export default sum
//正确
function calc(a, b) {
    return a + b
}
export default calc
```

**如何重命名**

`ESM`中提供了`as`关键词对导出的变量重命名，防止变量名重复。

```text
import { wife as girlfrend } from "./moduleA.js";
```

想要导出所有方法或变量为指定的一个变量时，可以用 `import * as mod from "xxxx"`

```text
import * as tool from "./moduleA.js";
console.log(tool.API) //3.141592
```

`ESM`模块规范规定，在`html`中 `script`标签设置`type='module'`可以书写`ESM`模块代码，包括导入功能：

```text
<script type="module">
    import { name,wife } from "./es6/moduleA.js"
    console.log(name,wife); //法外狂徒张三 小红
    //...其它代码
</script>
```