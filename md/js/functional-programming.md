# 函数式编程
[函数式编程指北](https://llh911001.gitbooks.io/mostly-adequate-guide-chinese/content/ch1.html#%E4%BB%8B%E7%BB%8D)

如何书写函数式的程序？

通过管道把数据在一系列纯函数间传递的程序

函数式编程思维还具有以下几个特征：

* **函数是第一等公民**
    所谓"第一等公民"（first class），指的是函数与其他数据类型一样，处于平等地位，可以赋值给其他变量，也可以作为参数，传入另一个函数，或者作为别的函数的返回值。

* **只用"表达式"，不用"语句"**
    "表达式"（expression）是一个单纯的运算过程，总是有返回值；"语句"（statement）是执行某种操作，没有返回值。函数式编程要求，只使用表达式，不使用语句。也就是说，每一步都是单纯的运算，而且都有返回值。

* **纯函数**（没有"副作用",单一职责，只做一件事，避免耦合关联。不修改状态, 引用透明）
    相同的输入总会得到相同的输出，并且不会产生副作用的函数，就是纯函数。
    所谓"副作用"（side effect），指的是函数内部与外部互动（最典型的情况，就是修改全局变量的值），产生运算以外的其他结果。
    （副作用是在计算结果的过程中，系统状态的一种变化，或者与外部世界进行的可观察的交互。）
    副作用可能包含，但不限于:
    * 更改文件系统
    * 往数据库插入记录
    * 发送一个 http 请求
    * 可变数据
    * 打印/log
    * 获取用户输入
    * DOM 查询
    * 访问系统状态

    函数式编程强调没有"副作用"，意味着函数要保持独立，所有功能就是返回一个新的值，没有其他行为，尤其是不得修改外部变量的值。

纯函数就是数学上的函数，而且是函数式编程的全部。

纯函数的特性:

**可缓存性（cacheable）**

纯函数总能够根据输入来做缓存。实现缓存的一种典型方式是 memoize 技术

```javascript
var memoize = function(f) {
  var cache = {};
  return function() {
    var arg_str = JSON.stringify(arguments);
    cache[arg_str] = cache[arg_str] || f.apply(f, arguments);
    return cache[arg_str];
  };
};

var squareNumber  = memoize(function(x){ return x*x; });

squareNumber(4); //=> 16
squareNumber(4); // 从缓存中读取输入值为 4 的结果  => 16
squareNumber(5); //=> 25
squareNumber(5); // 从缓存中读取输入值为 5 的结果  => 25

var pureHttpCall = memoize(function(url, params){
  return function() { return $.getJSON(url, params); }
});
```

**可移植性/子文档化(protable/self-documenting)**

纯函数的依赖很明确，因此更易于观察和理解

```javascript
// 不纯的
var signUp = function(attrs) {
  var user = saveUser(attrs);
  welcomeUser(user);
};
var saveUser = function(attrs) {
    var user = Db.save(attrs);
};
var welcomeUser = function(user) {
    Email(user, ...);
};

// 纯的
var signUp = function(Db, Email, attrs) {
  return function() {
    var user = saveUser(Db, attrs);
    welcomeUser(Email, user);
  };
};
var saveUser = function(Db, attrs) {};
var welcomeUser = function(Email, user) {};
```

**可测试性(testable)**

纯函数让测试更加容易。我们不需要伪造一个“真实的”支付网关，或者每一次测试之前都要配置、之后都要断言状态（assert the state）。只需简单地给函数一个输入，然后断言输出就好了。

**合理性(reasonable)**
纯函数最大的好处是引用透明性（referential transparency）。函数的运行不依赖于外部变量或"状态"，只依赖于输入的参数，任何时候只要参数相同，引用函数所得到的返回值总是相同的。

**并行代码**
可以并行运行任意纯函数。因为纯函数根本不需要访问共享的内存，而且根据其定义，纯函数也不会因副作用而进入竞争态（race condition）。

并行代码在服务端 js 环境以及使用了 web worker 的浏览器那里是非常容易实现的，因为它们使用了线程（thread）。不过出于对非纯函数复杂度的考虑，当前主流观点还是避免使用这种并行。

###### 函数式编程的有点:

1. **代码简洁，开发快速** 函数式编程大量使用函数，减少了代码的重复，因此程序比较短，开发速度较快。 
2. **接近自然语言，易于理解** 函数式编程的自由度很高，可以写出很接近自然语言的代码。
3. **更方便的代码管理** 函数式编程不依赖、也不会改变外界的状态，只要给定输入参数，返回的结果必定相同。因此，每一个函数都可以被看做独立单元，很有利于进行单元测试（unit testing）和除错（debugging），以及模块化组合。
4. **易于"并发编程"** 函数式编程不需要考虑"死锁"（deadlock），因为它不修改变量，所以根本不存在"锁"线程的问题。不必担心一个线程的数据，被另一个线程修改，所以可以很放心地把工作分摊到多个线程，部署"并发编程"（concurrency）。
5. **代码的热升级** 函数式编程没有副作用，只要保证接口不变，内部实现是外部无关的。所以，可以在运行状态下直接升级代码，不需要重启，也不需要停机。

备注:

**热更新** :

**死锁(deadlock)** 两个或两个以上的线程在执行过程中，由于竞争资源或者由于彼此通信而造成的一种阻塞的现象，若无外力作用，它们都将无法推进下去。 此时称系统处于死锁状态或系统产生了死锁，这些永远在互相等待的进程称为死锁进程。

### 柯里化(curry)

只传递给函数一部分参数来调用它，让它返回一个函数去处理剩下的参数。

柯里化允许和鼓励你将一个复杂过程分割成一个个更小的更容易分析的过程（这些小的逻辑单元将更容易被理解和测试），最后这样一个难于理解复杂的过程将变成一个个小的逻辑简单的过程的组合。

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

[这里的实现可以参考](https://github.com/mqyqingfeng/Blog/issues/42)

纯函数所说的接受一个输入返回一个输出。curry 函数所做的正是这样：每传递一个参数调用函数，就返回一个新函数处理剩余的参数。这就是一个输入对应一个输出啊。

哪怕输出是另一个函数，它也是纯函数。当然 curry 函数也允许一次传递多个参数，但这只是出于减少 () 的方便。

### 代码组合(compose)

compose函数的作用就是组合函数的，将函数串联起来执行，将多个函数组合起来，一个函数的输出结果是另一个函数的输入参数，一旦第一个函数开始执行，就会像多米诺骨牌一样推导执行了。

* compose的参数是函数，返回的也是一个函数
* 因为除了第一个函数的接受参数，其他函数的接受参数都是上一个函数的返回值，所以初始函数的参数是多元的，而其他函数的接受值是一元的
* compsoe函数可以接受任意的参数，所有的参数都是函数，且执行方向是自右向左的，初始函数一定放到参数的最右面

```javascript
// 普通实现
const compose = function(...args) {
  let length = args.length
  let count = length - 1
  let result
  return function f1 (...arg1) {
    result = args[count].apply(this, arg1)
    if (count <= 0) {
      count = length - 1
      return result
    }
    count--
    return f1.call(null, result)
  }
}

// es6 reduce实现
const compose = (...fns) => (...args) => fns.reduceRight((res, fn) => [fn.call(null, ...res)], args)[0];
```

代码组合的好处是可以大大的提高程序的可读性，比一大堆函数嵌套调用更加简单明了。

组合都符合结合律，符合结合律意味着不管你是把 a 和 b 分到一组，还是把 b 和 c 分到一组都不重要。

```javascript
compose(a, compose(b, c)) == compose(compose(a, b), c);
```

运用结合律能为我们带来强大的灵活性。结合律的一大好处是任何一个函数分组都可以被拆开来，然后再以它们自己的组合方式打包在一起。

例如a、b、c各有功能，我们结合a、b实现了功能d，然后结合d、c实现e功能。

#### pointfree
不使用所要处理的值（没有参数参与），只合成运算过程，叫做pointfree模式/风格。

```javascript
var addOne = x => x + 1;
var square = x => x * x;

var addOneThenSquare = compose(addOne, square);  //使用上面的compose，上面的compose函数调用过程是从右向左
addOneThenSquare(2) //  5
```
ddOneThenSquare是一个合成函数。定义它的时候，根本不需要提到要处理的值，这就是 Pointfree。

Pointfree 的本质就是使用一些通用的函数，组合出各种复杂运算。上层运算不要直接操作数据，而是通过底层函数去处理。这就要求，将一些常用的操作封装成函数。

简单说，Pointfree 就是运算过程抽象化，处理一个值，但是不提到这个值。这样做有很多好处，它能够让代码更清晰和简练，更符合语义，更容易复用，测试也变得轻而易举。

**pointfree事例**

一： 两个方法获取字符串中最长的单词长度和单词

```javascript
const str = 'fangbin zhen shuai a';

const splitWord = str => str.split(' ');
const getWordLength = str => str.length;
const getLengthArr = arr => arr.map(getWordLength);
const compareLength = (last, next) => last >= next ? last : next;
const findMaxNum = arr => arr.reduce(compareLength, 0);
const compareWordLength = (lastWord, nextWord) => getWordLength(lastWord) >= getWordLength(nextWord) ? lastWord : nextWord;
const findMaxEle = arr => arr.reduce(compareWordLength, '');

const getMaxLength = compose(findMaxNum, getLengthArr, splitWord);
const getMaxLengthWord = compose(findMaxEle, splitWord);
console.log(getMaxLength(str))  //7
console.log(getMaxLengthWord(str));  //fangbin
```
(上面获取最长单词的方法总感觉不太好，欢迎意见)

二： 提取下面数据中fangbin的所有未完成任务

```javascript
const data = {
  task: [
    { id: 1, user:'fangbin', complete: false, title: '哈哈哈', dueDate: '2020-4-28', priority: 'high', },
    { id: 2, user:'fangbin', complete: true, title: '呀呀呀', dueDate: '2020-4-28', priority: 'medium', },
    { id: 3, user:'fangbin', complete: false, title: '哇哇哇', dueDate: '2020-4-28', priority: 'medium', },
    { id: 4, user:'fangshiyu', complete: false, title: '米米米', dueDate: '2020-4-28', priority: 'medium', },
    { id: 5, user:'fangbin', complete: true, title: '啦啦啦', dueDate: '2020-4-28', priority: 'medium', },
    { id: 6, user:'fangbin', complete: true, title: '呵呵呵', dueDate: '2020-4-28', priority: 'medium', },
    { id: 7, user:'fangbin', complete: true, title: '哼哼哼', dueDate: '2020-4-28', priority: 'medium', },
  ]
}

const compose = (...fns) => (...args) => fns.reduce((res, fn) => [fn.call(null, ...res)] ,args)[0];  //这个地方为了方便理解，特意写了调用顺序为从左到右
const getProps = prop => data => data[prop];
const filterUser = userName => arr => arr.filter(e => e.user === userName);
const getNoCompleteTasks = arr => arr.filter(e => !e.complete);  //这个地方也可以将比对complete写成一个参数，这样更灵活，就像上面两行
const returnData = props => arr => arr.map(e => {
  let temObj = {};
  props.map(prop => {
    temObj[prop] = e[prop];
  });
  return temObj;
});
const findData = compose(
  getProps('task'), 
  filterUser('fangbin'), 
  getNoCompleteTasks, 
  returnData(['id', 'title', 'dueDate', 'priority'])
);
console.log(findData(data));

//{id: 1, title: "哈哈哈", dueDate: "2020-4-28", priority: "high"}
//{id: 3, title: "哇哇哇", dueDate: "2020-4-28", priority: "medium"}
```

#### pointfree的debug方法

在debug组合的时候总是困难重重，可以使用trace函数来追踪

```javascript
const trace = tag => data => {console.log(`${tag}: ${data}`); return data;}
```
将上面的 trace 函数放在compose中间需要查看的位置 就能打印出在相应位置通道中传递的数据。

#### 有原则的重构

```javascript
// map 的组合律
compose(map(f), map(g)) == map(compose(f, g));  //true
```

重构实例展示
```javascript
// 原有代码
var mediaUrl = _.compose(_.prop('m'), _.prop('media'));
var srcs = _.compose(_.map(mediaUrl), _.prop('items'));
var images = _.compose(_.map(img), srcs);

// 等式推导（equational reasoning）及纯函数的特性，我们可以内联调用 srcs 和 images，也就是把 map 调用排列起来。
var mediaUrl = _.compose(_.prop('m'), _.prop('media'));
var images = _.compose(_.map(img), _.map(mediaUrl), _.prop('items'));

// 把 map 排成一列之后就可以应用组合律了
var mediaUrl = _.compose(_.prop('m'), _.prop('media'));
var images = _.compose(_.map(_.compose(img, mediaUrl)), _.prop('items'));

//现在只需要循环一次就可以把每一个对象都转为 img 标签了。我们把 map 调用的 compose 取出来放到外面，提高一下可读性
var mediaUrl = _.compose(_.prop('m'), _.prop('media'));
var mediaToImg = _.compose(img, mediaUrl);
var images = _.compose(_.map(mediaToImg), _.prop('items'));
```

#### Hindley-Milner(HM 类型签名/类型推断)

类型签名除了能够帮助我们推断函数可能的实现，还能够给我们带来自由定理（free theorems）。

**推断函数的可能**
```javascript
//  strLength :: String -> Number
var strLength = function(s){
  return s.length;
}

//  join :: String -> [String] -> String
var join = curry(function(what, xs){
  return xs.join(what);
});

//  match :: Regex -> String -> [String]
var match = curry(function(reg, s){
  return s.match(reg);
});

//  replace :: Regex -> String -> String -> String
var replace = curry(function(reg, sub, s){
  return s.replace(reg, sub);
});
```

**自由定理**
```javascript
// head :: [a] -> a
compose(f, head) == compose(head, map(f));

// filter :: (a -> Bool) -> [a] -> [a]
compose(map(f), filter(compose(p, f))) == compose(filter(p), map(f));
```
第一个例子中，等式左边说的是，先获取数组的头部（译者注：即第一个元素），然后对它调用函数 f；等式右边说的是，先对数组中的每一个元素调用 f，然后再取其返回结果的头部。这两个表达式的作用是相等的，但是前者要快得多。你可能会想，这不是常识么。但根据我的调查，计算机是没有常识的。实际上，计算机必须要有一种形式化方法来自动进行类似的代码优化。数学提供了这种方法，能够形式化直观的感觉，这无疑对死板的计算机逻辑非常有用。

**类型约束**

```javascript
// sort :: Ord a => [a] -> [a]
```
胖箭头左边表明的是这样一个事实：a 一定是个 Ord 对象。也就是说，a 必须要实现 Ord 接口。Ord 到底是什么？它是从哪来的？在一门强类型语言中，它可能就是一个自定义的接口，能够让不同的值排序。通过这种方式，我们不仅能够获取关于 a 的更多信息，了解 sort 函数具体要干什么，而且还能限制函数的作用范围。我们把这种接口声明叫做类型约束（type constraints）。



#### 命令式编程 声明式编程

**命令式编程**：通过一系列改变程序状态的指令完成计算，模拟计算机的运行过程，告诉计算机怎么做。

**声明式编程**：利用数理逻辑或既定规范对已知条件进行推理运算，更看重结果而非过程，给计算机描述我们想要什么。（将不再指示计算机如何工作，而是指出明确希望得到的结果。与命令式不同，声明式意味着我们要写表达式，而不是一步一步的指示。）

声明式编程的思考层面要高于命令式编程，声明式语言往往通过命令式语言做底层实现。

属于命令式编程的有：汇编、C、面向过程编程、面向对象编程。

属于声明式编程的有：正则表达式、SQL、HTML、CSS、函数式编程。

```javascript
// 命令式
var makes = [];
for (i = 0; i < cars.length; i++) {
  makes.push(cars[i].make);
}


// 声明式
var makes = cars.map(function(car){ return car.make; });
```



###### 参考
[函数式编程之Compose](https://juejin.im/post/59bb6faf6fb9a00a59594395)

###### 闲扯
说的实话，前端世界中有不少不太起眼却能让人眼前一亮转而废寝忘食的东西，比如说我看到js底层执行原理、函数式编程、模块化和数学与计算机算法这些东西，他们或是解决了我长久以来的疑问或是编程过程中很多不足和痛点(有些情况下，就是感觉自己写的很烂，却不知道该怎么解决)，真好。