## Promise/Generator/Async实现原理解析

### Promise
Promise调用流程

1. Promise的构造方法接收一个executor()，在new Promise()时就立刻执行这个executor回调
2. executor()内部的异步任务被放入宏/微任务队列，等待执行
3. then()被执行，收集成功/失败回调，放入成功/失败队列
4. executor()的异步任务被执行，触发resolve/reject，从成功/失败队列中取出回调依次执行

以上流程就是观察者模式，**收集依赖 -> 触发通知 -> 取出依赖执行**，在Promise里，执行顺序是**then收集依赖 -> 异步触发resolve -> resolve执行依赖**

```javascript
const PENDING = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED = 'rejected';
class MyPromise {
  constructor(executor) {
    this._status = PENDING;
    this._val = undefined;
    this._resolveQueen = [];
    this._rejectQueen = [];
    const _resolve = (val) => {
      const run = () => {
        if (this._status !== PENDING) return;   //状态一旦固定下来，就不能更改
        this._status = FULFILLED;
        this._val = val;
        while (this._resolveQueen.length){
          const callback = this._resolveQueen.shift();
          callback && callback(val);
        }
      }
      setTimeout(run);
    }
    const _reject = (val) => {
      const run = () => {
        if (this._status !== PENDING) return;
        this._status = REJECTED;
        this._val = val; 
        while (this._rejectQueen.length) {
          const callback = this._rejectQueen.shift();
          callback && callback(val);
        }
      }
      setTimeout(run);
    }
    executor(_resolve, _reject);
  }
  then(resolveFn, rejectFn) {
    typeof resolveFn !== 'function' && (resolveFn = val => val);
    typeof rejectFn !== 'function' && (rejectFn = error => {
      throw new Error(error instanceof Error ? error.message : error);
    });
    return new MyPromise((resolve, reject) => {
      const fulfilledFn = val => {
        try {
          const result = resolveFn(val);
          result instanceof MyPromise ? result.then(resolve, reject) : resolve(result);
        } catch (error) {
          reject(error);
        }
      }
      const rejectedFn = err => {
        try {
          const result = rejectFn(err);
          result instanceof MyPromise ? result.then(resolve, reject) : resolve(result);   //这个地方要用resolve，因为rejectFn返回的值也可以传递给后面的then
        } catch (error) {
          reject(error);
        }
      }
      switch (this._status) {
        case PENDING:
          this._resolveQueen.push(fulfilledFn);
          this._rejectQueen.push(rejectedFn);
          break;
        case FULFILLED:
          fulfilledFn(this._val);  //状态固定的情况下立刻执行
          break;
        case REJECTED:
          rejectedFn(this._val);
          break;
      }
    });
  }
  catch(rejectFn) {
    return this.then(undefined, rejectFn);
  }
  finally(callback) {
    return this.then(
      value => MyPromise.resolve(callback()).then(() => value),
      error => MyPromise.resolve(callback()).then(() => {throw error}),
    );
  }
  static resolve(val) {
    if (val instanceof MyPromise) return val;
    return new MyPromise(resolve => resolve(val));
  }
  static reject(val) {
    return new MyPromise((resolve, reject) => reject(val));
  }
  static all(promiseArr) {
    let index = 0;
    let resultArr = [];
    return new MyPromise((resolve, reject) => {
      promiseArr.forEach((p, i) => {
        MyPromise.resolve(p).then(val => {
          index++;
          resultArr[i] = val;
          if(index === promiseArr.length) {
            resolve(resultArr);
          }
        }, err => {
          reject(err);
        });
      });
    });
  }
  static race(promiseArr) {
    return new MyPromise((resolve, reject) => {
      for (const p of promiseArr) {
        MyPromise.resolve(p).then(val => {
          resolve(val);
        }, err => {
          reject(err);
        });
      }
    });
  }
  static any(promiseArr) {
    return new MyPromise((resolve, reject) => {
      let index = 0;
      let errorArr = [];
      return new MyPromise((resolve, reject) => {
        promiseArr.forEach((p, i) => {
          MyPromise.resolve(p).then(val => {
            resolve(val);
          }, err => {
            index++;
            errorArr[i] = err;
            if (index === promiseArr.length){
              reject(errorArr);
            }
          });
        });
      });
    });
  }
  static allSettled(promiseArr) {
    let index = 0;
    let resultArr = [];
    return new MyPromise((resolve, reject) => {
      promiseArr.forEach((p, i) => {
        MyPromise.resolve(p).then(val => {
          index++;
          resultArr[i] = {
            status: 'fulfilled',
            value: val,
          };
          if (index === promiseArr.length){
            resolve(resultArr);
          }
        }, err => {
          index++;
          resultArr[i] = {
            status: 'rejected',
            reason: err,
          };
          if (index === promiseArr.length){
            resolve(resultArr);
          }
        });
      });
    });
  }
  // 实现Promise.try
  static try(fn) {
    return new MyPromise(resolve => resolve(typeof fn === 'function' ? fn() : fn));
  }
  // 实现一个可以返回带中断Promise的静态方法，参数只能是Promise
  static abort(promise) {
    if (!(promise instanceof MyPromise)) {
      return MyPromise.reject('错误');
    };
    let _abort = undefined;
    let _abort_promise = new MyPromise((_, reject) => {
      _abort = reject;
    });
    let p = MyPromise.race([promise, _abort_promise]);
    p.abort = _abort;
    return p;
  }
}
```

结合Promise的实现看上面的实现效果
```javascript
Promise
.resolve(1)
.then((res) => {
  console.log(res);  // 1
  return 2;
}).catch((err) => {
  return 3;
})then((res) => {
  console.log(res); // 2
});
```

```javascript
Promise
.reject(1)
.then((res) => {
  console.log(res);
  return 2;
}).catch((err) => {
  console.log(err);  // 1
  return 3;
}).catch(err => {
  console.log(err);
  return 1000;
}).then((res) => {
  console.log(res);  // 3
});
```

据我在v8以往版本中查看，v8中关于promise的实现，在 5.0版本之前完全都是通过js自托管实现的，实现方式原理和上方差不多，但是由于其中的微任务队列由c++代码负责管理，我这里实现的方式用的setTimeout不太合适（这就没办法了）而在5.6-6.0版本只见则取消了promise的js自托管，将一些promise新的静态方法通过js自托管实现,在6.0版本之后，则全部取消了js自托管。

### Generator

Generator实现的核心在于上下文的保存，函数并没有真的被挂起，每一次yield，其实都执行了一遍传入的生成器函数，只是在这个过程中间用了一个context对象储存上下文，使得每次执行生成器函数的时候，都可以从上一个执行结果开始执行，看起来就像函数被挂起了一样

```javascript
function gen$ (_context) {
  switch (_context.prev = _context.next) {
    case 0:
      _context.next = 2;
      return '1';
    case 2:
      _context.next = 4;
      return '2';
    case 4:
      _context.next = 6;
      return '3';
    case 6:
      return _context.stop();
  }
}
let context = {
  next: 0,
  prev: 0,
  done: false,
  stop() {
    this.done = true;
  }
}
let gen = () => {
  return {
    next() {
      const value = context.done ? undefined : gen$(context);
      const done = context.done;
      return {
        value,
        done,
      };
    }
  }
}
const g = gen();
console.log(g.next())
```

**利用Generator函数实现斐波那契数列**

```javascript
function* fibonacci() {
  let [prev, curr] = [0, 1];
  for (;;) {
    yield prev;
    [prev, curr] = [curr, prev + curr];
  }
}

for (let n of fibonacci()) {
  if (n > 1000) break;
  console.log(n);
}
```

同时利用递归也可以实现斐波那契数列

```javascript
function fb1(n){
  if(n <= 2){
    return 1;    
  }else{
    return fb1(n-1) + fb1(n-2);
  }
}
```

**利用Generator函数遍历完全二叉树**

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
```

**遍历二叉树**

```javascript
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

### Async

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
    }
    _next();
  });
}
```
