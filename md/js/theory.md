[有现成的总结非常好的文章，非常值得一看，下面的问题都可以在这里得到充分解答](https://www.jianshu.com/p/cd3fee40ef59)

**注意**： 上面文章中[前端基础进阶（十二）：深入核心，详解事件循环机制](https://www.jianshu.com/p/12b9f73c5a4f)对事件循环的解释中，对一次事件循环结束的节点不太正确，应该区分浏览器环境(chrome的webkit内核)和node(10版本,12版本)环境，v8早期版本和v8新版本

主要可以分为浏览器环境，node10环境和node12环境：

* 在低版本v8环境中，同源的task会在一轮事件循环中执行(setImmediate的优先级和setTimeout/setInterval的优先级不太好说，不确定)，然后才会去执行jobs，其优先级 process.nextTick >then>await
* 在高版本v8环境中，一轮事件循环中只执行**一个**task，之后会执行所有的jobs，jobs的优先级 process.nextTick>then=await
* node和浏览器的主要区别是：在node环境中, setTimeout和setInterval是同源的，setImmediate是单独的，而在浏览器环境中setTimeout和setInterval是非同源的

[setTimeout和setImmediate到底谁先执行](https://juejin.im/post/6844904100195205133)

Node.js的EventLoop是分阶段的

![avator](https://user-gold-cdn.xitu.io/2020/3/23/1710556d5509ef63)

1. timers: 执行setTimeout和setInterval的回调
2. pending callbacks: 执行延迟到下一个循环迭代的 I/O 回调
3. idle, prepare: 仅系统内部使用
4. poll: 检索新的 I/O 事件;执行与 I/O 相关的回调。事实上除了其他几个阶段处理的事情，其他几乎所有的异步都在这个阶段处理。
5. check: setImmediate在这里执行
6. close callbacks: 一些关闭的回调函数，如：socket.on('close', ...)

![avator](https://user-gold-cdn.xitu.io/2020/3/23/171055711f3b0aac)

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

#### 引用数据类型的赋值、深拷贝、浅拷贝
深拷贝，浅拷贝是针对引用数据类型。
引用数据类型的赋值，其实是复制的只是引用数据在堆内存中的指向。

浅拷贝指只复制对象的一层属性，而其子对象则没有复制

深拷贝是将原对象的各个属性逐个复制出去，而且将原对象各个属性所包含的对象也依次采用深复制的方法递归复制到新对象上。

如果被拷贝的对象只有一层，则深拷贝和浅拷贝无甚区别。

**浅拷贝**

针对数组使用过slice,concat,展开运算符都行

针对非数组对象
```javascript
let obj = {...};
let cloneObj = Object.assign({}, obj);
```

还有一个比较奇葩的方法：

`Object.freeze()` 冻结
freeze方法其效果在有一定程度与浅拷贝相同，但效果上还要比拷贝多上一层，即freeze冻结，但因为该方法自身 内部属性，该方法的名称又可以称为“浅冻结”，对于第一层数据，如浅拷贝一般，不可被新对象改变，但被freeze方法冻结过的对象，其自身也无法添加、删除或修改其第一层数据，但因为“浅冻结”这名称中浅的这一明显属性，freeze方法对于内部如果存在更深层的数据，是可以被自身修改，且也会被“=”号所引用给新的变量。


**深拷贝**

1. 方法一
    ```javascript
    let obj = {...};
    let cloneObj = JSON.parse(JSON.stringify(obj));
    ```
    注：
    **序列化**：把变量从内存中变成可存储或传输的过程称之为序列化
    **反序列化**：把变量内容从序列化的对象重新读到内存里称之为反序列化

    这种方法虽然可以实现数组或对象深拷贝,但不能处理函数,(函数会直接过滤掉)，而正则和Map,Set则会转化成空对象（{}）。

2. 方法二

    [clone函数](https://github.com/ConardLi/ConardLi.github.io/blob/master/demo/deepClone/src/clone_6.js)

    ```javascript
    function clone (target, map = new WeakMap()){
      if (!(target !== null && (typeof target === 'object' || typeof target === 'function'))) return target;
      const targetType = Object.prototype.toString.call(target).slice(8, -1).toLowerCase();
      let cloneTarget = undefined;
      const Ctor = target.constructor;

      if (map.get(target)) {
        return map.get(target);
      }
      map.set(target, cloneTarget);

      const cloneDeep = ['map', 'set', 'object', 'array', 'arguments'];
      if (cloneDeep.includes(targetType)) {
        cloneTarget = new Ctor();

        switch (targetType) {
          case 'set':
            target.forEach(e => {
              cloneTarget.add(clone(e, map));
            });
            break;
          case 'set':
            target.forEach((value, key) => {
              cloneTarget.set(key, clone(value, map));
            });
            break;
          case 'array':
          case 'arguments':
            target.forEach((e, i) => {
              cloneTarget[i] = clone(e, map);
            });
            break;
          case 'object':
            Object.keys(target).forEach(e => {
              cloneTarget[e] = clone(target[e], map);
            });
            break;
        }
        return cloneTarget;
      }else {
        switch (targetType) {
          case 'boolean':
          case 'string':
          case 'number':
          case 'error':
          case 'date':
            return new Ctor(target);
          case 'regexp':
            const reFlags = /\w*$/;
            const result = new Ctor(target.source, reFlags.exec(target));
            result.lastIndex = target.lastIndex;
            return result;
          case 'symbol':
            return Object(Symbol.prototype.valueOf.call(target));
          case 'function':
              const bodyReg = /(?<={)(.|\n)+(?=})/m;
              const paramReg = /(?<=\().+(?=\)\s+{)/;
              const funcString = target.toString();
              if (target.prototype) {
                  const param = paramReg.exec(funcString);
                  const body = bodyReg.exec(funcString);
                  if (body) {
                      if (param) {
                          const paramArr = param[0].split(',');
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
          default:
            return null;
        }
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

3. 方法三
函数库lodash中的_.cloneDeep用来做


#### 定时器的执行顺序或机制（牵扯到js的事件循环机制）

定时器其实是由浏览器当前页面标签进程中的定时器线程来管理的。

#### 作用域链

作用域链，是由当前环境和上层环境的一系列变量对象组成，它保证了当前执行环境对符合访问权限的变量和函数的有序访问。

#### 闭包（原理，使用，缺点）

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

#### valueOf 和 toString

#### instanceof 和 typeof 和 isPrototypeOf 和 Object.prototype.toString.call()

#### {}(字面量)、Object()、new Object() 创建对象有什么区别？ 还有Object.create()有什么区别

#### isPrototypeOf 和 setPrototypeOf