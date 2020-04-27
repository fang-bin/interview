# 函数式编程

### 柯里化
柯里化是指这样一个函数(假设叫做createCurry)，他接收函数A作为参数，运行后能够返回一个新的函数。并且这个新的函数能够处理函数A的剩余参数。

```javascript
function curry (func, args = []){
  const len = func.length;  //是函数的形参数量
  return (..._args) => {
    _args.push(...args);
    if (_args.length < len){
      return curry.call(this, func, _args);
    }
    return func.apply(this, _args);
  }
}
```