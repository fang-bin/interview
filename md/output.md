### 1.

```javascript
const value = { number: 10 };

const multiply = (x = { ...value }) => {
  console.log(x.number *= 2);
};

multiply();
multiply();
multiply(value);
multiply(value);
```

对象的扩展运算符

对象的解构赋值用于从一个对象取值，相当于将目标对象自身的所有可遍历的（enumerable）、但尚未被读取的属性，分配到指定的对象上面。所有的键和它们的值，都会拷贝到新对象上面。

对象的扩展运算符（...）用于取出参数对象的所有可遍历属性，拷贝（浅拷贝）到当前对象之中。

###### 答案: `20 20 20 40`

### 2.

```javascript
const settings = {
  username: "lydiahallie",
  level: 19,
  health: 90
};

const data = JSON.stringify(settings, ["health", "level"]);
```

`JSON.stringify(value[, replacer[, space]])`

space 指定缩进用的空白字符串，用于美化输出（pretty-print）；如果参数是个数字，它代表有多少的空格；上限为10。该值若小于1，则意味着没有空格；如果该参数为字符串（当字符串长度超过10个字母，取其前10个字母），该字符串将被作为空格；如果该参数没有提供（或者为 null），将没有空格。

JSON.stringify的第二个参数是 替代者(replacer). 替代者(replacer)可以是个函数或数组，用以控制哪些值如何被转换为字符串。

如果替代者(replacer)是个 数组 ，那么就只有包含在数组中的属性将会被转化为字符串。成员的转换顺序与键在数组中的顺序一样。

而如果替代者replacer是个 函数`replacer((index, ele))`(函数应当返回JSON字符串中的value)，这个函数将被对象的每个属性都调用一遍。函数返回的值会成为这个属性的值，最终体现在转化后的JSON字符串中（译者注：Chrome下，经过实验，如果所有属性均返回同一个值的时候有异常，会直接将返回值作为结果输出而不会输出JSON字符串），而如果返回值为undefined，则该属性会被排除在外。

###### 答案 `"{"health":90, "level":19}"`

几种特殊情况:

```javascript
let obj = {a: 1, b: 2};
Object.defineProperty(obj, 'c', {
  configurable: true,
  writable: true,
  value: 3,
  enumerable: false,
});
obj[Symbol.for('fang')] = 1000;

console.log(JSON.stringify(obj));  //{"a":1,"b":2}
console.log(JSON.stringify(obj, [Symbol.for('fang'), 'c', 'b', 'a']));  //{"c":3,"b":2,"a":1}
console.log(JSON.stringify(obj, (index, ele) => 100));  //100
console.log(JSON.stringify(obj, (index, ele) => ele));  //{"a":1,"b":2} 如果返回的是undefined 则会直接过滤掉
```

### 3.

```javascript
const name = "Lydia";
age = 21;

console.log(delete name);
console.log(delete age);
```

delete 操作符用于删除对象的某个属性；成功返回`true`，失败返回`false`;如果没有指向这个属性的引用，那它最终会被释放。

上面的因为都是写在全局上下文中，所以其执行的是 `delete window.name;  delete window.age`，而 const 声明的对象，并没有绑定在window上。

ES6 提供了为操作对象的新API `Reflect`，让Object操作都变成函数行为。某些Object操作是命令式，比如name in obj和delete obj\[name]，而Reflect.has(obj, name)和Reflect.deleteProperty(obj, name)让它们变成了函数行为。

###### 答案 `false true`;

### 4.

```javascript
// counter.js
let counter = 10;
export default counter;

// index.js
import myCounter from "./counter";
myCounter += 1;
console.log(myCounter);
```

ES6模块输出的是引用，重新赋值会编译报错，不能修改其变量的指针指向，但可以改变内部属性的值；

CommonJS模块输出的是拷贝（浅拷贝），可以重新赋值，可以修改指针指向；

CommonJS模块是运行时加载，ES6模块是编译时输出接口

###### ES6 Moudle

* ES6模块中的值属于动态只读引用。
* 对于只读来说，即不允许修改引入变量的值，import的变量是只读的，不论是基本数据类型还是复杂数据类型。当模块遇到import命令时，就会生成一个只读引用。等到脚本真正执行时，再根据这个只读引用，到被加载的那个模块里面去取值。
* 对于动态来说，原始值发生变化，import加载的值也会发生变化。不论是基本数据类型还是复杂数据类型。
* 循环加载时，ES6模块是动态引用。只要两个模块之间存在某个引用，代码就能够执行。

###### CommonJS

* 对于基本数据类型，属于复制。即会被模块缓存。同时，在另一个模块可以对该模块输出的变量重新赋值。
* 对于复杂数据类型，属于浅拷贝。由于两个模块引用的对象指向同一个内存空间，因此对该模块的值做修改时会影响另一个模块。
* 当使用require命令加载某个模块时，就会运行整个模块的代码。
* 当使用require命令加载同一个模块时，不会再执行该模块，而是取到缓存之中的值。也就是说，CommonJS模块无论加载多少次，都只会在第一次加载时运行一次，以后再加载，就返回第一次运行的结果，除非手动清除系统缓存。
* 当循环加载时，脚本代码在require的时候，就会全部执行。一旦出现某个模块被"循环加载"，就只输出已经执行的部分，还未执行的部分不会输出。

###### 答案 `Error`

### 5.

```javascript
function Car() {
  this.make = "Lamborghini";
  return { make: "Maserati" };
}

const myCar = new Car();
console.log(myCar.make);
```

构造函数如果有`return`

* return 基础类型，则没有任何影响
* return 引用类型(object、array、map、set、date、error、regexp等)，就返回这个引用类型。

可以参考 new 的实现：

```javascript
function mockNew (){
  const Con = Array.prototype.shift.call(arguments);
  let obj = Object.create(Con.prototype);
  let ret = Con.apply(obj, arguments);
  // 这里优先返回构造函数返回的对象
  return ret instanceof Object ? ret : obj;
}
```

###### 答案 `"Maserati"`

### 6.

```javascript
const num = parseInt("7*6", 10);
```

`parseInt(string[, radix])` 解析一个字符串并返回指定基数（string以radix进制解析）的十进制整数。

如果 radix 是 undefined、0 或未指定：
* 如果输入的 string以 "0x"或 "0x"（一个0，后面是小写或大写的X）开头，那么radix被假定为16，字符串的其余部分被当做十六进制数去解析。
* 如果输入的 string以 "0"（0）开头， radix被假定为8（八进制）或10（十进制）。具体选择哪一个radix取决于实现。ECMAScript 5 澄清了应该使用 10 (十进制)，但不是所有的浏览器都支持。因此，在使用 parseInt 时，一定要指定一个 radix
* 如果输入的 string 以任何其他值开头， radix 是 10 (十进制)。

###### 答案 `7`

### 7.

```javascript
let person = { name: "Lydia" };
const members = [person];
person = null;

console.log(members);
```

`const members = [person]`相当于`const members[0] = person`;

###### 答案 `[{ name: "Lydia" }]`

### 8.

```javascript
const firstPromise = new Promise((res, rej) => {
  setTimeout(res, 500, "one");
});

const secondPromise = new Promise((res, rej) => {
  setTimeout(res, 100, "two");
});

Promise.race([firstPromise, secondPromise]).then(res => console.log(res));
```

`window.setTimeout(function[, delay, arg1, arg2, ...])`

`window.setTimeout(function[, delay])`

`window.setTimeout(code[, delay])`

`code` 是一个可选语法，可以使用字符串而不是function ，在delay毫秒之后编译和执行字符串。（不推荐，和`eval`一样，有安全风险）

`arg1...argN` 附加参数，一旦定时器到期，它们会作为参数传递给function。

引申：

`eval(string)` 函数会将传入的字符串当做 JavaScript 代码进行执行。返回字符串中代码的返回值。如果返回值为空，则返回 undefined。

eval() 是一个危险的函数， 它使用与调用者相同的权限执行代码。如果用 eval() 运行的字符串代码被恶意方（不怀好意的人）修改，最终可能会在网页/扩展程序的权限下，在用户计算机上运行恶意代码。

###### 答案 `two`

### 9.

```javascript
!!null
!!''
!!1
```

js中的8个falsy值: `false 0 -0 NaN null '' undefined 0n`

```javascript
0 == -0;  //true
0 === -0;  //true
NaN === NaN; //false
NaN == NaN;  //false

Object.is(0, -0); //false;
Object.is(NaN, NaN); //false;

let set = new Set(); // {}
set.add(NaN); // {NaN}
set.add(NaN);  //{NaN}
set.add(0);  //{NaN, 0}
set.add(-0);  //{NaN, 0}
```

0 和 -0基本没有什么太多的负面影响，只有 `number / 0`时会判断失误：

```javascript
(1 / 0) === Infinity;
(1 / -0) === -Infinity;
```

所以尽量使用 `Object.is()` 来进行判断。`Object.is()` 内部的实现为：

```javascript
function is (x, y) {
  // 针对 0 不等于 -0
  if (x === y) return x !== 0 || y !== 0 || (1 / x === 1 / y);
  // 针对 NaN 等于 NaN
  else return x !== x && y !== y;
}
```

###### 答案 `false false true`

### 10.

```javascript
const obj = { 1: 'a', 2: 'b', 3: 'c' }
const set = new Set([1, 2, 3, 4, 5])

obj.hasOwnProperty('1');
obj.hasOwnProperty(1);
set.has('1');
set.has(1);
```

hasOwnProperty() 的参数为 number 时，会把 number 转化为 string。（object的键只有String和Symbol）

Set 内部判断两个值是否不同，使用的算法叫做“Same-value-zero equality”，它类似于精确相等运算符（===），主要的区别是向 Set 加入值时认为NaN等于自身，而精确相等运算符认为NaN不等于自身。（Set的判断，0和-0是一样的）

###### 答案 `true true false true`

### 11.

多长时间消失？

```javascript
sessionStorage.setItem('cool_secret', 123);
```

###### sessionStorage

sesssionStorage保存在内存中，是会话级别的本地保存，其**标签关闭数据也会被清除。**

页面会话在浏览器打开期间一直保持，并且(不是手动关闭之后重新打开)重新加载或恢复页面仍会保持原来的页面会话。

sessionStorage只有同一浏览器，同一窗口的同源文档才能共享数据（同一标签窗口不同的iframe也可以共享数据）。

###### localStorage

 localStroage以文件的形式存储在本地（硬盘）localStorage存储的数据是永久性的，除非用户人为删除否则一直存在。

 localStorage同一浏览器中，同源文档可以共享localStorage数据。

### 12.

```javascript
function getAge() {
  'use strict'
  age = 21;
  console.log(age);
}

getAge();
```

###### 答案 `ReferenceError`

#### JS中的几种报错

###### SyntaxError

```javascript
// SyntaxError: 语法错误
// 1) 变量名不符合规范
var 1       // Uncaught SyntaxError: Unexpected number
var 1a       // Uncaught SyntaxError: Invalid or unexpected token
// 2) 给关键字赋值
function = 5     // Uncaught SyntaxError: Unexpected token =
```

###### ReferenceError

```javascript
// ReferenceError：引用错误(要用的变量没找到)
// 1) 引用了不存在的变量
a()       // Uncaught ReferenceError: a is not defined
console.log(b)     // Uncaught ReferenceError: b is not defined
// 2) 给一个无法被赋值的对象赋值
console.log("abc") = 1   // Uncaught ReferenceError: Invalid left-hand side in assig
```

###### TypeError

```javascript
// TypeError: 类型错误(调用不存在的方法)
// 变量或参数不是预期类型时发生的错误。比如使用new字符串、布尔值等原始类型和调用对象不存在的方法就会抛出这种错误，因为new命令的参数应该是一个构造函数。
// 1) 调用不存在的方法
123()        // Uncaught TypeError: 123 is not a function
var o = {}
o.run()        // Uncaught TypeError: o.run is not a function
// 2) new关键字后接基本类型
var p = new 456      // Uncaught TypeError: 456 is not a constructor
```

###### RangeError

```javascript
// RangeError: 范围错误(参数超范围)
// 主要的有几种情况，第一是数组长度为负数，第二是Number对象的方法参数超出范围，以及函数堆栈超过最大值。
// 1) 数组长度为负数
[].length = -5      // Uncaught RangeError: Invalid array length
// 2) Number对象的方法参数超出范围
var num = new Number(12.34)
console.log(num.toFixed(-1))   // Uncaught RangeError: toFixed() digits argument must be between 0 and 20 at Number.toFixed
// 说明: toFixed方法的作用是将数字四舍五入为指定小数位数的数字,参数是小数点后的位数,范围为0-20.
```

###### EvalError

```javascript
// EvalError: 非法调用 eval()
// 在ES5以下的JavaScript中，当eval()函数没有被正确执行时，会抛出evalError错误。例如下面的情况：
var myEval = eval;
myEval("alert('call eval')");
// 需要注意的是：ES5以上的JavaScript中已经不再抛出该错误，但依然可以通过new关键字来自定义该类型的错误提示。以上的几种派生错误，连同原始的Error对象，都是构造函数。开发者可以使用它们，认为生成错误对象的实例。
new Error([message[fileName[lineNumber]]])
// 第一个参数表示错误提示信息，第二个是文件名，第三个是行号。
```

###### URIError

```javascript
// URIError: URI不合法
// 主要是相关函数的参数不正确。
decodeURI("%")     // Uncaught URIError: URI malformed at decodeURI
// jzz
```








