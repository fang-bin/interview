1. `window.name`

    window下自带一个name属性（该属性只能保存字符串，值如果不是字符串，会自动转成字符串），默认为""，你在全局条件下直接`var name = 'xxx'`或者`window.name = 'xxx'` 都可以直接修改这个属性，而且当前标签页刷新之后这个属性也不会回收或者重置，知道你关闭此标签页。

    有的说这个属性可以用来跨域信息传递，不过我个人感觉没什么人会这么用吧.
    [window.name实现的跨域数据传输](cnblogs.com/rainman/archive/2011/02/21/1960044.html)

2. 有关柯里化的面试题

    ```javascript
    // 实现一个add方法，使计算结果能够满足如下预期
    add(1)(2)(3);   //6
    add(1,2,3)(4);  // 10
    add(1)(2)(3)(4)(5)  //15
    ```

    自己手写一个柯里化实现
    ```javascript
    function add (...args){
        function adder (){
            function _adder (..._args){
            args.push(..._args);
            return _adder;
            }
            _adder.toString = function (){
            return args.reduce((a, b) => a + b);
            }
            return _adder;
        }
        return adder(...args);
    }
    ```

    其实它就是下面函数的柯里化函数

    ```javascript
    function add (...args) {
        return args.reduce((a, b) => a + b);
    }
    ```

    **顺便附上一个封装curry函数**
    ```javascript
    function curry (fn, args = []){
        const len = fn.length;
        return (..._args) => {
            args.push(..._args);
            if (args.length < len) {
            return curry.call(this, fn, args);
            }
            return fn.apply(this, args);
        }
    }
    ```


