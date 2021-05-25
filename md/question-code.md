1. 手写jsonp
2. Promise 带中断实现 和失败重发实现
3. curry
  封装一个curry函数，在实现一个以下效果的函数
    ```javascript
    console.log(add(1)(2)(3)); //6
    console.log(add(1,2,3)(4)); //10
    console.log(add(1)(2)(3)(4)(5)); //15
    ```

4. call apply bind new Object.create Object.assign实现
5. 防抖 节流
6. promisify实现
7. co模块简易实现
8. forEach中止
9. 深拷贝
10. 利用Generator函数实现斐波那契数列
11. 利用Generator函数遍历完全二叉树
12. 冒泡排序 选择排序 插入排序 快速排序 归并排序
13. 写一个 mySetInterVal(fn, a, b),每次间隔 a,a+b,a+2b 的时间，然后写一个 myClear，停止上面的 mySetInterVal(头条)
14. 合并二维有序数组成一维有序数组，归并排序的思路
15. 手写一个ajax实现
16. 尾递归优化实现
17. Object.is的实现
18. React 中 PureComponent 的 ShallowEqual 实现
19. instanceOf实现
20. indexOf、findIndex、find、map、forEach、reduce、splice、slice 实现



答案:
##### 1 手写jsonp
```javascript
(function(window, document) {
  const jsonp = function (url, options, callback){
    let jsonpStr = url.indexOf('?') > -1 ? '&' : '?';
    for (const key in options) {
      if (options.hasOwnProperty(key)) {
        jsonpStr += `${key}=${options[key]}&`;
      }
    }
    const callbackName = Math.random().toString(16).replace('.', '');
    jsonpStr += `callback=${callbackName}`;
    const scriptDom = document.createElement('script');
    scriptDom.src = `${url}${jsonpStr}`;
    window[callbackName] = function (data){
      callback(data);
      document.body.removeChild(scriptDom);
    }
    document.body.appendChild(scriptDom);
  }
  window.$jsonp = jsonp;
})(window, document);
```

##### 2 Promise带中断实现
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

Promise.retry = function (promiseFn, times = 3) {
  return new Promise(async (resolve, reject) => {
    while (times--) {
      try {
        var ret = await promiseFn();
        resolve(ret);
        break;
      } catch (error) {
        if (!times) reject(error);
      }
    }
  });
};
```

##### 3 curry
```javascript
function curry (fn, args = []){
  const len = fn.length;
  return function (..._args) {
    // 必须新建一个变量来承载参数
    let params = args.concat(_args);
    if (params.length < len) {
      return curry.call(this, fn, params);
    }
    return fn.apply(this, params);
  }
}
```

```javascript
// add(1)(2)(3);   //6
// add(1,2,3)(4);  // 10
// add(1)(2)(3)(4)(5)

function add (...args){
  function _adder (..._args){
    args.push(..._args);
    return _adder;
  }
  _adder.toString = function (){
    return args.reduce((a, b) => {
      return a + b;
    }, 0);
  }
  return _adder
}
```

##### 4 call apply bind new create assign

```javascript
Function.prototype.myCall = function (thisArg, ...args) {
  const fn = Symbol('fn');
  thisArg = thisArg || window;
  thisArg[fn] = this;
  const result = thisArg[fn](...args);
  delete thisArg[fn];
  return result;
}

Function.prototype.myApply = function (thisArg, args) {
  const fn = Symbol('fn');
  thisArg = thisArg || window;
  thisArg[fn] = this;
  const result = thisArg[fn](...args);
  delete thisArg[fn];
  return result;
}

Function.prototype.myBind = function (thisArg, ...args) {
  var self = this
  // new优先级
  var funcBind = function () {
    return self.apply(this instanceof self ? this : thisArg, args.concat(Array.prototype.slice.call(arguments)))
  }
  // 继承原型上的属性和方法
  funcBind.prototype = Object.create(self.prototype);
  return funcBind;
}


function mockNew (){
  const Con = Array.prototype.shift.call(arguments);
  let obj = Object.create(Con.prototype);
  let ret = Con.apply(obj, arguments);
  // 这里优先返回构造函数返回的对象
  return ret instanceof Object ? ret : obj;
}

function create (proto, descriptor){
  if (descriptor === null) throw new TypeError();
  function Obj() {}
  Obj.prototype = proto;
  let obj = new Obj();
  if (descriptor !== undefined) Object.defineProperties(obj, descriptor);
  if (proto === null) obj.__proto__ = null;
  return obj;
}

function assign (target, ...source) {
  if (target == null) {
    throw new TypeError('Cannot convert undefined or null to object')
  }
  let ret = Object(target) 
  source.forEach(function(obj) {
    if (obj != null) {
      for (let key in obj) {
        if (obj.hasOwnProperty(key)) {
          ret[key] = obj[key];
        }
      }
    }
  });
  return ret;
}
```

##### 5. 防抖、节流

```javascript
// 防抖
function debounce (fn, time){
  let _timer = undefined;
  return function (...args) {
    if (_timer){
      clearTimeout(_timer);
      _timer = null;
    }
    _timer = setTimeout(fn.bind(this, ...args), time);
  }
}

// 节流
function throttle (fn, time){
  let _last_time = undefined;
  return function (...args){
    const _now_time = new Date();
    if (!_last_time || _now_time - _last_time > time){
      fn.apply(this, args);
      _last_time = _now_time;
    }
  }
}
```

##### 6. promisify实现

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

##### 7. co模块简易实现

```javascript
function run (gen){
  return new Promise((resolve, reject) => {
    const g = gen();
    const _next = (val) => {
      let res = undefined;
      try {
        res = g.next(val);
      } catch (error) {
        return reject(error);
      }
      if(res.done) {
        return resolve(res.value);
      }
      Promise.resolve(res.value).then(v => {
        _next(v);
      }, err => {
        g.throw(err);
      });
      // 好像并不能这么写
      // queueMicrotask(() => {
      //   try {
      //     _next(res.value);
      //   }catch (err) {
      //     g.throw(err);
      //   }
      // });
    }
    _next();
  });
}
```

##### 8. forEach中止

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

##### 9. 深拷贝

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
      target.forEach((e, i) => {
        cloneTarget[i] = clone(e, map);
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

##### 10. 利用Generator函数实现斐波那契数列

```javascript
function* fibonacci() {
  let [prev, curr] = [0, 1];
  for (;;) {
    yield prev;
    [prev, curr] = [curr, prev + curr];
  }
}
```

```javascript
function fib(n){
   function fib_(n,a,b){
       if(n==0)  return a
       else return fib_(n-1,b,a+b)
   }
   return fib_(n,0,1)
}
```

##### 11. 利用Generator函数遍历完全二叉树

```javascript
function Tree(left, label, right) {
  this.left = left;
  this.label = label;
  this.right = right;
}
function make(array) {
  // 判断是否为叶节点
  if (array.length == 1) return new Tree(null, array[0], null);
  return new Tree(make(array[0]), array[1], make(array[2]));
}
let tree = make([[['a'], 'b', ['c']], 'd', [['e'], 'f', ['g']]]);

function* inorder(t) {
  if (t) {
    yield* inorder(t.left);
    yield t.label;
    yield* inorder(t.right);
  }
}
for (let node of inorder(tree)) {
  console.log(node); //'a', 'b', 'c', 'd', 'e', 'f', 'g'
}
```

##### 13. 写一个 mySetInterVal(fn, a, b),每次间隔 a,a+b,a+2b 的时间，然后写一个 myClear，停止上面的 mySetInterVal (头条面试题)

```javascript
function mySetInterVal (fn, a, b){
  let continueAct = true;
  (async function (){
    const sleep = time => new Promise(resolve => {
      setTimeout(() => {
        resolve();
      }, time);
    });
    const getTime = function* (a, b){
      while (true) {
        yield a;
        yield a + b;
        yield a + (2 * b);
      }
    }
    let time = getTime(a, b);
    while(continueAct) {
      await sleep(time.next().value);
      if (!continueAct) return;
      typeof fn === 'function' && fn();
    }
  })();
  return () => {
    continueAct = false;
  }
}

let t = Date.now();
const myClear = mySetInterVal(function (){
  console.log(Date.now() - t);
  t = Date.now();
}, 1000, 3000);

setTimeout(() => {
  myClear();
}, 10000);
```

##### 14. 合并二维有序数组成一维有序数组，归并排序的思路  (头条面试题)

```javascript
function mergeOrderSort (arr){
  const merge = (left, right) => {
    let res = [];
    while(left.length && right.length) {
      if (left[0] - right[0] < 0) res.push(left.shift());
      else res.push(right.shift());
    }
    return res.concat(left, right);
  }
  const mergeOrder = arr => {
    if (arr.length === 0) return [];
    while (arr.length > 1) {
      let arrItem1 = arr.shift();
      let arrItem2 = arr.shift();
      let mergeArr = merge(arrItem1, arrItem2);
      arr.push(mergeArr);
    }
    return arr[0];
  }
  return mergeOrder(arr);
}

let arr1 = [[1,2,3],[4,5,6],[7,8,9],[1,2,3],[4,5,6]];
let arr2 = [[1,4,6],[7,8,10],[2,6,9],[3,7,13],[1,5,12]];
console.log(mergeOrderSort(arr1));
console.log(mergeOrderSort(arr2));
```

##### 15. AJAX

```javascript
function ajax (options){
  let opts = Object.assign({
    url: '',
    data: {},
    async: true,
    dataType: 'json',
    fail: function () {},
    success: function() {},
    timeout: 0,
    withCredentials: false,  //默认为false
  }, options);
  opts.type = (options.type || 'GET').toUpperCase();
  const formatParams = obj => {
    let arr = [];
    for (const key in obj) {
      if (Object.hasOwnProperty.call(obj, key)) {
        arr.push(`${encodeURIComponent(key)}=${encodeURIComponent(obj[key])}`);
      }
    }
    arr.push(`v=${Math.random().toString(16).raplace('.', '')}`);
    return arr.join('&');
  }
  const params = formatParams(opts.data);
  let xhr = null;
  if (window.XMLHttpRequest) {
    xhr = new XMLHttpRequest();
  }else {
    xhr = new ActiveXObject();
  }
  xhr.onreadystatechange = function (){
    if (xhr.readystate === 4) {
      const status = xhr.status;
      if ((status >= 200 && status < 300) || status === 304) {
        opts.success && opts.success(data.dataType === 'json' ? JSON.parse(xhr.responseText) : xhr.responseText);
      }else {
        opts.fail && opts.fail(status);
      }
    }
  }
  xhr.onabort = function (){
    opts.fail('终止');
  }
  xhr.ontimeout = function (){
    opts.fail('超时');
  }
  xhr.onerror = function (err) {
    opts.fail(err);
  }

  if (opts.type === 'GET') {
    xhr.open('GET', `${opts.url}?${params}`, opts.async);
    opts.timeout && opts.timeout > 0 && (xhr.timeout = opts.timeout);
    xhr.withCredentials = opts.withCredentials;
    xhr.send(null);
  }else if (opts.type === 'POST') {
    xhr.open('GET', opts.url, opts.async);
    opts.timeout && options.timeout > 0 && (xhr.timeout = opts.timeout);
    xhr.withCredentials = opts.withCredentials;
    xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
    xhr.send(params);
  }
  return xhr;
}
```

##### 16. 尾递归优化实现

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
sum(1, 100000);
  // 100001

const fibonacci = tco(function (n, x = 0n, y = 1n){
  if (n === 0n) return x;
  return fibonacci(n - 1n, y, x + y);
});
console.log(fibonacci(10000n));  //这里必须使用BigInt 因为数值太大了已经超出JS双精度浮点数的最大范围
```

##### 17. Object.is的实现

```javascript
function is (x, y){
  if (x === y) {
    return x !== 0 || y !== 0 || (1 / x === 1 / y);
  }else {
    return x !== x && y !== y;
  }
}
```

##### 18. ShallowEqual实现

```javascript
function shallowEqual (a, b){
  if (Object.is(a, b)) return true;
  if (typeof a !== 'object' || a === null || typeof b !== 'object' || b === null) return false;
  const aKeys = Object.keys(a);
  const bkeys = Object.keys(b);
  if (aKeys.length !== bkeys.length) return false;
  for (let i = 0; i < aKeys.length; i++){
    if (!Object.prototype.hasOwnProperty.call(b, aKeys[i]) || !Object.is(a[aKeys[i]], b[aKeys[i]])) return false;
  }
  return true;
}
```

##### 19. instanceOf实现

```javascript
function myInstanceOf (obj, constructor){
  const proto = constructor.prototype;
  let obj_proto = Object.getPrototypeOf(obj);
  while (obj_proto) {
    if (proto === obj_proto) return true;
    obj_proto = Object.getPrototypeOf(obj_proto);
  }
  return false;
}
```

##### 20. indexOf 和 findIndex 实现

```javascript
Array.prototype.myIndexOf = function (search, index){
  const len = this.length;
  let fromIndex = index || 0;
  if (len < fromIndex) return -1;
  // fromIndex < 0 && (fromIndex = len - Math.abs(fromIndex));
  fromIndex < 0 && (fromIndex = Math.abs(fromIndex) > len ? 0 : len + fromIndex);
  // 如果是String.prototype.indexOf 如果起始位置小于0，则算为0
  while (fromIndex < len) {
    if (Reflect.has(this, fromIndex) && this[fromIndex] === search) return fromIndex;
    // indexOf内部使用 === 比对
    fromIndex++;
  }
  return -1;
}

Array.prototype.myFindIndex = function (fn, thisArg){
  if (typeof fn !== 'function') {
    throw new TypeError('fn must be a function');
  }
  const len = this.length;
  let index = 0;
  while (index < len) {
    if (fn.call(thisArg, this[index], index, this)) {
      return index;
    }
    index++;
  }
  return -1;
}

Array.prototype.myFind = function (fn, thisArg){
  if (typeof fn !== 'function') {
    throw new TypeError('fn must be a function');
  }
  const len = this.length;
  let index = 0;
  while (index < len) {
    if (fn.call(thisArg, this[index], index, this)) {
      return this[index];
    }
    index++;
  }
  return undefined;
}

Array.prototype.myMap = function (fn, thisArg){
  if (typeof fn !== 'function') {
    throw new TypeError('fn must be a function');
  }
  const len = this.length;
  let index = 0;
  let arr = [];
  while (index < len) {
    if (Reflect.has(this, index)) {  //会过滤empty
      arr[index] = fn.call(thisArg, this[index], index, this);
    }
    index++;
  }
  return arr;
}

Array.prototype.myForEach = function (fn, thisArg){
  if (typeof fn !== 'function') {
    throw new TypeError('fn must be a function');
  }
  const len = this.length;
  let index = 0;
  while (index < len) {
    if (Reflect.has(this, index)) {  //会过滤empty
      fn.call(thisArg, this[index], index, this);
    }
    index++;
  }
}

Array.prototype.myReduce = function (fn, initialValue){
  if (typeof fn !== 'function') {
    throw new TypeError(fn + 'must be a function');
  }
  const len = this.length;
  let index = 0;
  let value = initialValue;
  if (value === undefined) {
    while (index < len && !Reflect.has(this, index)) {
      index++;
    }
    if (index >= len) {
      throw new TypeError('Reduce of empty array with no initial value');
    }
    value = this[index++];
  }
  while (index < len) {
    if (Reflect.has(this, index)){
      value = fn(value, this[index], index, this);
    }
    index++;
  }
  return value;
}

Array.prototype.mySplice = function (start, delNum, ...adds){
  const len = this.length;
  const addCount = adds.length;

  let startIndex = undefined;
  if (start >= len) {
    startIndex = len;
  }else if (start < 0) {
    startIndex = Math.abs(start) > len ? 0 : len + start;
  }else {
    startIndex = start;
  }

  let delCount = undefined;
  if (delNum < 0) {
    delCount = 0;
  }else if (delNum > len - startIndex) {
    delCount = len - startIndex;
  }else {
    delCount = delNum;
  }

  // 密封对象且删除元素个数和添加元素个数不一致
  if(Object.isSealed(array) && delCount !== addCount) {
    throw new TypeError('the array is sealed')
  }
  // 冻结对象
  if(Object.isFrozen(array)) {
    throw new TypeError('the array is frozen');
  }

  let delArr = new Array(delCount);
  for (let i = 0; i < delCount; i++){
    delArr[i] = this[i + startIndex];
  }
  let over = addCount - delCount;
  if (over > 0) {
    for (let i = len - 1; i >= startIndex + delCount; i--){
      this[i + over] = this[i];
    }
  }else if(over < 0) {
    for (let i = startIndex + addCount; i < len; i++){
      if (i + Math.abs(over) > len - 1) {
        Reflect.deleteProperty(this, i);
        continue;
      }
      this[i] = this[i + Math.abs(over)];
    }
  }

  let i = 0;
  while (i < addCount) {
    this[startIndex++] = adds[i++];
  }
  this.length = len + over;
  return delArr;
}

Array.prototype.mySlice = function (start, end){
  const len = this.length;
  let startIndex = start || 0;
  if (startIndex > len) {
    startIndex = len;
  }else if (startIndex < 0) {
    startIndex = Math.abs(startIndex) > len ? 0 : len + startIndex;
  }

  let endIndex = end || len;
  if (endIndex > len) {
    endIndex = len;
  }else if (endIndex < 0) {
    endIndex = Math.abs(endIndex) > len ? 0 : len + endIndex;
  }

  if (startIndex >= endIndex) return [];
  const over = endIndex - startIndex;
  let arr = new Array(over);
  for (let i = 0; i < over; i++){
    arr[i] = this[i + startIndex];
  }
  return arr;
}
```