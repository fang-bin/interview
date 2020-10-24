[有现成的总结非常好的文章，非常值得一看，下面的问题都可以在这里得到充分解答](https://www.jianshu.com/p/cd3fee40ef59)

**注意**： 上面文章中[前端基础进阶（十二）：深入核心，详解事件循环机制](https://www.jianshu.com/p/12b9f73c5a4f)对事件循环的解释中，对一次事件循环结束的节点不太正确，应该区分浏览器环境(chrome的webkit内核)和node环境
1. 在node环境中，只有一类宏任务执行完之后，才会去执行所有微任务（setTimeout和setInterval是同源的）。
2. 在浏览器环境中，一个宏任务（setTimeout都属于一个任务，setInterval都属于另一个任务）结束之后，就会去所有微任务。

相关问题:
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
```javascript
const clone = (obj)=>{
  if(typeof obj !== 'object' || obj === null) return obj;
  const objType = Object.prototype.toString.call(obj).slice(8, -1);
  let temObj;
  if (objType === 'Array'){
    temObj = [];
  }else if (objType === 'Object'){
    temObj = {};
  }
  for (let item in obj) {
    if (obj[item] === obj) continue;
    if (typeof obj[item] === 'object'){
      temObj[item] = clone(obj[item]);
    }else{
      temObj[item] = obj[item];
    }
  }
  return temObj;
}
```
这种方法还是处理不了正则、Map、Set、和对象的原型链等

3. 方法三
函数库lodash中的_.cloneDeep用来做


#### 定时器的执行顺序或机制（牵扯到js的事件循环机制）

#### 闭包（原理，使用，优劣）

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

补充一道:

```javascript
function foo() {
  console.log( this.a );
}
var a = 2;
(function(){
  "use strict";
  foo(); //最后打印出来2
})();

```

[再来40道this面试题酸爽继续(1.2w字用手整理](https://juejin.im/post/6844904083707396109)

#### valueOf 和 toString