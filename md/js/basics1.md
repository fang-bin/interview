# 基础1

### 1. 如何在ES5环境下实现let

这个问题首先要看let的特性

1. let声明的变量只在它所在的代码块有效
2. 不存在变量提升
3. 暂时性死区（如果区块中存在let和const命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。“暂时性死区”也意味着typeof不再是一个百分之百安全的操作。）
4. 不允许在相同作用域内，重复声明同一个变量

ES5 只有全局作用域和函数作用域，没有块级作用域，let实际上为 JavaScript 新增了块级作用域。

主要实现的方式就是通过接近闭包的方式来实现

```javascript
{
  let a = 1;
  console.log(a);
}
console.log(a);
```

可以通过以下方式实现

```javascript
(function(){
  var a = 1;
  console.log(a);
})();
console.log(a);
```

当然这种方式有一些let特性并不能完全实现，比如说不存在变量提升等

**延伸一下**在es5环境下实现const的难点在于一旦声明，常亮的值就不能更改，可以通过Object.defineProperty来实现，不过这种方式实现的话，只能将声明的变量挂在对象下面，要么是全局对象挂在window下面，要么是自定义一个object来当容器

```javascript
function _const(key, value) {    
    const desc = {        
        value,        
        writable: false    
    }    
    Object.defineProperty(window, key, desc)
}
    
_const('obj', {a: 1})   //定义obj
obj.b = 2               //可以正常给obj的属性赋值
obj = {}                //抛出错误，提示对象read-only
```

### 2. call,apply,bind实现

首先先要知道call,apply,bind的作用:

* call()调用一个指定this值的函数, 分别地接受参数。语法: `func.call(thisArg, arg1, arg2, ...)`
* apply()调用一个指定this值的函数, 接收作为一个数组或者类数组对象提供的参数。语法: `func.apply(thisArg, [argsArray])`
* bind()方法创建一个指定this值的新函数，并在调用新函数时，将给定参数列表作为原函数的参数序列的前若干项。语法: `func.bind(thisArg, [arg1[, arg2[, ...]]])`


**Function.prototype.call**

```javascript
Function.prototype.myCall = function (thisArg, ...args) {
  if (typeof this !== 'function') {
    throw new TypeError('Call must be called on a function');
  }
  const fn = Symbol('fn');
  thisArg = thisArg || window;
  thisArg[fn] = this;
  const result = thisArg[fn](...args);
  delete thisArg[fn];
  return result;
}
```

**Function.prototype.apply**

```javascript
Function.prototype.myApply = function (thisArg, args) {
  if (typeof this !== 'function') {
    throw new TypeError('Apply must be called on a function');
  }
  const fn = Symbol('fn');
  thisArg = thisArg || window;
  thisArg[fn] = this;
  const result = thisArg[fn](...args);
  delete thisArg[fn];
  return result;
}
```

**Function.prototype.bind**

```javascript
Function.prototype.myBind = function(thisArg, ...args) {
  return () => {
    this.apply(thisArg, args)
  }
}
```

以上方法有如下问题：

1. bind()除了this还接收其他参数，bind()返回的函数也接收参数，这两部分的参数都要传给返回的函数
2. new的优先级：如果bind绑定后的函数被new了，那么此时this指向就发生改变。此时的this就是当前函数的实例
3. 没有保留原函数在原型链上的属性和方法

-正确方法

```javascript
Function.prototype.myBind = function (thisArg, ...args) {
  if (typeof this !== "function") {
    throw TypeError("Bind must be called on a function")
  }
  var self = this
  // new优先级
  var funcBind = function () {
    self.apply(this instanceof self ? this : thisArg, args.concat(Array.prototype.slice.call(arguments)))
  }
  // 继承原型上的属性和方法
  funcBind.prototype = Object.create(self.prototype);
  return funcBind;
}
```

### 3. new实现

```javascript
function mockNew() {
  let Constructor = Array.prototype.shift.call(arguments); // 取出构造函数  这个地方之后，arguments就已经去除了obj
  
  let obj = {}   // new 执行会创建一个新对象
  
  obj.__proto__ = Constructor.prototype 
  
  Constructor.apply(obj, arguments)
  return obj
}
```

### 4. 防抖与节流

* **防抖**：短时间内大量触发，只执行最后一次（延迟执行），多用于用户注册时候手机号和邮箱验证
* **节流**：短时间内只执行一次（间隔执行），多用于监听元素滚动时间

```javascript
// 防抖
function debounce (fn, time){
  let _timer = null;
  return function (...args) {
    const context = this;
    if (_timer){
      clearTimeout(_timer);
      _timer = null;
    }
    _timer = setTimeout(() => {
      fn.apply(context, args);
    }, time);
  }
}

// 节流
function throttle (fn, time){
  let _last_time = null;
  return function (...args){
    const _now_time = new Date();
    if (!_last_time || _now_time - _last_time > time){
      fn(...args);
      _last_time = _now_time;
    }
  }
}
```

### 5. 数组扁平化

###### Array.prototype.flat

###### 递归

```javascript
function flat(arr) {
  let result = [];
  for (const item of arr) {
    Object.prototype.toString.call(item).slice(8, -1) === 'Array' ? result = result.concat(flat(item)) : result.push(item);
  }
  return result;
}
```

###### reduce递归

```javascript
function flat(arr) {
  return arr.reduce((prev, cur) => {
    return prev.concat(Object.prototype.toString.call(cur).slice(8, -1) === 'Array' ? flat(cur) : cur)
  }, [])
}
```

###### 迭代加展开运算符

```javascript
function flat(arr) {
  while (arr.some(Array.isArray)) {
    arr = [].concat(...arr);
  }
  return arr;
}
```

### 手写一个Promise

##### Promise解决了什么问题

传统的异步编程中，如果异步之间存在依赖关系，就需要通过层层嵌套回调的方式满足这种依赖，如果嵌套层数过多，可读性和可以维护性都会变得很差，产生所谓的“回调地狱”，而 Promise 将嵌套调用改为链式调用，增加了可阅读性和可维护性。也就是说，Promise 解决的是异步编码风格的问题。

##### 业界比较著名实现的Promise的库

bluebird、Q、ES6-Promise

Promise/A+ 规范，业界所有 Promise 的类库都遵循这个规范。

Promise的实现就是一个发布订阅模式，这种收集依赖 -> 触发通知 -> 取出依赖执行的方式，被广泛运用于发布订阅模式的实现。

#### Promise的一些特别的特性

**then 的链式调用&值穿透特性**

在我们使用 Promise 的时候，当 then 函数中 return 了一个值，不管是什么值，我们都能在下一个 then 中获取到，这就是所谓的then 的链式调用。而且，当我们不在 then 中放入参数，例：promise.then().then()，那么其后面的 then 依旧可以得到之前 then 返回的值，这就是所谓的值的穿透。（.then 或者 .catch 的参数期望是函数，传入非函数则会发生值穿透。）

```javascript
Promise.resolve('foo')
  .then(Promise.resolve('bar'))
  .then(function(result){
    console.log(result)   //foo
  });
```
Promise方法链通过return传值，没有return就只是相互独立的任务而已

#### Promise的缺陷：Promies没有中断的方法

因为Promise 是没有中断方法的，xhr.abort()、ajax 有自己的中断方法，axios 是基于 ajax 实现的；fetch 基于 promise，所以他的请求是无法中断的。

不过可以使用race来封装Promise的中断方法

```javascript
function abortPromise (promise){
  let _abort = undefined;
  let _abort_promise = new Promise((resolve, reject) => {
    _abort = reject;
  });
  let p = Promise.race([promise, _abort_promise]);
  p.abort = _abort;
  return p;
}
```

#### promisify

promisify是node的utils模块中的一个函数，它作用就是为了转换最后一个参数是回调函数的函数为promise函数，且回调函数中有两个参数：error 和 data

使用:

```javascript
// 使用前
fs.readFile('./index.js', (err, data) => {
   if(!err) {
       console.log(data.toString())
   }
   console.log(err)
})
// 使用promisify后
const readFile = promisify(fs.readFile)
readFile('./index.js')
   .then(data => {
       console.log(data.toString())
   })
   .catch(err => {
       console.log('error:', err)
   })
```

promisify 函数实现
```javascript
// const newFn = promisify(fn)
// newFn(a) 会执行Promise参数方法
function promisify(fn) {
  return function(...args) {
    // 返回promise的实例
    return new Promise(function(reslove, reject) {
      // newFn(a) 时会执行到这里向下执行
      // 加入参数cb => newFn(a)
      args.push(function(err, data) {
        if (err) {
          reject(err)
        } else {
          reslove(data)
        }
      })
      // 这里才是函数真正执行的地方执行newFn(a, cb)
      fn.apply(null, args)
    })
  }
}
```

或者(其实下面和上面没啥两样，但是我认为这个更好理解)

```javascript
const promisify = (fn) => { // 典型的高阶函数 参数是函数 返回值是函数 
  return (...args)=>{
    return new Promise((resolve,reject)=>{
      fn(...args,function (err,data) { // node中的回调函数的参数 第一个永远是error
        if(err) return reject(err);
        resolve(data);
      })
    });
  }
}

//如果想要把 node 中所有的 api 都转换成 promise 的写法呢：
const promisifyAll = (target) =>{
  Reflect.ownKeys(target).forEach(key=>{
    if(typeof target[key] === 'function'){
      // 默认会将原有的方法 全部增加一个 Async 后缀 变成 promise 写法
      target[key+'Async'] = promisify(target[key]);
    }
  });
  return target;
}
```

**记录**: 我总是对Promise的一些特性有一些误解，以下为个人记录

1. Promise.reject一旦被catch捕获，就不会将接着传递，而catch后面的then则会一直传递
2. 内部error一旦被catch捕获，就不会向外层传递

[至此可以引申出Promise/Generator/Async实现原理](./writeCode.md)

### 手写Generator实现

执行 Generator 函数会返回一个遍历器对象，也就是说，Generator 函数除了状态机，还是一个遍历器对象生成函数。

调用 Generator 函数后，该函数并不执行，返回的也不是函数运行结果，而是一个指向内部状态的指针对象，也就是遍历器对象。

yield表达式后面的表达式，只有当调用next方法、内部指针指向该语句时才会执行，因此等于为 JavaScript 提供了手动的“惰性求值”（Lazy Evaluation）的语法功能。

由于next方法的参数表示上一个yield表达式的返回值，所以在第一次使用next方法时，传递参数是无效的。从语义上讲，第一个next方法用来启动遍历器对象，所以不用带有参数

```javascript
function* foo(x) {
  var y = 2 * (yield (x + 1));
  var z = yield (y / 3);
  return (x + y + z);
}

var a = foo(5);
a.next() // Object{value:6, done:false}
a.next() // Object{value:NaN, done:false}
a.next() // Object{value:NaN, done:true}

var b = foo(5);
b.next() // { value:6, done:false }
b.next(12) // { value:8, done:false }
b.next(13) // { value:42, done:true }
```

[至此可以引申出Promise/Generator/Async实现原理](./writeCode.md)

##### Thunk函数
Thunk 函数是自动执行 Generator 函数的一种方法。

起源: 函数的参数到底应该何时求值，一种是"传值调用"，一种是“传名调用”。

传值调用比较简单，但是对参数求值的时候，实际上还没用到这个参数，有可能造成性能损失。

编译器的“传名调用”实现，往往是将参数放到一个临时函数之中，再将这个临时函数传入函数体。这个临时函数就叫做 Thunk 函数。

JavaScript 语言是传值调用，它的 Thunk 函数含义有所不同。在 JavaScript 语言中，Thunk 函数替换的不是表达式，而是多参数函数，将其替换成一个只接受回调函数作为参数的单参数函数。

##### co模块
使用 co 的前提条件是，Generator 函数的yield命令后面，只能是 Thunk 函数或 Promise 对象。

例如`yield 1`这种情况就不行

下面还有简单的通过generator函数实现async函数就是通过让generator函数自执行来做的

[co模块](https://github.com/tj/co/blob/master/index.js)

### 手写Async实现

Async函数就是Generator函数的语法糖，Async函数可以理解成可以自执行的generator函数，同时它返回的是Promise对象，而Generator 函数返回的是Iterator对象。

[类似co模块通过不断调用自身，让Generator自执行实现Async函数效果](./writeCode.md)

### javascript绑定事件的方法都有哪些
1. on{事件名} 只能绑定一个事件，后面绑定的事件会覆盖前面绑定的事件
2. addEventListener 可以绑定多个事件，事件执行的顺序是先绑定先执行，可以设置第三个参数来确定事件流，false-->冒泡阶段（默认） true-->捕获阶段
3. attachEvent 是ie才有的方法，事件执行的顺序是后绑定先执行，默认是采用的冒泡的事件流，其监听的事件，之前必须加上on,例如 obj.attachEvent('onclick',fn);



### document.body 和 document.documentElement的区别

document.body：返回html dom中的body节点 即body标签元素

document.documentElement： 返回html dom中的root 节点 即html标签元素

两者主要的区别表现在获取 scrollTop 方面的差异(chrome亲测，其他没测试，可以通过`<!DOCTYPE >`与`<!DOCTYPE html>`来查看效果)：

* 当文档使用了DTD时， `document.body.scrollTop`的值为0,此时需要使用 `document.documentElement.scrollTop`来获取滚动条滚过的长度。
* 在未使用DTD定义文档时，使用`document.body.scrollTop`来获取值。

注：DTD(Document Type Definition，文档定义类型)的作用是定义XML文档的合法构建模块，可被成行的声明于XML文档中，也可作为一个外部引用。

兼容解决方案：

```javascript
var scrollTop = document.documentElement.scrollTop || document.body.scrollTop;
```

#### 判断一个变量是对象还是数组
```javascript
Object.prototype.toString.call(obj) === '[object Array]'  //true 数组
Object.prototype.toString.call(obj) === '[object Object]'  // true 对象

console.log(Object.prototype.toString.call("jerry"));//[object String]
console.log(Object.prototype.toString.call(12));//[object Number]
console.log(Object.prototype.toString.call(true));//[object Boolean]
console.log(Object.prototype.toString.call(undefined));//[object Undefined]
console.log(Object.prototype.toString.call(null));//[object Null]
console.log(Object.prototype.toString.call({name: "jerry"}));//[object Object]
console.log(Object.prototype.toString.call(function(){}));//[object Function]
console.log(Object.prototype.toString.call([]));//[object Array]
console.log(Object.prototype.toString.call(new Date));//[object Date]
console.log(Object.prototype.toString.call(/\d/));//[object RegExp]
function Person(){};
console.log(Object.prototype.toString.call(new Person));//[object Object]
```

**为什么不直接使用obj.toString()呢？**
同样是检测对象obj调用toString方法，obj.toString()的结果和Object.prototype.toString.call(obj)的结果不一样，这是为什么？

这是因为toString为Object的原型方法，而Array ，Function等类型作为Object的实例，都重写了toString方法。不同的对象类型调用toString方法时，根据原型链的知识，调用的是对应的重写之后的toString方法（function类型返回内容为函数体的字符串，Array类型返回元素组成的字符串.....），而不会去调用Object上原型toString方法（返回对象的具体类型），所以采用obj.toString()不能得到其对象类型，只能将obj转换为字符串类型；因此，在想要得到对象的具体类型时，应该调用Object上原型toString方法。

假如将数组的toString方法删除：

```javascript
var arr=[1,2,3];
console.log(Array.prototype.hasOwnProperty("toString"));//true
console.log(arr.toString());//1,2,3
delete Array.prototype.toString;//delete操作符可以删除实例属性
console.log(Array.prototype.hasOwnProperty("toString"));//false
console.log(arr.toString());//"[object Array]"
```

删除了Array的toString方法后，同样再采用arr.toString()方法调用时，不再有屏蔽Object原型方法的实例方法，因此沿着原型链，arr最后调用了Object的toString方法，返回了和Object.prototype.toString.call(arr)相同的结果。

#### 封装SDK(Software Development Kit 软件开发工具包)

**什么是sdk**
一些软件工程师为特定的软件包、软件框架、硬件平台、操作系统等建立应用软件时的开发工具的集合。一些软件工程师为特定的软件包、软件框架、硬件平台、操作系统等建立应用软件时的开发工具的集合。通常一个SDK包含一个或多个API，编程工具和文档。

**sdk必须具备原生的，短，速度快，干净，可读可测试特性。**

sdk要达到的目的
* 提供一个加载方案
* 暴露一个公共变量，最好能支持多种加载方式
* 提供未压缩版与压缩版
* 可以为不同的合作商提供定制版本
* 内部实现通过模块引用，方便扩展

每一次的SDK版本发布，确保它不仅适用于旧版本而且适应于未来的新版本。所以，记得为你的SDK写文档，代码要写注释，同时做好单元测试和用户场景测试。

* 引入SDK采用异步加载脚本的方式（优化网站的用户体验，所以不希望我们的SDK库阻塞其它主要进程，处于解决执行顺序的问题，可以让事件保存在某个对象中，然后SDK初始化的时候执行这个对象）,不过现在很多几乎都是写的基于webpack的sdk包（模块儿化加载），不用担心这些问题。
* 版本名尽量使用stable unstable alpha latest experimental 规范化，以免出现混乱
* 更新日志文件 新增，删除，更新，废弃，修正，安全
* 在SDK中只定义一个全局命名空间，并且不要用太过通用的名字，避免和其它类库名发生冲突。（现在webpack模块儿化也不太需要考虑这个问题）

#### 计算字符串中出现相同字母的最大数量和最大的字母

```javascript
function maxNum (str){
  if (str.length <= 1){
    return str;
  }
  let newArr = {};
  for (let i = 0; i < str.length; i++){
    if(newArr[str.charAt(i)]){
      newArr[str[i]] += 1;
    }else{
      newArr[str[i]] = 1;
    }
  }
  let maxNum = 0,
    maxCode = '';
  for (let key in newArr) {
    if (newArr[key] > maxNum){
      maxNum = newArr[key];
      maxCode = key;
    }else if (newArr[key] == maxNum){
      maxCode = [...maxCode, key];
    }
  }
  return maxCode;
}
```

#### JavaScript中 null 和 undefined的区别

在JavaScript中，undefined或null几乎没区别。

在最初JavaScript设计中，null是一个表示"无"的对象，转为数值时为0；undefined是一个表示"无"的原始值，转为数值时为NaN，但是，这种区分，在实践中很快就被证明不可行。

区别：

* null表示"没有对象"，即该处不应该有值。
  * null作为函数的参数，表示该函数的参数不是对象。
  * null作为对象原型链的终点。
* undefined表示"缺少值"，就是此处应该有一个值，但是还没有定义。
  * 变量被声明了，但没有赋值时，就等于undefined。
  * 调用函数时，应该提供的参数没有提供，该参数等于undefined。
  * 对象没有赋值的属性，该属性的值为undefined。
  * 函数没有返回值时，默认返回undefined。
* null转化为数值时为0，undefined转化为数值时为NaN

#### 很多地方用 `void 0` 代表 undefined 为什么

undefined 不是保留字（Reserved Word），它只是全局对象的一个属性，在低版本的IE浏览器(<=ie8)中会被重写，在局部作用域(包括chrome，不论使用var, const, let)中 undefined 也可以被重写。
void 运算符可以对给定的表达式求值，并且无论后面跟的是什么，都是返回 undefined，所以说不论是void 0 还是void 1都是可以的，更重要的是void不能被重写。

注意：undefined在现在主流的绝大部分浏览器的全局作用域上面都是不能更改了,但是在函数作用域或者块级作用域下面还是能被重写的,当然绝大部分人应该都不会去干这种傻事,但是还是用viod 0吧,这样可以以防万一,同时也更像一个老司机的代码啊。

#### 讲讲Map和Set

Map和Set是ES6提供的新的数据结构。

**Set**

* Set类似于数组，但是成员的值都是唯一的，没有重复的值。
* Set函数可以接受一个数组（或者具有 iterable 接口的其他数据结构）作为参数，用来初始化。Set本身是一个构造函数，用来生成 Set 数据结构。
* 向 Set 加入值的时候，不会发生类型转换，所以5和"5"是两个不同的值。Set 内部判断两个值是否不同，使用的算法叫做“Same-value-zero equality”，它类似于精确相等运算符（===），主要的区别是向 Set 加入值时认为NaN等于自身，而精确相等运算符认为NaN不等于自身。
* Set.prototype.size：返回Set实例的成员总数。
* Set.prototype.add(value)：添加某个值，返回 Set 结构本身。
* Set.prototype.delete(value)：删除某个值，返回一个布尔值，表示删除是否成功。
* Set.prototype.has(value)：返回一个布尔值，表示该值是否为Set的成员。
* Set.prototype.clear()：清除所有成员，没有返回值。
* Array.from方法可以将 Set 结构转为数组。
* Set.prototype.keys()：返回键名的遍历器
* Set.prototype.values()：返回键值的遍历器
* Set.prototype.entries()：返回键值对的遍历器
* Set.prototype.forEach()：使用回调函数遍历每个成员
* Set的遍历顺序就是插入顺序。

**Map**

* 类似于对象，也是键值对的集合，但是“键”的范围不限于字符串，各种类型的值（包括对象）都可以当作键。也就是说，Object 结构提供了“字符串—值”的对应，Map 结构提供了“值—值”的对应，是一种更完善的 Hash 结构实现。
* Map可以接受一个数组(任何具有 Iterator 接口、且每个成员都是一个双元素的数组的数据结构)作为参数。该数组的成员是一个个表示键值对的数组。
* Map.prototype.size： 返回 Map 结构的成员总数
* Map.prototype.set(key, value)： set方法设置键名key对应的键值为value，然后返回整个 Map 结构。如果key已经有值，则键值会被更新，否则就新生成该键。
* Map.prototype.get(key)： get方法读取key对应的键值，如果找不到key，返回undefined。
* Map.prototype.has(key)： has方法返回一个布尔值，表示某个键是否在当前 Map 对象之中。
* Map.prototype.delete(key)： delete方法删除某个键，返回true。如果删除失败，返回false。
* Map.prototype.clear()： clear方法清除所有成员，没有返回值。
* 只有对同一个对象的引用，Map 结构才将其视为同一个键,Map 的键实际上是跟内存地址绑定的，只要内存地址不一样，就视为两个键。
  ```javascript
  const map = new Map();
  map.set(['a'], 555);
  map.get(['a']) // undefined
  ```
* Map.prototype.keys()：返回键名的遍历器。
* Map.prototype.values()：返回键值的遍历器。
* Map.prototype.entries()：返回所有成员的遍历器。
* Map.prototype.forEach()：遍历 Map 的所有成员。
* Map 的遍历顺序就是插入顺序。

1. Map 转为数组最方便的方法，就是使用扩展运算符
    ```javascript
    const myMap = new Map()
      .set(true, 7)
      .set({foo: 3}, ['abc']);
    [...myMap] // [ [ true, 7 ], [ { foo: 3 }, [ 'abc' ] ] ]
    ```
2. 数组传入 Map 构造函数，就可以转为 Map。(`[[1,2], [3, 4]]`)
3. Map 转为对象
    ```javascript
    function strMapToObj(strMap) {
      let obj = Object.create(null);
      for (let [k,v] of strMap) {
        obj[k] = v;
      }
      return obj;
    }
    ```
    如果所有 Map 的键都是字符串，它可以无损地转为对象。
4. 对象转为 Map
    ```javascript
    let obj = {"a":1, "b":2};
    let map = new Map(Object.entries(obj));
    ```
5. Map 转为 JSON
    * Map 的键名都是字符串，这时可以选择转为对象 JSON。
    * Map 的键名有非字符串，这时可以选择转为数组 JSON。

6. JSON 转为 Map
    * JSON所有键名都是字符串，先转化为对象，然后对象转Map
    * 整个 JSON 就是一个数组，且每个数组成员本身，又是一个有两个成员的数组。先转化为数组，然后直接转Map

#### WeakSet和Set
WeakSet 结构与 Set 类似，也是不重复的值的集合。但是，它与 Set 有两个区别。
* WeakSet 的成员只能是对象（null除外），而不能是其他类型的值。
* WWeakSet 中的对象都是弱引用，即垃圾回收机制不考虑 WeakSet 对该对象的引用，也就是说，如果其他对象都不再引用该对象，那么垃圾回收机制会自动回收该对象所占用的内存，不考虑该对象还存在于 WeakSet 之中。
* WeakSet 的成员是不适合引用的，因为它会随时消失。另外，由于 WeakSet 内部有多少个成员，取决于垃圾回收机制有没有运行，运行前后很可能成员个数是不一样的，而垃圾回收机制何时运行是不可预测的，因此 ES6 规定 WeakSet 不可遍历。

**WeakSet用处**

1. 储存 DOM 节点，而不用担心这些节点从文档移除时，会引发内存泄漏。（存储的对象不会造成内存泄漏）

#### WeakMap和Map的区别，相比Object有什么优点

* WeakMap只接受对象作为键名（null除外），不接受其他类型的值作为键名。
* WeakMap键名所引用的对象都是弱引用，即垃圾回收机制不将该引用考虑在内。因此，只要所引用的对象的其他引用都被清除，垃圾回收机制就会释放该对象所占用的内存。也就是说，一旦不再需要，WeakMap 里面的键名对象和所对应的键值对会自动消失，不用手动删除引用。
* WeakMap既没有遍历操作（即没有keys()、values()和entries()方法），没有size属性，也无法清空，即不支持clear方法。WeakMap只有四个方法可用：get()、set()、has()、delete()。

WeakMap结构有助于防止内存泄漏。

**WeakMap应用**
1. WeakMap 应用的典型场合就是 DOM 节点作为键名。

    ```javascript
    let myWeakmap = new WeakMap();
    myWeakmap.set(
      document.getElementById('logo'),
      {timesClicked: 0})
    ;
    document.getElementById('logo').addEventListener('click', function() {
      let logoData = myWeakmap.get(document.getElementById('logo'));
      logoData.timesClicked++;
    }, false);
    ```
    上面代码中，document.getElementById('logo')是一个 DOM 节点，每当发生click事件，就更新一下状态。我们将这个状态作为键值放在 WeakMap 里，对应的键名就是这个节点对象。一旦这个 DOM 节点删除，该状态就会自动消失，不存在内存泄漏风险。

2. WeakMap 的另一个用处是部署私有属性
    ```javascript
    const _counter = new WeakMap();
    const _action = new WeakMap();
    class Countdown {
      constructor(counter, action) {
        _counter.set(this, counter);
        _action.set(this, action);
      }
      dec() {
        let counter = _counter.get(this);
        if (counter < 1) return;
        counter--;
        _counter.set(this, counter);
        if (counter === 0) {
          _action.get(this)();
        }
      }
    }
    const c = new Countdown(2, () => console.log('DONE'));
    c.dec();
    c.dec();
    ```
    上面代码中，Countdown类的两个内部属性_counter和_action，是实例的弱引用，所以如果删除实例，它们也就随之消失，不会造成内存泄漏。

**注**:
```javascript
const _map = new Map();
const _set = new Set();
const _weak_map = new WeakMap();
const _weak_set = new WeakSet();

typeof _map;  //object
typeof _set;  //object
typeof _weak_map;  //object
typeof _weak_set;  //object

Object.prototype.toString.call(_map);  //[object Map]
Object.prototype.toString.call(_set);  //[object Set]
Object.prototype.toString.call(_weak_map);  //[object WeakMap]
Object.prototype.toString.call(_weak_set);  //[object WeakSet]
```

#### 弱引用
弱引用是指垃圾回收的过程中不会将键名对该对象的引用考虑进去。

#### this指向
[这里就要详细了解js底层原理](./theory.md)

#### 深拷贝和浅拷贝
[这里也需要了解js底层原理](./theory.md)

#### javascript是按值传递还是按引用传递

javascript是值传递

ES中所有函数（方法）的参数都是按值传递的，即调用一个方法时，是将调用该方法时传入该方法的参数的值复制给函数内部的参数（将实参的值复制给形参）

**向参数传递基本类型的值**被传递的值会被复制给一个局部变量（这个局部变量就是形参，在ES中就是arguments对象的一个元素）

**向参数传递引用类型的值**JS会把被传递的值的地址复制给一个局部变量，因为复制的是地址，所以在函数执行时，函数形参在函数内部改变时会影响到函数外部的该引用变量的值，因为两个地址指向同一片内存区域，但在函数执行结束，函数内部的局部变量被销毁，影响即会消失。

#### 事件冒泡和事件捕获

![brower_event](https://github.com/fang-bin/interview/blob/master/image/brower_event.png)

```javascript
element.addEventListener(event, function, useCapture);

element.attachEvent('on' + event, function); //attachEvent只支持事件冒泡 
```
useCapture:
* true 事件在捕获阶段执行
* false 事件在冒泡阶段执行

**事件代理**
利用事件冒泡机制，可以给父元素中的多个子元素统一绑定事件。

使用事件代理的好处不仅在于将多个事件处理函数减为一个，而且对于不同的元素可以有不同的处理方法。假如上述列表元素当中添加了其他的元素节点（如：a、span等），我们不必再一次循环给每一个元素绑定事件，直接修改事件代理的事件处理函数即可。

`event.stopPropagation()` 阻止事件冒泡（它不是阻止事件本身）
`event.preventDefault()` 阻止默认事件

同时`return false`也可以阻止事件冒泡（不过它是直接阻止事件本身）。

#### 箭头函数和普通函数有什么区别，能不能作为构造函数？

* 函数体内的 this 对象，就是定义时所在的作用域中的 this 值，因为 JS 的静态作用域的机制，this 相当于一个普通变量会向作用域链中查询结果，同时定义时所在对象也并不等于所在对象中的 this 值。（可以理解成箭头函数中的this的值继承自外部作用域）
* 不可以使用 arguments 对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替。
* 不可以使用 yield 命令，因此箭头函数不能用作 Generator 函数。
* 箭头函数不能用作构造函数，和new一起使用会抛出错误（因为箭头函数没有自己的 this，无法调用 call，apply。同时没有 prototype 属性 ，而 new 命令在执行时需要将构造函数的 prototype 赋值给新的对象的 __proto__）

new的大致过程:

```javascript
function newFunc(father, ...rest) {
  var result = {};
  result.__proto__ = father.prototype;
  var result2 = father.apply(result, rest);
  if (
    (typeof result2 === 'object' || typeof result2 === 'function') &&
    result2 !== null
  ) {
    return result2;
  }
  return result;
}
```

**箭头函数的优点**
* 极简的写法
* 不需要绑定的this

**react中使用的箭头函数**
类的方法内部如果含有this，它默认指向类的实例。但是，一旦单独使用该方法，this会指向该方法运行时所在的环境（由于 class 内部是严格模式，所以 this 实际指向的是undefined），从而导致找不到该方法而报错。

解决的办法有三个:
* 在构造方法中绑定this
* 使用箭头函数 (箭头函数内部的this总是指向定义时所在的对象或者更准确的说是继承其定义位置的外部作用域)
* 使用Proxy，获取方法的时候，自动绑定this

在react中使用箭头函数坏处:

箭头函数在每次 render 时都会重新分配（和使用 bind 的方式相同），创建一个新的函数对象，是一个比较昂贵的操作。

避免在 render 中使用箭头函数和绑定。否则会打破 shouldComponentUpdate 和 PureComponent 的性能优化


#### requestAnimationFrame的理解

出现的原因: css仍存在无法实现的动画效果，js通过定时器实现动画的方法并不可靠，因为事件循环机制，延迟/间隔函数执行的时间并不精确，可能因为宏任务队列中的任务执行时间过长，而远远超过自身的延迟/间隔时间，而想要实现流畅的动画效果，刷新最好能达到60帧，也就是间隔时间为1000/60也就是16ms，很显然js定时器无法保证。

requestAnimationFrame应运而生，其作用就是让浏览器流畅的执行动画效果。可以将其理解为专门用来实现动画效果的api，通过这个api,可以告诉浏览器某个JavaScript代码要执行动画，浏览器收到通知后，则会运行这些代码的时候进行优化，实现流畅的效果，而不再需要开发人员烦心刷新频率的问题了。

```javascript
function animationWidth() {
  var div = document.getElementById('box');
  div.style.width = parseInt(div.style.width) + 1 + 'px';

  if(parseInt(div.style.width) < 200) {
    requestAnimationFrame(animationWidth)
  }
}
requestAnimationFrame(animationWidth);
```
requestAnimationFrame接受一个动画执行函数作为参数，这个函数的作用是仅执行一帧动画的渲染，并根据条件判断是否结束，如果动画没有结束，则继续调用requestAnimationFrame并将自身作为参数传入。这样就巧妙地避开了每一帧动画渲染的时间间隔问题。

动画在浏览器下次重绘之前调用指定的回调函数更新动画。（由于多数屏幕都是60Hz刷新率，浏览器也采用了16ms进行绘制节流，16ms毫秒内多次commit的DOM改动会合并为一次渲染）

requestAnimationFrame的兼容性问题主要是\<ie9，其他大多数都没有问题。


#### forEach for...in for...of的终止和跳过

* **`forEach`** 遍历数组的每一项，并对每一项执行一个 callback 函数。没有返回值(不论什么情况返回值都可以视为undefined)。`forEach` 方法没办法使用 `break` 语句跳出循环，或者使用 `continue` 跳过这次循环进入下次循环，但是可以通过 `return` 来实现跳过这次循环进入下次循环。
* **`for...of`** ES6提出的语句，在可迭代对象（`Array，Map，Set，String，TypedArray，arguments`）上创建一个迭代循环。（所有遍历器对象都可以遍历，即 `[Symbol.iterator]=collection` 都可以），不同于 `forEach` 可以使用 `break`, `continue`，但是不可以使用return;
* **`for...in`** `for...in` 语句以任意顺序遍历一个对象的可枚举属性的属性名。但是 `for...in` 会遍历对象本身的所有可枚举属性和从它原型继承而来的可枚举属性，因此如果想要仅迭代对象本身的属性，要结合`hasOwnProperty()` 来使用。 `for...in`遍历数组的情况下，可能会随机顺序遍历。(不可使用 `continue break return`)

**forEach如何实现break(跳出循环)**

1. 抛出错误

    抛出一个错误，但是需要注意的是要抛出一个可以与别的错误区别开的错误，这样不会干扰别的代码抛出的错误。这样做代码极为丑陋，也增加了维护的难度。
    ```javascript
    var BreakException = {};
    try {
      [1, 2, 3].forEach(function(v) {
        console.log(v); //只输出1,2
        if (v === 2) throw BreakException;
      });
    } catch (e) {
      if (e !== BreakException) throw e;
    }
    ```
2. 空跑循环（虽然后面的代码不再执行，但是循环并没有跳出）

    ```javascript
    [1, 2, 3].forEach(function(v) {
      if (this.breakFlag === true) {
        return false;
      }
      if (v === 2) {
        this.breakFlag = true
      }
      console.log(v) //只输出1,2
    }, {});
    ```
    这种方法不可避免的导致了不必要的运行

3. 太牛逼的一个方法，牛逼

    ```javascript
    var array = [1, 2, 3, 4, 5];
    array.forEach(function(item, index) {
        if (item === 2) {
            array = array.concat(array.splice(index, array.length - index));
            return;    //这里可以跳出当前的条件循环，要不然还会执行后面
        }
        console.log(item); //只输出1,2
    });
    ```

    引用数据类型存放在堆内存中，变量放在栈内存中的值只是一个指针，指向堆内存的数据。
    在终止循环条件中，先是原来的堆内存中的数据切掉后面的数据，然后通过concat浅复制之后重新赋值给原有的数据，这样循环条件中从堆内存读取到的原数据已经改变，而反应在变量之上的指针已经变了，但是数据由于是浅复制的，所有数组的数据并没有变。
    我自己解释一下，理解个大概意思。真牛逼。

