### 1. 如何在ES5环境下实现let

这个问题首先要看let的特性

1. let声明的变量只在它所在的代码块有效
2. 不存在变量提升
3. 暂时性死区（如果区块中存在let和const命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。“暂时性死区”也意味着typeof不再是一个百分之百安全的操作。）
4. 不允许在相同作用域内，重复声明同一个变量

ES5 只有全局作用域和函数作用域，没有块级作用域，let实际上为 JavaScript 新增了块级作用域。

主要实现的方式就是通过接近闭包的方式来实现

    {
      let a = 1;
      console.log(a);
    }
    console.log(a);

可以通过以下方式实现

    (function(){
      var a = 1;
      console.log(a);
    })();
    console.log(a);

当然这种方式有一些let特性并不能完全实现，比如说不存在变量提升等

**延伸一下**在es5环境下实现const的难点在于一旦声明，常亮的值就不能更改，可以通过Object.defineProperty来实现，不过这种方式实现的话，只能将声明的变量挂在对象下面，要么是全局对象挂在window下面，要么是自定义一个object来当容器

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

### 2. call,apply,bind实现

首先先要知道call,apply,bind的作用:

* call()调用一个指定this值的函数, 分别地接受参数。语法: `func.call(thisArg, arg1, arg2, ...)`
* apply()调用一个指定this值的函数, 接收作为一个数组或者类数组对象提供的参数。语法: `func.apply(thisArg, [argsArray])`
* bind()方法创建一个指定this值的新函数，并在调用新函数时，将给定参数列表作为原函数的参数序列的前若干项。语法: `func.bind(thisArg, [arg1[, arg2[, ...]]])`


**Function.prototype.call**

    Function.prototype.myCall = function (thisArg, ...args) {
      if (typeof this !== 'undefined') {
        throw new TypeError('Call must be called on a function');
      }
      const fn = Symbol('fn');
      thisArg = thisArg || window;
      thisArg[fn] = this;
      const result = thisArg[fn](...args);
      delete thisArg[fn];
      return result;
    }

**Function.prototype.apply**

    Function.prototype.myApply = function (thisArg, args) {
      if (typeof this !== 'undefined') {
        throw new TypeError('Apply must be called on a function');
      }
      const fn = Symbol('fn');
      thisArg = thisArg || window;
      thisArg[fn] = this;
      const result = thisArg[fn](...args);
      delete thisArg[fn];
      return result;
    }

**Function.prototype.bind**

    Function.prototype.myBind = function(thisArg, ...args) {
      return () => {
        this.apply(thisArg, args)
      }
    }

以上方法有如下问题：

1. bind()除了this还接收其他参数，bind()返回的函数也接收参数，这两部分的参数都要传给返回的函数
2. new的优先级：如果bind绑定后的函数被new了，那么此时this指向就发生改变。此时的this就是当前函数的实例
3. 没有保留原函数在原型链上的属性和方法

-正确方法

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

### 3. new实现

    function mockNew() {
        let Constructor = Array.prototype.shift.call(arguments); // 取出构造函数  这个地方之后，arguments就已经去除了obj
        
        let obj = {}   // new 执行会创建一个新对象
        
        obj.__proto__ = Constructor.prototype 
        
        Constructor.apply(obj, arguments)
        return obj
    }

### 4. 防抖与节流

* 防抖：短时间内大量触发，只执行最后一次（延迟执行）
* 节流：短时间内只执行一次（间隔执行）

### 5. 数组扁平化

###### Array.prototype.flat

###### 递归

    function flat(arr) {
        let result = [];
        for (const item of arr) {
            Object.prototype.toString.call(item).slice(8, -1) === 'Array' ? result = result.concat(flat(item)) : result.push(item);
        }
        return result;
    }

###### reduce递归

    function flat(arr) {
      return arr.reduce((prev, cur) => {
        return prev.concat(Object.prototype.toString.call(cur).slice(8, -1) === 'Array' ? flat(cur) : cur)
      }, [])
    }

###### 迭代加展开运算符

    function flat(arr) {
      while (arr.some(Array.isArray)) {
        arr = [].concat(...arr);
      }
    }

### 手写一个Promise

我总是对Promise的一些特性有一些误解，以下为个人记录

1. Promise.reject一旦被catch捕获，就不会将接着传递，而catch后面的then则会一直传递
2. 内部error一旦被catch捕获，就不会向外层传递

[至此可以引申出Promise/Generator/Async实现原理](./writeCode.md)



