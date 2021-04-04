## Proxy

Proxy 用于修改某些操作的默认行为，等同于在语言层面做出修改，所以属于一种“元编程”（meta programming），即对编程语言进行编程。

Proxy 可以理解成，在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。

Proxy 实际上重载（overload）了点运算符，即用自己的定义覆盖了语言的原始定义。

`new Proxy(target, handler);`

Proxy 构造函数，用来生成 Proxy 实例，要使得Proxy起作用，必须针对Proxy实例（上例是proxy对象）进行操作，而不是针对目标对象（上例是空对象）进行操作。

其中 `handler` 可以为`{}`却不能为undefined、null等空值。当设置`handler`为`{}`时，没有任何拦截效果，等同于访问原始对象。

Proxy 实例也可以作为其他对象的原型对象。

```javascript
var proxy = new Proxy({}, {
  get: function(target, propKey) {
    return 35;
  }
});

let obj = Object.create(proxy);
obj.time // 35
```

可以拦截多个操作
```javascript
var handler = {
  get: function(target, name) {
    if (name === 'prototype') {
      return Object.prototype;
    }
    return 'Hello, ' + name;
  },

  apply: function(target, thisBinding, args) {
    return args[0];
  },

  construct: function(target, args) {
    return {value: args[1]};
  }
};

var fproxy = new Proxy(function(x, y) {
  return x + y;
}, handler);

fproxy(1, 2) // 1
new fproxy(1, 2) // {value: 2}
fproxy.prototype === Object.prototype // true
fproxy.foo === "Hello, foo" // true
```

对于可以设置、但没有设置拦截的操作，则直接落在目标对象上，按照原先的方式产生结果。

| Proxy 可以拦截的操作 | 含义(其中receiver代表proxy 实例本身，严格地说，是操作行为所针对的对象) |
| --- | --- |
| get(target, propKey[, receiver]) | 拦截对象属性的读取 |
| set(target, propKey, value, receiver) | 拦截对象属性的设置 |
| has(target, propKey) | 拦截propKey in proxy的操作。has方法拦截的是HasProperty操作，而不是HasOwnProperty操作，即has方法不判断一个属性是对象自身的属性，还是继承的属性。 |
| deleteProperty(target, propKey) | 拦截delete proxy\[propKey]的操作，返回一个布尔值。 |
| ownKeys(target) | 拦截Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)、for...in循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而Object.keys()的返回结果仅包括目标对象自身的可遍历属性。 |
| getOwnPropertyDescriptor(target, propKey) | 拦截Object.getOwnPropertyDescriptor(proxy, propKey)，返回属性的描述对象。 |
| defineProperty(target, propKey, propDesc) | 拦截Object.defineProperty(proxy, propKey, propDesc）、Object.defineProperties(proxy, propDescs)，返回一个布尔值。 |
| preventExtensions(target) | 拦截Object.preventExtensions(proxy)，返回一个布尔值。 |
| getPrototypeOf(target) | 拦截Object.getPrototypeOf(proxy)，返回一个对象。 |
| isExtensible(target) | 拦截Object.isExtensible(proxy)，返回一个布尔值。|
| setPrototypeOf(target, proto) |拦截Object.setPrototypeOf(proxy, proto)，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。|
| apply(target, object/thisBinding, args) | 拦截 Proxy 实例作为函数调用的操作，比如proxy(...args)、proxy.call(object, ...args)、proxy.apply(...)。|
| construct(target, args) | 拦截 Proxy 实例作为构造函数调用的操作，比如new proxy(...args)。|

**注意**:
* 拦截方法(get,set等)可以被继承，拦截操作定义在Prototype对象上面。
* get方法的第三个参数的例子，它总是指向原始的读操作所在的那个对象，一般情况下就是 Proxy 实例。(一般可以通过getReceiver获取，`proxy.getReceiver === proxy`)
* 如果一个属性不可配置（configurable）且不可写（writable），则 Proxy 不能拦截方法（诸如get,set等），否则通过 Proxy 对象访问该属性会报错。
* 如果目标对象自身的某个属性不可写，那么set方法将不起作用。
* 严格模式下，set代理如果没有返回true，就会报错。
* apply方法拦截函数的调用、call和apply操作。(设置了Proxy拦截，直接调用Reflect.apply方法，也会被拦截。)
* 如果原对象不可配置或者禁止扩展，这时has拦截会报错。
* construct方法返回的必须是一个对象，否则会报错。
* deleteProperty方法抛出错误或者返回false，当前属性就无法被delete命令删除。

```javascript
const proxy = new Proxy({}, {
  get: function(target, key, receiver) {
    return receiver;
  }
});

const d = Object.create(proxy);
d.a === d // true
/*
d对象本身没有a属性，所以读取d.a的时候，会去d的原型proxy对象找。这时，receiver就指向d，代表原始的读操作所在的那个对象。
*/

const handler = {
  set: function(obj, prop, value, receiver) {
    obj[prop] = receiver;
  }
};
const proxy = new Proxy({}, handler);
const myObj = {};
Object.setPrototypeOf(myObj, proxy);

myObj.foo = 'bar';
myObj.foo === myObj // true
/*
设置myObj.foo属性的值时，myObj并没有foo属性，因此引擎会到myObj的原型链去找foo属性。myObj的原型对象proxy是一个 Proxy 实例，设置它的foo属性会触发set方法。这时，第四个参数receiver就指向原始赋值行为所在的对象myObj。
*/
```

**Proxy.revocable方法返回一个可取消的 Proxy 实例。**
```javascriptlet target = {};
let handler = {};

let {proxy, revoke} = Proxy.revocable(target, handler);

proxy.foo = 123;
proxy.foo // 123

revoke();
proxy.foo // TypeError: Revoked
```
Proxy.revocable方法返回一个对象，该对象的proxy属性是Proxy实例，revoke属性是一个函数，可以取消Proxy实例。上面代码中，当执行revoke函数之后，再访问Proxy实例，就会抛出一个错误。

revoke之后，就收回代理权，不允许再次访问，不论什么操作，都会报错。

**Prxoy的this指向问题**
虽然 Proxy 可以代理针对目标对象的访问，但它不是目标对象的透明代理，即不做任何拦截的情况下，也无法保证与目标对象的行为一致。主要原因就是在 Proxy 代理的情况下，目标对象内部的this关键字会指向 Proxy 代理。

```javascript
const target = {
  m: function () {
    console.log(this === proxy);
  }
};
const handler = {};
const proxy = new Proxy(target, handler);
target.m() // false
proxy.m()  // true
// 一旦proxy代理target.m，后者内部的this就是指向proxy，而不是target。
```

有些原生对象的内部属性，只有通过正确的this才能拿到，所以 Proxy 也无法代理这些原生对象的属性。

this绑定原始对象，可以解决这个问题。

```javascript
const target = new Date('2015-01-01');
const handler = {
  get(target, prop) {
    if (prop === 'getDate') {
      return target.getDate.bind(target);
    }
    return Reflect.get(target, prop);
  }
};
const proxy = new Proxy(target, handler);
proxy.getDate() // 1
```

Proxy 拦截函数内部的this，指向的是handler对象。

#### 实例：Web服务器的客户端

Proxy 对象可以拦截目标对象的任意属性，这使得它很合适用来写 Web 服务的客户端。

```javascript
function createWebService(baseUrl) {
  return new Proxy({}, {
    get(target, propKey, receiver) {
      return () => httpGet(baseUrl + '/' + propKey);
    }
  });
}
const service = createWebService('http://example.com/data');
service.employees().then(json => {
  const employees = JSON.parse(json);
  // ···
});
```

Proxy 可以拦截 createWebService 的任意属性，所以不用为每一种数据写一个适配方法，只要写一个 Proxy 拦截就可以了。

#### Proxy 例子

```javascript
/*get方法*/
// 实现数组读取负数的索引
function createArray(...elements) {
  let handler = {
    get(target, propKey, receiver) {
      let index = Number(propKey);
      if (index < 0) {
        propKey = String(target.length + index);
      }
      return Reflect.get(target, propKey, receiver);
    }
  };

  let target = [];
  target.push(...elements);
  return new Proxy(target, handler);
}
let arr = createArray('a', 'b', 'c');
arr[-1] // c

// 将函数名链式使用
var pipe = function (value) {
  var funcStack = [];
  var oproxy = new Proxy({} , {
    get : function (pipeObject, fnName) {
      if (fnName === 'get') {
        return funcStack.reduce(function (val, fn) {
          return fn(val);
        },value);
      }
      funcStack.push(window[fnName]);
      return oproxy;
    }
  });

  return oproxy;
}

var double = n => n * 2;
var pow    = n => n * n;
var reverseInt = n => n.toString().split("").reverse().join("") | 0;
pipe(3).double.pow.reverseInt.get; // 63

// 生成各种 DOM 节点的通用函数dom
const dom = new Proxy({}, {
  get(target, property) {
    return function(attrs = {}, ...children) {
      const el = document.createElement(property);
      for (let prop of Object.keys(attrs)) {
        el.setAttribute(prop, attrs[prop]);
      }
      for (let child of children) {
        if (typeof child === 'string') {
          child = document.createTextNode(child);
        }
        el.appendChild(child);
      }
      return el;
    }
  }
});
const el = dom.div({},
  'Hello, my name is ',
  dom.a({href: '//example.com'}, 'Mark'),
  '. I like:',
  dom.ul({},
    dom.li({}, 'The web'),
    dom.li({}, 'Food'),
    dom.li({}, '…actually that\'s it')
  )
)
document.body.appendChild(el);
```

## Reflect

#### Reflect对象的设计目

1. 将Object对象的一些明显属于语言内部的方法（比如Object.defineProperty），放到Reflect对象上。现阶段，某些方法同时在Object和Reflect对象上部署，未来的新方法将只部署在Reflect对象上。也就是说，从Reflect对象上可以拿到语言内部的方法。
2. 修改某些Object方法的返回结果，让其变得更合理。比如，Object.defineProperty(obj, name, desc)在无法定义属性时，会抛出一个错误，而Reflect.defineProperty(obj, name, desc)则会返回false。
3. 让Object操作都变成函数行为。某些Object操作是命令式，比如name in obj和delete obj[name]，而Reflect.has(obj, name)和Reflect.deleteProperty(obj, name)让它们变成了函数行为。
4. Reflect对象的方法与Proxy对象的方法一一对应，只要是Proxy对象的方法，就能在Reflect对象上找到对应的方法。这就让Proxy对象可以方便地调用对应的Reflect方法，完成默认行为，作为修改行为的基础。也就是说，不管Proxy怎么修改默认行为，你总可以在Reflect上获取默认行为。

#### Reflect方法

* Reflect.apply(target, thisArg, args)
* Reflect.construct(target, args)
* Reflect.get(target, name, receiver)
* Reflect.set(target, name, value, receiver)
* Reflect.defineProperty(target, name, desc)
* Reflect.deleteProperty(target, name)
* Reflect.has(target, name)
* Reflect.ownKeys(target)
* Reflect.isExtensible(target)
* Reflect.preventExtensions(target)
* Reflect.getOwnPropertyDescriptor(target, name)
* Reflect.getPrototypeOf(target)
* Reflect.setPrototypeOf(target, prototype)

#### 注意

**如果 Proxy 对象和 Reflect 对象联合使用，前者拦截赋值操作，后者完成赋值的默认行为，而且传入了receiver，那么 Reflect.set 会触发 Proxy.defineProperty 拦截。**

```javascript
let p = {
  a: 'a'
};

let handler = {
  set(target, key, value, receiver) {
    console.log('set');
    Reflect.set(target, key, value, receiver)
  },
  defineProperty(target, key, attribute) {
    console.log('defineProperty');
    Reflect.defineProperty(target, key, attribute);
  }
};

let obj = new Proxy(p, handler);
obj.a = 'A';
// set
// defineProperty
```


#### 通过Proxy实现观察者模式

观察者模式（Observer mode）指的是函数自动观察数据对象，一旦对象有变化，函数就会自动执行。

```javascript
const queueObservers = new Set();
const observe = fn => queueObservers.add(fn);
const set = (target, key, value, receiver) => {
  const res = Reflect.set(target, key, value, receiver);
  queueObservers.forEach(fn => {
    typeof fn === 'function' && fn.apply(receiver);
  });
  return res;
}
const observable = target => new Proxy(target, {set});

const person = observable({
  name: 'fangbin',
  age: 18,
});

const changeInfo = function() {
  console.log(this.name, this.age);
}
observe(changeInfo);

person.name = 'bin-fang';
// bin-fang 18

setTimeout(() => {
  person.age = 20;
  // bin-fang 20
}, 2000);
```
