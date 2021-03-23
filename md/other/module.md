# 前端模块化

##### 什么是模块？
将一个复杂程序封装成几个块，块的内容数据私有，只向外部暴露一些借口进行通信。

##### 模块化的好处
* 避免命名冲突(减少命名空间污染)
* 更好的分离, 按需加载
* 更高复用性
* 高可维护性

### AMD

CommonJS 规范很好，但是不适用于浏览器环境，于是有了 AMD 和 CMD 两种方案。AMD 全称 Asynchronous Module Definition，即异步模块定义。它采用异步方式加载模块，模块的加载不影响它后面语句的运行。所有依赖这个模块的语句，都定义在一个回调函数中，等到加载完成之后，这个回调函数才会运行。除了和 CommonJS 同步加载方式不同之外，AMD 在模块的定义与引用上也有所不同。

`define(id?, dependencies?, factory);`

RequireJS的基本思想是，通过define方法，将代码定义为模块；通过require方法，实现代码的模块加载。

### CMD

CMD 是 Sea.js 所推广的一个模块化方案的输出。

在 CMD define 的入参中，虽然也支持包含 id, deps 以及 factory 三个参数的形式，但推荐的是接受 factory 一个入参，然后在入参执行时，填入三个参数 require、exports 和 module：

```javascript
define(function(require, exports, module) {
  var a = require('./a');
  a.doSomething();
  var b = require('./b'); 
  b.doSomething();
  ...
});
```

CMD规范专门用于浏览器端，模块的加载是异步的，模块使用时才会加载执行。CMD规范整合了CommonJS和AMD规范的特点。在 Sea.js 中，所有 JavaScript 模块都遵循 CMD模块定义规范。

##### AMD 和 CMD

* AMD 是 RequireJS 在推广过程中对模块定义的规范化产出。
* CMD 是 SeaJS 在推广过程中对模块定义的规范化产出。

### UMD

UMD，即通用模块规范。既然 CommonJs 和 AMD 风格一样流行，那么需要一个可以统一浏览器端以及非浏览器端的模块化方案的规范。

UMD的实现很简单：

* 先判断是否支持 AMD（define 是否存在），存在则使用 AMD 方式加载模块；
* 再判断是否支持 Node.js 模块格式（exports 是否存在），存在则使用 Node.js 模块格式；
* 前两个都不存在，则将模块公开到全局（window 或 global）；

### CommonJS

CommonJS 的一个模块，就是一个脚本文件。require命令第一次加载该脚本，就会执行整个脚本，然后在内存生成一个对象。

##### 特点

* 所有代码都运行在模块作用域（每个文件都是一个模块，都有自己的作用域），不会污染全局作用域。
* 模块可以多次加载，但是只会在第一次加载时运行一次，然后运行结果就被缓存了，以后再加载，就直接读取缓存结果。要想让模块再次运行，必须清除缓存。
* 模块加载的顺序，按照其在代码中出现的顺序，其模块加载也是同步的，只有加载完，才能执行后面的操作。

##### 用法
* 暴露模块：`module.exports = value` 或 `module.exports.xxx = value` 或 `exports.xxx = value`
（Node为每个模块提供一个exports变量，指向module.exports，相当于在头部写了`var exports = module.exports`，但是注意不能写 `exports = value`，因为这样相当于切断了 exports 和 module.exports 的联系）
* 引入模块：`require(xxx)`，如果是第三方模块，xxx 为模块名；如果是自定义模块，xxx 为模块文件路径

##### 模块加载机制
**CommonJS模块的加载机制是，输入的是被输出的值的拷贝。也就是说，一旦输出一个值，模块内部的变化就影响不到这个值。**

##### 补充例子:

###### 例一
```json
{
  id: '...',
  exports: { ... },
  loaded: true,
  ...
}
```

上面代码就是 Node 内部加载模块后生成的一个对象。该对象的id属性是模块名，exports属性是模块输出的各个接口，loaded属性是一个布尔值，表示该模块的脚本是否执行完毕。其他还有很多属性，这里都省略了。

以后需要用到这个模块的时候，就会到exports属性上面取值。即使再次执行require命令，也不会再次执行该模块，而是到缓存之中取值。也就是说，CommonJS 模块无论加载多少次，都只会在第一次加载时运行一次，以后再加载，就返回第一次运行的结果，除非手动清除系统缓存。

###### 例二
```javascript
//文件 a
var EventEmitter = require('events').EventEmitter;
module.exports = new EventEmitter();

setTimeout(function() {
  module.exports.emit('ready');
}, 1000);


//文件b
var a = require('./a');
a.on('ready', function() {
  console.log('module a is ready');
});
```
上面模块会在加载后1秒后，发出ready事件。文件b可以监听该事件。

###### 例三

```javascript
exports.hello = function() {
  return 'hello';
};
module.exports = 'Hello world';
```
上面代码中，hello函数是无法对外输出的，因为module.exports被重新赋值了。

如果模块单一输出一个值，只能使用 `module.exports`

### ES Module

ES6 模块的设计思想是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。CommonJS 和 AMD 模块，都只能在运行时确定这些东西。比如，CommonJS 模块就是对象，输入时必须查找对象属性。

**ES6模块的好处**
* 不再需要UMD模块格式了，将来服务器和浏览器都会支持 ES6 模块格式。
* 将来浏览器的新 API 就能用模块格式提供，不再必须做成全局变量或者navigator对象的属性。
* 不再需要对象作为命名空间（比如Math对象），未来这些功能可以通过模块提供。

ES6 的模块自动采用严格模式，不管你有没有在模块头部加上`"use strict";`。

**严格模式的限制**
* 变量必须声明后再使用
* 函数的参数不能有同名属性，否则报错
* 不能使用with语句
* 不能对只读属性赋值，否则报错
* 不能使用前缀 0 表示八进制数，否则报错
* 不能删除不可删除的属性，否则报错
* 不能删除变量delete prop，会报错，只能删除属性delete global[prop]
* eval不会在它的外层作用域引入变量
* eval和arguments不能被重新赋值
* arguments不会自动反映函数参数的变化
* 不能使用arguments.callee
* 不能使用arguments.caller
* 禁止this指向全局对象
* 不能使用fn.caller和fn.arguments获取函数调用的堆栈
* 增加了保留字（比如protected、static和interface）

#### CommonJS模块和ES6模块的差异
1. CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。
2. CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。

第一个差异CommonJS 模块输出的是值的拷贝，也就是说，一旦输出一个值，模块内部的变化就影响不到这个值。ES6 模块的运行机制与 CommonJS 不一样。JS 引擎对脚本静态分析的时候，遇到模块加载命令import，就会生成一个只读引用。等到脚本真正执行时，再根据这个只读引用，到被加载的那个模块里面去取值。

第二个差异是因为 CommonJS 加载的是一个对象（即module.exports属性），该对象只有在脚本运行完才会生成。而 ES6 模块不是对象，它的对外接口只是一种静态定义，在代码静态解析阶段就会生成。

在node中，ES6 模块与 CommonJS 模块尽量不要混用。

##### CommonJS模块的循环加载

CommonJS 模块的重要特性是加载时执行，即脚本代码在require的时候，就会全部执行。一旦出现某个模块被"循环加载"，就只输出已经执行的部分，还未执行的部分不会输出。

```javascript
// a.js
exports.done = false;
var b = require('./b.js');
console.log('在 a.js 之中，b.done = %j', b.done);
exports.done = true;
console.log('a.js 执行完毕');

// b.js
exports.done = false;
var a = require('./a.js');
console.log('在 b.js 之中，a.done = %j', a.done);
exports.done = true;
console.log('b.js 执行完毕');

// main.js
var a = require('./a.js');
var b = require('./b.js');
console.log('在 main.js 之中, a.done=%j, b.done=%j', a.done, b.done);

// $ node main.js

// 在 b.js 之中，a.done = false
// b.js 执行完毕
// 在 a.js 之中，b.done = true
// a.js 执行完毕
// 在 main.js 之中, a.done=true, b.done=true
```

##### ES6模块的循环加载

ES6 处理“循环加载”与 CommonJS 有本质的不同。ES6 模块是动态引用，如果使用import从一个模块加载变量（即import foo from 'foo'），那些变量不会被缓存，而是成为一个指向被加载模块的引用，需要开发者自己保证，真正取值的时候能够取到值。

```javascript
// a.mjs
import {bar} from './b';
console.log('a.mjs');
console.log(bar);
export let foo = 'foo';

// b.mjs
import {foo} from './a';
console.log('b.mjs');
console.log(foo);
export let bar = 'bar';

// $ node --experimental-modules a.mjs
// b.mjs
// ReferenceError: foo is not defined
```

首先，执行a.mjs以后，引擎发现它加载了b.mjs，因此会优先执行b.mjs，然后再执行a.mjs。接着，执行b.mjs的时候，已知它从a.mjs输入了foo接口，这时不会去执行a.mjs，而是认为这个接口已经存在了，继续往下执行。执行到第三行console.log(foo)的时候，才发现这个接口根本没定义，因此报错。

解决这个问题的方法，就是让b.mjs运行的时候，foo已经有定义了。这可以通过将foo写成函数来解决。

#### 总结:

CommonJS规范主要用于服务端编程，加载模块是同步的，这并不适合在浏览器环境，因为同步意味着阻塞加载，浏览器资源是异步加载的，因此有了AMD CMD解决方案。
AMD规范在浏览器环境中异步加载模块，而且可以并行加载多个模块。不过，AMD规范开发成本高，代码的阅读和书写比较困难，模块定义方式的语义不顺畅。
CMD规范与AMD规范很相似，都用于浏览器编程，依赖就近，延迟执行，可以很容易在Node.js中运行。不过，依赖SPM 打包，模块的加载逻辑偏重
ES6 在语言标准的层面上，实现了模块功能，而且实现得相当简单，完全可以取代 CommonJS 和 AMD 规范，成为浏览器和服务器通用的模块解决方案。

