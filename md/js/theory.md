[有现成的总结非常好的文章，非常值得一看，下面的问题都可以在这里得到充分解答](https://www.jianshu.com/p/996671d4dcc4)

**注意**： 上面文章中[前端基础进阶（十二）：深入核心，详解事件循环机制](https://www.jianshu.com/p/12b9f73c5a4f)对事件循环的解释中，对一次事件循环结束的节点不太正确，应该区分浏览器环境(chrome的webkit内核)和node(10版本,12版本)环境，v8早期版本和v8新版本

主要可以分为浏览器环境，node10环境和node12环境：

* 在低版本v8环境中，**同源的task**会在一轮事件循环中执行，然后才会去执行jobs，其优先级 process.nextTick >then>await
* 在高版本v8环境中，一轮事件循环中只执行**一个task(没有同源不同源，两个setTimeout也是分成两个task)**，之后会执行所有的jobs，jobs的优先级 process.nextTick>then=await。
* 在node环境中,setImmediate的执行时机，还要根据node事件循环流程来考虑
* node和浏览器的主要区别是：在node环境中, setTimeout和setInterval是同源的，setImmediate是单独的，而在浏览器环境中setTimeout和setInterval是非同源的(**在浏览器中高版本的v8环境中，是否是同源任务也不重要了，都是一个macro任务源为一次事件循环**)

[setTimeout和setImmediate到底谁先执行](https://juejin.im/post/6844904100195205133)

Node.js的EventLoop是分阶段的

![node事件循环](https://github.com/fang-bin/interview/blob/master/image/node-event-loop.jpg)

1. timers: 执行setTimeout和setInterval的回调
2. pending callbacks: 执行延迟到下一个循环迭代的 I/O 回调
3. idle, prepare: 仅系统内部使用
4. poll: 检索新的 I/O 事件;执行与 I/O 相关的回调。事实上除了其他几个阶段处理的事情，其他几乎所有的异步都在这个阶段处理。
5. check: setImmediate在这里执行
6. close callbacks: 一些关闭的回调函数，如：socket.on('close', ...)

![node事件循环流程](https://github.com/fang-bin/interview/blob/master/image/node-event-loop-flow.jpg)

例一:
```javascript
console.log('outer');

setTimeout(() => {
  setTimeout(() => {
    console.log('setTimeout');
  }, 0);
  setImmediate(() => {
    console.log('setImmediate');
  });
}, 0);


// outer
// setImmediate
// setTimeout
```

题解:

1. 外层是一个setTimeout，所以执行他的回调的时候已经在timers阶段了
2. 处理里面的setTimeout，因为本次循环的timers正在执行，所以他的回调其实加到了下个timers阶段
3. 处理里面的setImmediate，将它的回调加入check阶段的队列
4. 外层timers阶段执行完，进入pending callbacks，idle, prepare，poll，这几个队列都是空的，所以继续往下
5. 到了check阶段，发现了setImmediate的回调，拿出来执行
6. 然后是close callbacks，队列是空的，跳过
7. 又是timers阶段，执行我们的console


例二：
```javascript
console.log('outer');

setTimeout(() => {
  console.log('setTimeout');
}, 0);

setImmediate(() => {
  console.log('setImmediate');
});

//这次，outer之后，可能setTimeout在前，也可能setImmediate在前
```
题解:

注： node.js里面setTimeout(fn, 0)会被强制改为setTimeout(fn, 1),HTML 5里面setTimeout最小的时间限制是4ms).

1. 外层同步代码一次性全部执行完，遇到异步API就塞到对应的阶段
2. 遇到setTimeout，虽然设置的是0毫秒触发，但是被node.js强制改为1毫秒，塞入times阶段
3. 遇到setImmediate塞入check阶段
4. 同步代码执行完毕，进入Event Loop
5. 先进入times阶段，检查当前时间过去了1毫秒没有，如果过了1毫秒，满足setTimeout条件，执行回调，如果没过1毫秒，跳过
6. 跳过空的阶段，进入check阶段，执行setImmediate回调

关键就在这个1毫秒，如果同步代码执行时间较长，进入Event Loop的时候1毫秒已经过了，setTimeout执行，如果1毫秒还没到，就先执行了setImmediate。每次我们运行脚本时，机器状态可能不一样，导致运行时有1毫秒的差距，一会儿setTimeout先执行，一会儿setImmediate先执行。但是这种情况只会发生在还没进入timers阶段的时候。像我们第一个例子那样，因为已经在timers阶段，所以里面的setTimeout只能等下个循环了，所以setImmediate肯定先执行。

同理的还有其他poll阶段的API也是这样的，比如：

```javascript
var fs = require('fs')

fs.readFile(__filename, () => {
    setTimeout(() => {
        console.log('setTimeout');
    }, 0);
    setImmediate(() => {
        console.log('setImmediate');
    });
});
```

1. 我们代码基本都在readFile回调里面，他自己执行时，已经在poll阶段
2. 遇到setTimeout(fn, 0)，其实是setTimeout(fn, 1)，塞入后面的timers阶段
3. 遇到setImmediate，塞入后面的check阶段
4. 遇到nextTick，立马执行，输出'nextTick 1'
5. 到了check阶段，输出'setImmediate',又遇到个nextTick,立马输出'nextTick 2'
6. 到了下个timers阶段，输出'setTimeout'

### 引用数据类型的赋值、深拷贝、浅拷贝
深拷贝，浅拷贝是针对引用数据类型。
引用数据类型的赋值，其实是复制的只是引用数据在堆内存中的指向。

浅拷贝指只复制对象的一层属性，而其子对象则没有复制

深拷贝是将原对象的各个属性逐个复制出去，而且将原对象各个属性所包含的对象也依次采用深复制的方法递归复制到新对象上。

如果被拷贝的对象只有一层，则深拷贝和浅拷贝无甚区别。

##### 浅拷贝

针对数组使用过slice,concat,展开运算符都行

针对非数组对象
```javascript
let obj = {...};
let cloneObj = Object.assign({}, obj);
```

还有一个比较奇葩的方法：

`Object.freeze()` 冻结
freeze方法其效果在有一定程度与浅拷贝相同，但效果上还要比拷贝多上一层，即freeze冻结，但因为该方法自身 内部属性，该方法的名称又可以称为“浅冻结”，对于第一层数据，如浅拷贝一般，不可被新对象改变，但被freeze方法冻结过的对象，其自身也无法添加、删除或修改其第一层数据，但因为“浅冻结”这名称中浅的这一明显属性，freeze方法对于内部如果存在更深层的数据，是可以被自身修改，且也会被“=”号所引用给新的变量。


#### 深拷贝

##### 方法一
```javascript
let obj = {...};
let cloneObj = JSON.parse(JSON.stringify(obj));
```
注：
**序列化**：把变量从内存中变成可存储或传输的过程称之为序列化

**反序列化**：把变量内容从序列化的对象重新读到内存里称之为反序列化

**序列化过程中，不安全的值(undefined, Symbol, function, Map, Set, RegExp)不能识别，undefined、Symbol、function会变成null，Map、Set、RegExp则会变成{}**

**同时，`JSON.stringify` 只串行化自身的可枚举属性**

JSON.stringify在将JSON对象序列化为字符串时也使用了toString方法,但需要注意的是JSON.stringify并非严格意义上的强制类型转换，只是涉及toString的相关规则.(`JSON.stringify([1,2,3]); // "[1,2,3]"`)

如果对象定义了toJSON方法，会先调用此方法，然后用它的返回值来进行序列化。默认对象是没有此属性的，如果有需要可以手动添加。

`JSON.stringify(value[, replacer[, space]])`;
其中replacer可选，如果该参数是一个函数，则在序列化过程中，被序列化的值的每个属性都会经过该函数的转换和处理（类似map,不过其虚参顺序刚好和map相反，`replacer(key, value)）`；如果该参数是一个数组，则只有包含在这个数组中的属性名才会被序列化到最终的 JSON 字符串中；如果该参数为 null 或者未提供，则对象所有的属性都会被序列化。

space参数是缩进的字符，如果它是一个数字，就代表缩进多少空格符。如果是字符串，则固定以它为缩进符号。

深拷贝代码：

```javascript
var obj = {name:"Jack"}
obj.toJSON = function(){ return {name:”Join"} }

JSON.stringify(obj)   // “{“name”:”Join"}"
```

这种方法虽然可以实现数组或对象深拷贝,但不能处理函数,undefined,Symbol,(函数它们会直接过滤掉)，而正则和Map,Set则会转化成空对象（{}），而且也会丢失对象的原型链。

##### 方法二

```javascript
let obj1 = {a: 1};
Object.defineProperty(obj1, 'b', {
  configurable: true,
  enumerable: false,
  value: 2,
  writable: true,
});

let obj2 = Object.assign({}, obj1);
// 如果是 let obj2 = Object.assign(obj1)  则会拷贝obj1所有自身的属性，包括不可枚举属性
```

缺点：浅拷贝的缺点还有 **`Object.assign(target, ...source)`只拷贝自身的可枚举属性，会忽略`enumerable`为`false`的属性(仅限于source对象)**

##### 方法三

[clone函数](https://github.com/ConardLi/ConardLi.github.io/blob/master/demo/deepClone/src/clone_6.js)

```javascript
function clone (target, map = new WeakMap()){
  if (target === null || typeof target !== 'object') return target;
  const Ctor = target.constructor;
  let cloneTarget = undefined;
  const targetType = Object.prototype.toString.call(target).slice(8, -1).toLowerCase();
  const deepType = ['map', 'set', 'array', 'object', 'arguments'];
  if (deepType.includes(targetType)) cloneTarget = new Ctor();
  if (map.get(target)) return map.get(target);
  map.set(target, cloneTarget);

  switch (targetType) {
    case 'map':
      target.forEach((value, key) => {
        cloneTarget.set(key, clone(value, map));
      });
      return cloneTarget;
    case 'set':
      target.forEach(e => {
        cloneTarget.add(clone(e, map));
      })
      return cloneTarget;
    case 'object':
      Reflect.ownKeys(target).forEach(key => {
        cloneTarget[key] = clone(target[key], map);
      })
      return cloneTarget;
    case 'array':
    case 'arguments':
      target.forEach(e => {
        cloneTarget.push(clone(e, map));
      })
      return cloneTarget;
    case 'string':
    case 'number':
    case 'boolean':
    case 'date':
    case 'error':
      return new Ctor(target);
    case 'regexp':
      const reFlags = /\w*$/;
      const res = new Ctor(target.source, reFlags.exec(target));
      res.lastIndex = target.lastIndex;
      return res;
    case 'symbol':
    case 'bigint':
      return Object(Object.prototype.valueOf.call(target));
    default:
      return null;
  }
}
```
实际上克隆函数是没有实际应用场景的，两个对象使用一个在内存中处于同一个地址的函数也是没有任何问题的，lodash对函数的处理是：

```javascript
const isFunc = typeof value == 'function'
if (isFunc || !cloneableTags[tag]) {
        return object ? value : {}
}
```

**通过prototype来区分下箭头函数和普通函数，箭头函数是没有prototype的。**

**可以直接使用eval和函数字符串来重新生成一个箭头函数，注意这种方法是不适用于普通函数的。**

分别使用正则取出函数体和函数参数，然后使用new Function ([arg1[, arg2[, ...argN]],] functionBody)构造函数重新构造一个新的函数

```javascript
function cloneFunction(func) {
    const bodyReg = /(?<={)(.|\n)+(?=})/m;
    const paramReg = /(?<=\().+(?=\)\s+{)/;
    const funcString = func.toString();
    if (func.prototype) {
        console.log('普通函数');
        const param = paramReg.exec(funcString);
        const body = bodyReg.exec(funcString);
        if (body) {
            console.log('匹配到函数体：', body[0]);
            if (param) {
                const paramArr = param[0].split(',');
                console.log('匹配到参数：', paramArr);
                return new Function(...paramArr, body[0]);
            } else {
                return new Function(body[0]);
            }
        } else {
            return null;
        }
    } else {
        return eval(funcString);
    }
}
```

##### 方法四
函数库lodash中的_.cloneDeep用来做


### 定时器的执行顺序或机制（牵扯到js的事件循环机制）

定时器其实是由浏览器当前页面标签进程中的定时器线程来管理的。

### 作用域链

作用域链，是由当前环境和上层环境的一系列变量对象组成，它保证了当前执行环境对符合访问权限的变量和函数的有序访问。

### 闭包（原理，使用，缺点）

闭包是一种特殊对象，它由两部分组成，执行上下文和在该执行上下文中创建的函数，如果函数执行时，访问了那个执行上下文中变量对象的值，就会产生闭包。（和网上其他一些言论不太一样，chrome上也是这么认定的），一些书籍中都以函数名指代这里生成的闭包，而在chrome中，则以那个执行上下文的函数名指代闭包。

```javascript
(function () {
  var a = 10;
  var b = 20;

  var test = {
    m: 20,
    add: function (x) {
      return a + x;
    },
    sum: function () {
      return a + b + this.m;
    },
    mark: function (k, j) {
      return k + j;
    }
  }

  window.test = test;
})();

test.add(100);
test.sum();
test.mark();

var _mark = test.mark;
_mark();
```

![avator](https://upload-images.jianshu.io/upload_images/599584-77888095edb980a7.png)

mark执行时，闭包为外层的自执行函数，this指向test

![avator](https://upload-images.jianshu.io/upload_images/599584-fedeee99354936a9.png)

_mark执行时，闭包为外层的自执行函数，this指向window

**上面例子和下面例子的闭包可能出乎很多人的预料**

```javascript
function foo() {
  var a = 10;
  function fn1() {
    return a;
  }
  function fn2() {
    return 10;
  }
  fn2();
}
foo();
```
这个例子，和其他例子不太一样。虽然fn2并没有访问到foo的变量，但是foo执行时仍然变成了闭包。而当我将fn1的声明去掉时，闭包便不会出现了。

那么结合这个特殊的例子，我们可以这样这样定义闭包。

闭包是指这样的作用域(foo)，它包含有一个函数(fn1)，这个函数(fn1)可以调用被这个作用域所封闭的变量(a)、函数、或者闭包等内容。通常我们通过闭包所对应的函数来获得对闭包的访问。

##### 使用: 通过闭包，我们可以在其他的执行上下文中，访问到函数的内部变量。

1. 柯里化
2. 模块化

##### 缺点:

javascript有自动的垃圾回收机制，当一个值在内存中失去引用，垃圾回收机制很快会根据特殊的算法找到它，并将其回收，释放内存。而函数的执行上下文，在执行完毕之后，生命周期结束，那么该函数的执行上下文就会失去引用，其占用的内存空间很快会被垃圾回收器释放，可是闭包的存在，会阻止这一过程。

#### this指向

一些特殊的题型（主要是有关箭头函数）:

1. 构造函数对象中普通函数和箭头函数的区别：一层函数的题目
    ```javascript
    var name = 'window'
    function Person (name) {
      this.name = name
      this.foo1 = function () {
        console.log(this.name)
      }
      this.foo2 = () => {
        console.log(this.name)
      }
    }
    var person2 = {
      name: 'person2',
      foo2: () => {
        console.log(this.name)
      }
    }
    var person1 = new Person('person1')
    person1.foo1()
    person1.foo2()
    person2.foo2()
    ```

    题解：
    * person1.foo1()是个普通函数，this由最后调用它的对象决定，即person1。
    * person1.foo2()为箭头函数，this由外层作用域决定，且指向函数定义时的this而非执行时，在这里它的外层作用域是函数Person，且这个是构造函数，并且使用了new来生成了对象person1，所以此时this的指向是为person1。
    * person2.foo2()字面量创建的的对象person2中的foo2是个箭头函数，由于person2是直接在window下创建的，你可以理解为它所在的作用域就是在window下，因此person2.foo2()内的this应该是window。

    答案:
    ```html
    'person1'
    'person1'
    'window'
    ```

箭头函数的this指向:

* 它里面的this是由外层作用域来决定的，且指向函数定义时的this而非执行时
* 字面量创建的对象，作用域是window，如果里面有箭头函数属性的话，this指向的是window
* 构造函数创建的对象，作用域是可以理解为是这个构造函数，且这个构造函数的this是指向新建的对象的，因此this指向这个对象。
* 箭头函数的this是无法通过bind、call、apply来直接修改，但是可以通过改变作用域中this的指向来间接修改。

箭头函数的优点：

* 箭头函数写代码拥有更加简洁的语法(当然也有人认为这是缺点)
* this由外层作用域决定，所以在某些场合我们不需要写类似const that = this这样的代码

**箭头函数还有一个特性，其没有arguments对象，同时箭头函数不能用来当构造函数**

##### 避免使用箭头的场景

1. 使用箭头函数定义对象的方法
    ```javascript
    let obj = {
        value: 'LinDaiDai',
        getValue: () => console.log(this.value)
    }
    obj.getValue() // undefined
    ```
2. 定义原型方法
    ```javascript
    function Foo (value) {
        this.value = value
    }
    Foo.prototype.getValue = () => console.log(this.value)

    const foo1 = new Foo(1)
    foo1.getValue() // undefined
    ```
3. 不能作为构造函数，会直接报TypeError
4. 作为某些事件的回调函数
    ```javascript
    const button = document.getElementById('myButton');
    button.addEventListener('click', () => {
        console.log(this === window); // => true
        this.innerHTML = 'Clicked button';
    });
    ```

##### 构造函数与原型方法上的this

通过new操作符调用构造函数，会经历以下4个阶段:

* 创建一个新的对象；
* 将构造函数的this指向这个新对象；
* 指向构造函数的代码，为这个对象添加属性，方法等；
* 返回新对象。

补充一道:

```javascript
function foo() {
  console.log( this.a );
}
var a = 2;
(function(){
  "use strict";
  foo(); //最后打印出来2，这个是作用域链的问题
})();
```

[再来40道this面试题酸爽继续(1.2w字用手整理](https://juejin.im/post/6844904083707396109)

### valueOf 和 toString 和 [Symbol.toPrimitive]

**对象转原始类型过程**

1. 如果有\[Symbol.toPrimitive]方法，优先调用再返回
2. 调用valueOf()，如果转换为原始类型，则返回
3. 调用toString()，如果转换为原始类型，则返回
4. 如果都没有返回原始类型，会报错

valueOf: 返回对象的原始值表示（可以对一些封装过的基本类型进行拆封操作）
toString: 返回对象的字符串表示

valueOf和toString都是Object.prototype的方法，不过很多内置对象都会重写这两个方法，以适应实际需要。

这里主要讨论对象在参与运算时候，调用这两个方法的情况：

##### Object -> Boolean (`Boolean({})`)

直接转换为true（包装类型也一样），不调用valueOf和toString;

`Boolean(new Boolean(false))` === `true`

##### Object -> Number （`Number({})`）

1. 如果对象具有 valueOf 方法且返回原始值(string、number、boolean、undefined、null),则将该原始值转换为数字（转换失败则会返回NaN），并返回这个数字
2. 如果对象具有toString方法且返回原始值(string、number、boolean、undefined、null)，则将该原始值转换为数字(转换失败会返回NaN)，并返回这个数字
3. 如果toString方法返还的还是对象，抛出TypeError

##### Object -> String (`String({})`)

1. 如果对象具有toString方法且返回原始值(string、number、boolean、undefined、null)，则将该原始值转换为字符串，并返回该字符串
2. 如果对象具有valueOf方法且返回原始值(string、number、boolean、undefined、null)，则将该原始值转换为字符串，并返回该字符串
3. 如果valueOf方法返回的是对象，抛出TypeError

toString转换规则
| 对象 |  toString 返回值 |
| :---: | :---: |
| Array | 以逗号分割的字符串，如[1,2]的toString返回值为"1,2" |
| Boolean | "true" |
| Date | 可读的时间字符串，如"Tue Oct 15 2019 12:20:56 GMT+0800 (中国标准时间)" |
| Function | 声明函数的JS源代码字符串 |
| Number | "数字值" |
| Object | "[object Object]" | 
| String | "字符串" |

#### 这里补充一下js中有关运算的一些隐性转换

1. 如果有一个是对象，则遵循对象对原始值的转换过程(Date对象直接调用toString完成转换，其他对象通过valueOf转化，如果转换不成功则调用toString)
2. 如果两个都是对象，两个对象都遵循步骤1转换到字符串
3. 两个数字，进行算数运算
4. 两个字符串，直接拼接
5. 一个字符串一个数字，直接拼接为字符串

相关面试题:

```javascript
var a = {};
var b = {};
var c = {};
c[a] = 1;
c[b] = 2;

console.log(c[a]);
console.log(c[b]);
```

由于对象的key是字符串，所以c[a]和c[b]中的a和b会执行[对象到字符串]的转换。

根据转换规则, a和b都转换为了[object Object]，所以c[a]和c[b]操作的是同一个键。

答案是输出两个2，c对象的最终结构如下：

```javascript
{
  '[object Object]':2
}
```

### instanceof 和 typeof 和 isPrototypeOf 和 Object.prototype.toString.call()

##### typeof运算符可以返回一个值的数据类型。

```javascript
typeof ''; //string;
typeof 1; //number;
typeof false; //boolean;
typeof Symbol('fangbin'); //symbol;
typeof undefined;   //undefined;

typeof null;  //object;
typeof {};  //object;
typeof (() => {});   //function
typeof (new Date());  //object;
typeof Date.now();   //number;
typeof /abcd/;   //object;
typeof arguments;  //函数里的arguments   object;
typeof [];  //object;
typeof new Set();  //object;
typeof new Map(); //object;
```

typeof 在ES5中是一个安全操作，但是在ES6中，由于let/const的暂时性死区的问题，typeof也不再安全。

typeof 只能大致区分类型，尤其面对复杂类型的时候，更是无法区分。

##### instanceof运算符返回一个布尔值，表示对象是否为某个构造函数的实例。

会检查右边构造函数的原型对象（prototype），是否在左边对象的原型链上。

instanceof的原理是检查右边构造函数的prototype属性，是否在左边对象的原型链上。有一种特殊情况，就是左边对象的原型链上，只有null对象。这时，instanceof判断会失真。

**instanceof运算符只能用于对象（纯对象和数组），不适用原始类型（Undefined、Null、Boolean、Number 和 String）的值。**

利用instanceof运算符，还可以巧妙地解决，调用构造函数时，忘了加new命令的问题。

```javascript
function Fubar (foo, bar) {
  if (this instanceof Fubar) {
    this._foo = foo;
    this._bar = bar;
  } else {
    return new Fubar(foo, bar);
  }
}
```

##### isPrototypeOf

用于测试一个对象是否存在于另一个对象的原型链上。

`prototypeObj.isPrototypeOf(object)`

```javascript
var v = new Vehicle();
v instanceof Vehicle // true

v instanceof Vehicle
// 等同于
Vehicle.prototype.isPrototypeOf(v)
```

##### Object.prototype.toString.call()

要想区别对象、数组、函数单纯使用 typeof 是不行的。null 和Array 的结果也是 object，有时候我们需要的是 "纯粹" 的 object 对象。

我们可以通过Object.prototype.toString方法准确判断某个对象值属于哪种内置类型。

在JavaScript中所有类型(如：对象，数组，函数)都包含一个内部属性 \[[calss]] ，此属性可以看作是一个内部分类。它并不是传统面向对象上的类,由于是内部属性，所以我们无法直接访问,不过，可以转换为字符串来查看.

```javascript
Object.prototype.toString.call([1,2,3]) // '[Object Array]'
Object.prototype.toString.call(/^[1,2]$/)  // '[Object RegExp]'
```

每个不同的类型中的 \[[class]] ，都对应着它们相应的内部构造函数，也就是对象的内部 \[[class]] 属性和创建该对象的内建原生构造函数相对应，但有些特例.

```javascript
//一说特例，我估计就有人想到javascript中比较蛋疼的两个类型
Object.prototype.toString.call(null) // '[Object Null]'
Object.prototype.toString.call(undefined) // '[Object Undefined]'
```

**除了null和undefined，其他都有各自的构造类，都是javascript的内置构造函数。**

#### {}(字面量)、Object()、new Object() 创建对象有什么区别？ 还有Object.create()有什么区别

##### 封装对象

在日常开发中，我们通常不直接使用内置的构造类，而是直接通过常量访问。

通过构造函数实例出来的常量变成了对象，其实就是手动创建其封装对象，封装对象上存在对应的数据类型方法。

**我们在使用常量的方式（.的方式或者[’属性名‘]这类方式）直接访问属性和方法时，javascript会自动为你包装一个封装对象,相当于上面我们手动包装。在操作属性或方法完成之后JavaScript也会释放当前封装对象**

补充:

说到这里，我们可能会想到一个问题，如果需要经常用到这些字符串的属性和方法，比如在for循环当中使用`i < a.length`,那么一开始创建一个封装对象也许更为方便，这样JavaScript引擎就不用每次都自动创建和自动释放循环执行这些操作了。

其实我们的想法很好，但实际证明这并不是一个好办法，因为**浏览器已经为.length这样常见情况做了性能优化，直接使用封装对象来提前优化代码反而会降低执行效率。**

一般情况下，我们不需要直接使用封装对象，最好是让JavaScript引擎自动选择什么时候应该使用封装对象。

```javascript
var test = 'abc';
test.len = 123;
var t = test.len;
```

此处t为undefined，第三行是通过新的原始对象访问其.len属性，这并不是上次添加的.len，上次的已经被销毁，当前是一个新的封装对象.

例子:

```javascript
var s = 'hello world';
var world = s.toUpperCase();
```

首先 javascript 会将字符串值通过 new String(s) 的方式转换为封装对象，这个对象继承了来自字符串构造函数的所有方法(这些操作都从第二行访问方法时开始发生),当前 s 已经变成了一个封装对象，接下来在封装对象中查找需要的方法或属性，找到了之后做出相应的操作。**一旦引用结束，这个新创建的对象就会销毁**。这时候 s.toUpperCase 已经运行了该方法，随即销毁封装对象。

###### 拆封

封装对象中的基本类型值，我们可以使用valueOf方法拆封获取。

```javascript
var ss = new String("123");
ss.valueOf()  //"123"
```

##### Object() 强制类型转化

Object强制类型转换相当于我们对原有类型进行手动对象包装。

##### 直接字面量创建 (`{} '' [] 1`)
相当于new关键字创建的一种语法糖。

##### new关键字创建 (`new String() new Boolean() new Object()`)
相当于直接创建好一个封装对象， 和字面量创建的方式并没有什么区别，当然字面量更高效一些（js已经为字面量对象做了性能优化）

```javascript
let a = new Object();
let b = {}
a.__proto__ === b.__proto__;  //true
b.__proto__ === Object.prototype;  //true
a.__proto__ === Object.prototype;  //true

let x = '';
let y = new String('');
x.__proto__ === y.__proto__;  //true
x.__proto__ === String.prototype;  //true
```

##### Object.create()

`Object.create()`方法创建一个新对象，使用现有的对象来提供新创建对象的`__proto__`

`Object.create(proto[, propertiesObject])`

* proto 必填参数，是新对象的原型对象，如果这个参数是Null，那新对象就彻彻底底是个空对象，没有继承Object.prototype上的任何属性和方法，如`hasOwnProperty()、toString()`等。
* propertiesObject 是可选参数，指定要添加到新对象上的可枚举属性（即其自定义的属性和方法，可用`hasOwnProperty()`获取的，而不是原型对象上的）的描述符及相应的属性名称。

```javascript
var bb = Object.create(null, {
    a: {
        value: 2,
        writable: true,
        configurable: true
    }
});
console.dir(bb); // {a: 2}
console.log(bb.__proto__); // undefined
console.log(bb.__proto__ === Object.prototype); // false
console.log(bb instanceof Object); // false 没有继承`Object.prototype`上的任何属性和方法，所以原型链上不会出现Object


var cc = Object.create({b: 1}, {
    a: {
        value: 3,
        writable: true,
        configurable: true
    }
});
console.log(cc); // {a: 3}
console.log(cc.hasOwnProperty('a'), cc.hasOwnProperty('b')); // true false 说明第二个参数设置的是新对象自身可枚举的属性
console.log(cc.__proto__); // {b: 1} 新对象cc的__proto__指向{b: 1}
console.log(cc.__proto__ === Object.protorype); // false
console.log(cc instanceof Object); // true cc是对象，原型链上肯定会出现Object
```

##### 总结
* 字面量和new关键字创建的对象是Object的实例，原型指向Object.prototype，继承内置对象Object
* Object.create(arg, pro)创建的对象的原型取决于arg，arg为null，新对象是空对象，没有原型，不继承任何对象；arg为指定对象，新对象的原型指向指定对象，继承指定对象

#### Object.create() 和 Object.setPrototypeOf()区别

`Object.setPrototypeOf(obj, prototype)`  

obj为要设置其原型的对象  

prototype为该对象的新原型(一个对象 或 null)

Object.setPrototypeOf()方法设置一个指定的对象的原型。

两者都能达到设置对象原型的功能，但是具体表现上有一些区别。

例子: 假设有Animal和Plants俩个函数用于生成对象，并在原型上具备一些方法。 然后我们让Animal的原型指向Plants
```javascript
// 初始代码
function Animal (name,sound) {
        this.name = name
        this.sound = sound
    }
Animal.prototype.shout = function () {
    console.log(this.name + this.sound)
}
let dog = new Animal('pipi','wang!wang!')

// 定义Plants
function Plants (name) {
    this.name = name
    this.sound = null
}

// 函数接收参数用于区分
Plants.prototype.shout = function (xssss) {
    console.log(this.name + this.sound +'plants tag')
}
Plants.prototype.genO2 = function () {
    console.log(this.name + '生成氧气。')
}
```

```javascript
// 使用Object.create
Animal.prototype = Object.create(Plants.prototype)
console.log(Animal.prototype)
/*
Plants {}
    __proto__:
        shout: ƒ (xssss)
        genO2: ƒ ()
        constructor: ƒ Plants()
        __proto__: Object
*/
let cat = new Animal('mimi','miao~miao~')

dog.shout() // pipi wang!wang!
cat.shout() // mimi miao~miao~ plants tag
cat.genO2() // mimi 生成氧气。
```

```javascript
Object.setPrototypeOf(Animal.prototype,Plants.prototype)
console.log(Animal.prototype)
/*
Plants {shout: ƒ, constructor: ƒ}
    shout: ƒ (xssss)
    constructor: ƒ Animal(name,sound)
    __proto__:
    shout: ƒ ()
    genO2: ƒ ()
    constructor: ƒ Plants()
    __proto__: Object
*/
dog.shout() // pipi wang!wang!
cat.shout() // mimi miao~miao~
cat.genO2() // mimi 生成氧气。
```

总结:
使用Object.create,Animal.prototype将会指向一个空对象，空对象的原型属性指向Plants的prototytpe。所以我们不能再访问Animal的原有prototypoe中的属性。Object.create的使用方式也凸显了直接重新赋值

使用Object.setPrototypeOf则会将Animal.prototype将会指向Animal原有的prototype，然后这个prototype的prototype再指向Plants的prototytpe。所以我们优先访问的Animal，然后再是plants。

在进行俩个原型之间的委托时使用setPrototype更好，Object.create更适和直接对一个无原生原型的对象快速进行委托。

但是，由于现代js引擎优化属性访问所带来的特性的关系，更改对象的原型在各个浏览器和js引擎上都是一个很慢的操作。其在更改继承的性能上的影响是微妙而又广泛的，这不仅仅限于`obj.__proto__ = ...`语句上的时间花费，而且可能会延伸到任何代码，那些可以访问任何原型已被更改的对象的代码。如果关心性能，应该避免设置一个对象的原型，相反，应该使用`Object.create()`来创建带有想要原型的新对象。

[Object.setPrototypeOf()和Object.create()的区别](https://juejin.im/post/6844903527941144589)

##### Object.getPrototypeOf() 
可以通过该方法获取对象的原型

`__proto__` 并不是语言本身的特性，这是各大厂商具体实现时添加的私有属性，虽然目前很多现代浏览器的 JS 引擎中都提供了这个私有属性，但依旧不建议在生产中使用该属性，避免对环境产生依赖。生产环境中，我们可以使用 Object.getPrototypeOf 方法来获取实例对象的原型，然后再来为原型添加方法/属性。
