## 记录es6一些特性的问题

### 函数的扩展

#### 1. 函数默认值的作用域

```javascript
// 一
let x = 1;
function test (y = x){
  let x = 2;  // x = 2; 也是一样的结果
  console.log(y);
}
test() //1

// 二
let foo = 'outer';
function bar(func = () => foo) {
  let foo = 'inner';
  console.log(func());
}
bar(); // outer

// 三
function foo(x, y = function() { x = 2; }) {
  var x = 3;
  y();
  console.log(x);
}
foo(); // 3
x; // 1

// 四
var x = 1;
function foo(x, y = function() { x = 2; }) {
  x = 3;
  y();
  console.log(x);
}
foo(); // 2
x; // 1
```

一旦设置了函数的默认值，函数进行声明初始化时，参数会形成一个单独的作用域。等到初始化结束，这个作用域就会消失，这种语法行为，在不设置参数默认值时，是不会出现的。

#### 2. 函数严格模式

ES5开始，函数内部可以设定严格模式。ES2016规定，**只要函数参数使用了默认值、解构赋值、或者扩展运算符，那么函数内部就不能显式设定为严格模式**，否则会报错。

```javascript
// 没问题
function test() {
  'use strict';
  ...
}

// 会报错
function test1(a = 1) {
  'use strict';
}
```

#### 3. 函数作用域

* 静态（词法）作用域：在函数声明（定义）时确定
* 动态作用域：在函数调用时确定

js是静态作用域，即在函数执行上下文的创建阶段就已经确定了作用域链。

但是js的普通函数里面的this机制却和动态作用于类似，在执行阶段确定this执行，不过箭头函数是在定义阶段就绑定了外部this的指向（箭头函数内部没有this，内部this指向外部的this，apply、bind、call也没办法改变箭头函数的this指向，不过可以通过改变箭头函数外部的this指向来改变箭头函数的this指向）。

#### 4. 尾调用优化 和 尾递归优化

#####  尾调用

尾调用（Tail Call）是函数式编程的一个重要概念，指某个函数的最后一步是调用另一个函数。

```javascript
const a = () => {};
const b = () => {
  a();
}
// 并不是尾调用
```

尾调用不一定出现在函数尾部，只要是最后一步操作即可。

```javascript
function f(x) {
  if (x > 0) {
    return m(x)
  }
  return n(x);
}
// 这是尾调用
```

**尾调用优化** 只保留内层函数的调用帧。

尾调用由于是函数的最后一步操作，所以不需要保留外层函数的调用帧，因为调用位置、内部变量等信息都不会再用到了，只要直接用内层函数的调用帧，取代外层函数的调用帧就可以了。

如果所有函数都是尾调用，那么完全可以做到每次执行时，调用帧只有一项，这将大大节省内存。这就是“尾调用优化”的意义。

注意，只有不再用到外层函数的内部变量，内层函数的调用帧才会取代外层函数的调用帧，否则就无法进行“尾调用优化”。

注意，目前只有 Safari 浏览器支持尾调用优化(我自己没有测试出来)，Chrome 和 Firefox 都不支持。

###### 尾调用的条件

ES6 的尾调用优化只在严格模式下开启，正常模式是无效的。

这是因为在正常模式下，函数内部有两个变量，可以跟踪函数的调用栈。

* func.arguments：返回调用时函数的参数。
* func.caller：返回调用当前函数的那个函数。

尾调用优化发生时，函数的调用栈会改写，因此上面两个变量就会失真。严格模式禁用这两个变量，所以尾调用模式仅在严格模式下生效。

##### 尾递归

函数调用自身，称为递归。如果尾调用自身，就称为尾递归。

递归非常耗费内存，因为需要同时保存成千上百个调用帧，很容易发生“栈溢出”错误（stack overflow）。但对于尾递归来说，由于只存在一个调用帧，所以永远不会发生“栈溢出”错误。

##### 尾递归优化实现

尾递归之所以需要优化，原因是调用栈太多，造成溢出，那么只要减少调用栈，就不会溢出。

可以采用“循环”换掉“递归”。

蹦床函数（trampoline）可以将递归执行转为循环执行。

```javascript
function trampoline(f) {
  while (f && f instanceof Function) {
    f = f();
  }
  return f;
}

function sum(x, y) {
  if (y > 0) {
    return sum.bind(null, x + 1, y - 1);  //将原来的递归函数，改写为每一步返回另一个函数。
  } else {
    return x;
  }
}

trampoline(sum(1, 100000));
// 100001
```

蹦床函数并不是真正的尾递归优化，下面的实现才是。

```javascript
function tco(f) {
  var value;
  var active = false;
  var accumulated = [];

  return function accumulator() {
    accumulated.push(arguments);
    if (!active) {
      active = true;
      while (accumulated.length) {
        value = f.apply(this, accumulated.shift());
      }
      active = false;
      return value;
    }
  };
}

var sum = tco(function(x, y) {
  if (y > 0) {
    return sum(x + 1, y - 1)
  }
  else {
    return x
  }
});

sum(1, 100000)
// 100001
// 妙啊
```

上面代码中，tco函数是尾递归优化的实现，它的奥妙就在于状态变量active。默认情况下，这个变量是不激活的。一旦进入尾递归优化的过程，这个变量就激活了。然后，每一轮递归sum返回的都是undefined，所以就避免了递归执行；而accumulated数组存放每一轮sum执行的参数，总是有值的，这就保证了accumulator函数内部的while循环总是会执行。这样就很巧妙地将“递归”改成了“循环”，而后一轮的参数会取代前一轮的参数，保证了调用栈只有一层。

### 数组的扩展

#### 1. 扩展运算符

扩展运算符背后调用的是遍历器接口（Symbol.iterator），如果一个对象没有部署这个接口，就无法使用扩展运算符。

#### 2. Array.from()

Array.from方法用于将两类对象转为真正的数组，返回一个新的浅拷贝的数组实例。

针对对象：类似数组的对象（array-like object）和可遍历（iterable）的对象（包括 ES6 新增的数据结构 Set 和 Map）。

`Array.from(arrayLike[, mapFn[, thisArg]])`

* `arrayLike` 想要转换成数组的伪数组对象或可迭代对象。
* `mapFn` 可选参数，如果指定了该参数，新数组中的每个元素会执行该回调函数。
* `thisArg` 可选参数，执行回调函数 mapFn 时 this 对象。

`Array.from()` 会将数组的空位转化为undefined，并不会忽视空位。

#### 3. Array.of()

Array.of()方法用于将一组值，转换为数组。

这个方法的主要目的，是弥补数组构造函数Array()的不足。因为参数个数的不同，会导致Array()的行为有差异。

#### 4. find() 和 findIndex()

这两个方法都可以发现NaN，弥补了数组的indexOf方法的不足。

* `Array.prototype.find(callback[, thisArg])` 返回第一个符合条件的数组成员，没有返回undefined。
* `Array.prototype.findIndex(callback[, thisArg])` 返回第一个符合条件的数组成员索引值，没有返回-1

`callback(ele, index, arr)`;  

ele：数组元素 

index：数组索引值

arr：查找的数组本身


注: NaN 在js中的比对很特殊，`NaN != NaN` `NaN !== NaN`

* 可以通过`Object.is()`来比对两者 `Object.is(NaN, NaN); //true;`
* Set类似于数组，但是成员的值都是唯一的，没有重复的值，它对NaN的值比对就是正确的。
* 数组新添的 `includes` 方法也可以准确比对 NaN
* findIndex比对可以在其函数参数中通过`Object.is()`来比对。

```javascript
[NaN].indexOf(NaN)
// -1

[NaN].findIndex(y => Object.is(NaN, y))
// 0
```

#### 5. 其他一些方法和影响

* `copyWithin(target[, start[, end]])`

  在当前数组内部，将指定位置的成员复制到其他位置（会覆盖原有成员），然后返回当前数组。(**会修改当前数组**)
  target（必需）：从该位置开始替换数据。如果为负值，表示倒数。
  start（可选）：从该位置开始读取数据，默认为 0。如果为负值，表示从末尾开始计算。
  end（可选）：到该位置前停止读取数据，默认等于数组长度。如果为负值，表示从末尾开始计算。

* `fill(ele[, start[, end]])`

  fill方法使用给定值，填充一个数组。另外两个参数指定填充的起始位置和结束位置。

  如果填充的类型为对象，那么被赋值的是同一个内存地址的对象，而不是深拷贝对象。

* `entries()`，`keys()` 和 `values()`

* `includes(ele[, start])`
  Array.prototype.includes方法返回一个布尔值，表示某个数组是否包含给定的值，与字符串的includes方法类似。
  第二个参数表示搜索的起始位置，默认为0。如果第二个参数为负数，则表示倒数的位置，如果这时它大于数组长度（比如第二个参数为-4，但数组长度为3），则会重置为从0开始。
  它支持对 NaN 的正确比对判断。

* `flat()`，`flatMap(callback[, thisArg])`
  flat() 方法会跳过数组的空位。
  flatMap()方法对原数组的每个成员执行一个函数（相当于执行Array.prototype.map()），然后对返回值组成的数组执行flat()方法。该方法返回一个新数组，不改变原数组。

  flatMap()只能展开一层数组。

* 数组的空位的处理

### 对象的扩展和新增的方法

#### 1. 属性名表达式

ES6 允许字面量定义对象时，用表达式作为对象的属性名，即把表达式放在方括号内。

```javascript
let propKey = 'foo';

let obj = {
  [propKey]: true,
  ['a' + 'bc']: 123,
  ['h' + 'ello']() {
    return 'hi';
  }
};
```

#### 2. 属性的可枚举性和遍历

`Object.getOwnPropertyDescriptor(obj, prop)`

对象的每个属性都有一个描述对象（Descriptor），用来控制该属性的行为。该方法可以获取该属性的描述对象。

**ES6 规定，所有 Class 的原型的方法都是不可枚举的。**

* `for in` 会遍历对象包括其原型链上所有可枚举性的属性(不包含 Symbol 属性)
* `hasOwnProperty` 不会去查找原型，只会检查属性是否在当前对象中
* `Object.keys() Object.values() Object.entries()` 不会去查找原型，只会遍历对象本身可枚举（不包含 Symbol）属性（属性对）
* `Object.getOwnPropertyNames()` 不会去查找原型，会遍历对象本身所有的属性（无论是否具有可枚举性）
* `Object.prototype.propertyIsEnumerable` 只会检查属性在该对象上是否可枚举
* `Object.getOwnPropertyDescriptors()` 获取一个对象的所有自身属性的描述符
* `Object.getOwnPropertyDescriptor()` 获取对象自身某个属性的描述符
* `Object.getOwnPropertySymbols()` 获取一个给定对象自身的所有 Symbol 属性的数组
* `Object.assign(target, ...source)`只拷贝自身的可枚举属性，会忽略`enumerable`为`false`的属性(仅限于source对象，target对象的不可枚举属性，也会拷贝)
* `JSON.stringify()` 只串行化对象自身的可枚举的属性
* `Reflect.ownKeys(obj)` Reflect.ownKeys返回一个数组，包含对象自身的（不含继承的）所有键名，不管键名是 Symbol 或字符串，也不管是否可枚举。

`for in` `Object.keys()` `Object.getOwnPropertyNames()` `Object.getOwnPropertySymbols()` `Reflect.ownKeys()`遍历对象的键名，都遵守同样的属性遍历的次序规则:

* 首先遍历所有数值键，按照数值升序排列。
* 其次遍历所有字符串键，按照加入时间升序排列。
* 最后遍历所有 Symbol 键，按照加入时间升序排列。

```javascript
Reflect.ownKeys({ [Symbol()]:0, b:0, 10:0, 2:0, a:0 })
// ['2', '10', 'b', 'a', Symbol()]
```

#### 3. super

**super关键字表示原型对象时，只能用在对象的方法之中。(只有对象方法的简写法可以让 JavaScript 引擎确认，定义的是对象的方法)**

```javascript
const proto = {
  foo: 'hello'
};
const obj = {
  foo: 'world',
  find() {
    return super.foo;
  }
};

Object.setPrototypeOf(obj, proto);
obj.find() // "hello"
```

```javascript
const proto = {
  x: 'hello',
  foo() {
    console.log(this.x);
  },
};
const obj = {
  x: 'world',
  foo() {
    super.foo();
  }
}

Object.setPrototypeOf(obj, proto);
obj.foo() // "world"
```

#### 4. 链判断运算符

ES2020 引入了“链判断运算符”（optional chaining operator）`?.`

左侧的对象是否为null或undefined。如果是的，就不再往下运算，而是返回undefined。

```javascript
const firstName = message?.body?.user?.firstName || 'default';  //左侧的对象是否为null或undefined。如果是的，就不再往下运算，而是返回undefined
const fooValue = myForm.querySelector('input[name=foo]')?.value;

iterator.return?.();   //判断对象方法是否存在，如果存在就立即执行。
```

特点:

* 短路机制
* delete 运算符（可以直接写在 delete 后面，如果链判断运算符返回undefined，不会进行 delete 运算，反之则进行delete运算）
* 括号的影响 (如果属性链有圆括号，链判断运算符对圆括号外部没有影响，只对圆括号内部有影响。一般来说，使`·?.`运算符的场合，不应该使用圆括号。)
* 报错的场合
  * 构造函数 (`new a?.()`)
  * 链判断运算符的右侧有模板字符串 (a?.\`{b}\`)
  * 链判断运算符的左侧是 super (`super?.()`)
  * 链运算符用于赋值运算符左侧 (`a?.b = c`)
* 右侧不得为十进制数值
为了保证兼容以前的代码，允许foo?.3:0被解析成foo ? .3 : 0，因此规定如果?.后面紧跟一个十进制数字，那么?.不再被看成是一个完整的运算符，而会按照三元运算符进行处理，也就是说，那个小数点会归属于后面的十进制数字，形成一个小数。

#### 5. Null判断运算符

一般有这种需求，如果某个属性是`null`或者`undefined`，则通过`||`为它添加默认值。不过，如果数值的值为`''`(空字符串)或`0`或`false`，默认值也会生效。

为了避免以上情况，ES2020 引入了一个新的 Null 判断运算符`??`。

它的行为类似`||`，但是只有运算符左侧的值为`null`或`undefined`时，才会返回右侧的值。

```javascript
const animationDuration = response.settings?.animationDuration ?? 300;
```

`??`有一个运算优先级问题，它与`&&`和`||`的优先级孰高孰低。现在的规则是，如果多个逻辑运算符一起使用，必须用括号表明优先级，否则会报错。

```javascript
// 报错
lhs && middle ?? rhs
lhs ?? middle && rhs
lhs || middle ?? rhs
lhs ?? middle || rhs

// 正确
(lhs && middle) ?? rhs;
lhs && (middle ?? rhs);

(lhs ?? middle) && rhs;
lhs ?? (middle && rhs);

(lhs || middle) ?? rhs;
lhs || (middle ?? rhs);

(lhs ?? middle) || rhs;
lhs ?? (middle || rhs);
```