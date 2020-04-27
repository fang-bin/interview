[有现成的总结非常好的文章，非常值得一看](https://www.jianshu.com/p/cd3fee40ef59)

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

#### valueOf 和 toString