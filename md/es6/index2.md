## 记录es6一些特性的问题

### 对象的新增方法

#### 1. Object.is()

ES5 比较两个值是否相等，只有两个运算符：相等运算符（\==）和严格相等运算符（===）。它们都有缺点，前者会自动转换数据类型，后者的NaN不等于自身，以及+0等于-0。

```javascript
+0 === -0 //true
NaN === NaN // false

Object.is(+0, -0) // false
Object.is(NaN, NaN) // true
```

ES5 可以通过下面的代码，部署`Object.is`

```javascript
Object.defineProperty(Object, 'is', {
  value: function(x, y) {
    if (x === y) {
      // 针对+0 不等于 -0的情况
      return x !== 0 || 1 / x === 1 / y;
    }
    // 针对NaN的情况
    return x !== x && y !== y;
  },
  configurable: true,
  enumerable: false,
  writable: true,
});
```

#### 2. Object.assign()

`Object.assign(target, source)`方法用于对象的合并，将源对象（source）的所有可枚举属性，复制到目标对象（target）。返回值是目标对象。

如果目标对象与源对象有同名属性，或多个源对象有同名属性，则后面的属性会覆盖前面的属性。

**如果只有一个参数，Object.assign()会直接返回该参数。** 如果该参数不是对象，则会先转成对象，然后返回。

由于undefined和null无法转成对象，所以如果它们作为target参数，就会报错。

```javascript
Object.assign(undefined) // 报错
Object.assign(null) // 报错
```

如果非对象参数出现在源对象的位置（即非首参数），那么处理规则有所不同。首先，这些参数都会转成对象，如果无法转成对象，就会跳过。这意味着，如果undefined和null不在首参数，就不会报错。

```javascript
let obj = {a: 1};
Object.assign(obj, undefined) === obj // true
Object.assign(obj, null) === obj // true
```

其他类型的值（即数值、字符串和布尔值）不在首参数，也不会报错。但是，除了字符串会以数组形式，拷贝入目标对象，其他值都不会产生效果。

```javascript
const v1 = 'abc';
const v2 = true;
const v3 = 10;
const v4 = 'efg';

const obj = Object.assign({}, v1, v2, v3, v4);
console.log(obj); // { "0": "e", "1": "f", "2": "g" }  v4作为['e', 'f', 'g'] 覆盖了前面的 v1 ['a', 'b', 'c']
```

**因为只有字符串的包装对象，会产生可枚举属性。**

```javascript
Object.getOwnPropertySymbols(Object('abc').__proto__); //[Symbol(Symbol.iterator)]
```

**Object.assign()拷贝的属性是有限制的，只拷贝源对象的自身属性（不拷贝继承属性），也不拷贝不可枚举的属性（enumerable: false）。**

**Object.assign()只能进行值的复制，如果要复制的值是一个取值函数，那么将求值后再复制。**

```javascript
const source = {
  get foo() { return 1 }
};
const target = {};

Object.assign(target, source)
// { foo: 1 }
```

###### 常见用途

* 为对象添加属性、方法
* 克隆对象（浅拷贝，不能拷贝继承的对象和原型链）
* 合并多个对象
* 为属性指定默认值

#### 3. Object.getOwnPropertyDescriptor() 和 Object.getOwnPropertyDescriptors()

ES5 的Object.getOwnPropertyDescriptor()方法会返回某个对象属性的描述对象（descriptor）。

ES2017 引入了Object.getOwnPropertyDescriptors()方法，返回指定对象所有自身属性（非继承属性）的描述对象。

实现:

```javascript
function getOwnPropertyDescriptors(obj) {
  const result = {};
  for (let key of Reflect.ownKeys(obj)) {
    result[key] = Object.getOwnPropertyDescriptor(obj, key);
  }
  return result;
}
```

**该方法的引入目的，主要是为了解决Object.assign()无法正确拷贝get属性和set属性的问题。**

```javascript
const source = {
  set foo(value) {
    console.log(value);
  }
};

const target2 = {};
Object.defineProperties(target2, Object.getOwnPropertyDescriptors(source));
Object.getOwnPropertyDescriptor(target2, 'foo')
// { get: undefined,
//   set: [Function: set foo],
//   enumerable: true,
//   configurable: true }

// 上面代码中，两个对象合并的逻辑可以写成一个函数。

const shallowMerge = (target, source) => Object.defineProperties(
  target,
  Object.getOwnPropertyDescriptors(source)
);
```

Object.getOwnPropertyDescriptors()方法的另一个用处，是配合Object.create()方法，将对象属性克隆到一个新对象。这属于浅拷贝。

```javascript
const clone = Object.create(Object.getPrototypeOf(obj),
  Object.getOwnPropertyDescriptors(obj));

// 或者

const shallowClone = (obj) => Object.create(
  Object.getPrototypeOf(obj),
  Object.getOwnPropertyDescriptors(obj)
);
```

另外，Object.getOwnPropertyDescriptors()方法可以实现一个对象继承另一个对象。

```javascript
const obj = Object.create(prot);
obj.foo = 123;
// 或者
const obj = Object.assign(
  Object.create(prot),
  {
    foo: 123,
  }
);

// 可以改写为下面方式

const obj = Object.create(
  prot,
  Object.getOwnPropertyDescriptors({
    foo: 123,
  })
);
```

Object.getOwnPropertyDescriptors()也可以用来实现 Mixin（混入）模式。

```javascript
let mix = (object) => ({
  with: (...mixins) => mixins.reduce(
    (c, mixin) => Object.create(
      c, Object.getOwnPropertyDescriptors(mixin)
    ), object)
});

// multiple mixins example
let a = {a: 'a'};
let b = {b: 'b'};
let c = {c: 'c'};
let d = mix(c).with(a, b);

d.c // "c"
d.b // "b"
d.a // "a"
// 返回一个新的对象d，代表了对象a和b被混入了对象c的操作。
```

#### 4. __proto__、Object.setPrototypeOf()、Object.getPrototypeOf()

__proto__前后的双下划线，说明它本质上是一个内部属性，而不是一个正式的对外的 API，只是由于浏览器广泛支持，才被加入了 ES6。标准明确规定，只有浏览器必须部署这个属性，其他运行环境不一定需要部署，而且新的代码最好认为这个属性是不存在的。因此，无论从语义的角度，还是从兼容性的角度，都不要使用这个属性，而是使用下面的`Object.setPrototypeOf()`（写操作）、`Object.getPrototypeOf()`（读操作）、`Object.create()`（生成操作）代替。

`Object.setPrototypeOf`方法的作用与__proto__相同，用来设置一个对象的原型对象（prototype），返回参数对象本身。它是 ES6 正式推荐的设置原型对象的方法。

`Object.getPrototypeOf()` 用于读取一个对象的原型对象。

#### 5. 其他

* `Object.keys()`
* `Object.values()`
* `Object.entries()`
* `Object.fromEntries()`
  Object.fromEntries()方法是Object.entries()的逆操作，用于将一个键值对数组转为对象。
  该方法的主要目的，是将键值对的数据结构还原为对象，因此特别适合将 Map 结构转为对象。
  ```javascript
  // 例一
  const entries = new Map([
    ['foo', 'bar'],
    ['baz', 42]
  ]);

  Object.fromEntries(entries)
  // { foo: "bar", baz: 42 }

  // 例二
  const map = new Map().set('foo', true).set('bar', false);
  Object.fromEntries(map)
  // { foo: true, bar: false }
  ```
  该方法的一个用处是配合URLSearchParams对象，将查询字符串转为对象。
  ```javascript
  Object.fromEntries(new URLSearchParams('foo=bar&baz=qux'))
  // { foo: "bar", baz: "qux" }
  ```

补充:

`URLSearchParams`用来处理URL上的参数串

`Object.fromEntries(new URLSearchParams(window.location.search))`

* URLSearchParams.append(name, value)
* URLSearchParams.delete(name)
* URLSearchParams.entries()
* URLSearchParams.forEach()
* URLSearchParams.get(name)
* URLSearchParams.getAll(name) 返回数组
* URLSearchParams.has(name)
* URLSearchParams.keys()
* URLSearchParams.set(name, value)
* URLSearchParams.sort()
* URLSearchParams.toString()
* URLSearchParams.values()

```javascript
var paramsString = "q=URLUtils.searchParams&topic=api";
var searchParams = new URLSearchParams(paramsString);

//Iterate the search parameters.
for (let p of searchParams) {
  console.log(p);
}

searchParams.has("topic") === true; // true
searchParams.get("topic") === "api"; // true
searchParams.getAll("topic"); // ["api"]
searchParams.get("foo") === null; // true
searchParams.append("topic", "webdev");
searchParams.toString(); // "q=URLUtils.searchParams&topic=api&topic=webdev"
searchParams.set("topic", "More webdev");
searchParams.toString(); // "q=URLUtils.searchParams&topic=More+webdev"
searchParams.delete("topic");
searchParams.toString(); // "q=URLUtils.searchParams"
```

### Symbol

ES5 的对象属性名都是字符串，这容易造成属性名的冲突。如果有一种机制，保证每个属性的名字都是独一无二的就好了，这样就从根本上防止属性名的冲突。这就是 ES6 引入Symbol的原因。

原始数据类型Symbol，表示独一无二的值。

对象的属性名现在可以有两种类型，一种是原来就有的字符串，另一种就是新增的 Symbol 类型。凡是属性名属于 Symbol 类型，就都是独一无二的，可以保证不会与其他属性名产生冲突。

Symbol函数可以接受一个字符串作为参数，表示对 Symbol 实例的描述，主要是为了在控制台显示，或者转为字符串时，比较容易区分。

```javascript
let s1 = Symbol('foo');
let s2 = Symbol('bar');

s1 // Symbol(foo)
s2 // Symbol(bar)

s1.toString() // "Symbol(foo)"
s2.toString() // "Symbol(bar)"
```

Symbol函数的参数只是表示对当前 Symbol 值的描述，因此相同参数的Symbol函数的返回值是不相等的。

Symbol不能与其他类型的值进行运算，其可以显示转化为字符串，也可以转化为布尔值，但是不能转化为数值。

`Symbol.prototype.description` ES2019 提供了一个实例属性description，直接返回 Symbol 的描述。

**Symbol 值作为对象属性名时，不能用点运算符。在对象的内部，使用 Symbol 值定义属性时，Symbol 值必须放在方括号之中。**

Symbol 类型还可以用于定义一组常量，保证这组常量的值都是不相等的。

```javascript
const COLOR_RED    = Symbol();
const COLOR_GREEN  = Symbol();

function getComplement(color) {
  switch (color) {
    case COLOR_RED:
      return COLOR_GREEN;
    case COLOR_GREEN:
      return COLOR_RED;
    default:
      throw new Error('Undefined color');
    }
}
```

常量使用 Symbol 值最大的好处，就是其他任何值都不可能有相同的值了，因此可以保证上面的switch语句会按设计的方式工作。

Symbol可以用来消除魔术字符串，魔术字符串指的是，在代码之中多次出现、与代码形成强耦合的某一个具体的字符串或者数值。

Symbol 作为属性名，遍历对象的时候，该属性不会出现在`for...in、for...of`循环中，也不会被`Object.keys()`、`Object.getOwnPropertyNames()`、`JSON.stringify()`返回。

但是，它也不是私有属性，有一个Object.getOwnPropertySymbols()方法，可以获取指定对象的所有 Symbol 属性名。

Reflect.ownKeys()方法也可以返回所有类型的键名，包括常规键名和 Symbol 键名。

**由于以 Symbol 值作为键名，不会被常规方法遍历得到。我们可以利用这个特性，为对象定义一些非私有的、但又希望只用于内部的方法。**

#### 1. Symbol.for()  Symbol.keyFor()

`Symbol.for()` 接受一个字符串作为参数，然后搜索有没有以该参数作为名称的 Symbol 值。如果有，就返回这个 Symbol 值，否则就新建一个以该字符串为名称的 Symbol 值，并将其注册到全局。

`Symbol.for()`与`Symbol()`这两种写法，都会生成新的 Symbol。它们的区别是，前者会被登记在全局环境中供搜索，后者不会。

`Symbol.keyFor()`方法返回一个已登记的 Symbol 类型值的key。

```javascript
let s1 = Symbol.for("foo");
Symbol.keyFor(s1) // "foo"

let s2 = Symbol("foo");
Symbol.keyFor(s2) // undefined
```

**Symbol.for()为 Symbol 值登记的名字，是全局环境的，不管有没有在全局环境运行。**

#### 2. Singleton模式 (单例模式)

Singleton 模式指的是调用一个类，任何时候返回的都是同一个实例。

```javascript
// mod.js
const FOO_KEY = Symbol('foo');

function A() {
  this.foo = 'hello';
}

if (!global[FOO_KEY]) {
  global[FOO_KEY] = new A();
}

module.exports = global[FOO_KEY];
```

Symbol 可以保证 global[FOO_KEY] 不会被无意间覆盖和改写。

#### 3. 内置的Symbol值

除了定义自己使用的 Symbol 值以外，ES6 还提供了 11 个内置的 Symbol 值，指向语言内部使用的方法。

* `Symbol.hasInstance`
  指向一个内部方法。当其他对象使用instanceof运算符，判断是否为该对象的实例时，会调用这个方法。比如，foo instanceof Foo在语言内部，实际调用的是Foo\[Symbol.hasInstance](foo)。
  ```javascript
  class MyClass {
    [Symbol.hasInstance](foo) {
      return foo instanceof Array;
    }
  }
  [1, 2, 3] instanceof new MyClass() // true
  ```
* `Symbol.isConcatSpreadable`
  该对象用于Array.prototype.concat()时，是否可以展开。默认为undefined,其为true都表示可以展开,false为不可展开。
* `Symbol.species`
  指向一个构造函数。创建衍生对象时，会使用该属性。
* `Symbol.match`
  指向一个函数。当执行str.match(myObject)时，如果该属性存在，会调用它，返回该方法的返回值。
* `Symbol.replace`
  指向一个方法，当该对象被String.prototype.replace方法调用时，会返回该方法的返回值。
* `Symbol.search`
  指向一个方法，当该对象被String.prototype.search方法调用时，会返回该方法的返回值。
* `Symbol.split`
  指向一个方法，当该对象被String.prototype.split方法调用时，会返回该方法的返回值。
* **`Symbol.iterator`**
  对象的Symbol.iterator属性，指向该对象的默认遍历器方法。
  对象进行for...of循环时，会调用Symbol.iterator方法，返回该对象的默认遍历器。
* `Symbol.toPrimitive`
  指向一个方法。该对象被转为原始类型的值时，会调用这个方法，返回该对象对应的原始类型值。
* **`Symbol.toStringTag`**
  指向一个方法。在该对象上面调用Object.prototype.toString方法时，如果这个属性存在，它的返回值会出现在toString方法返回的字符串之中，表示对象的类型。也就是说，这个属性可以用来定制[object Object]或[object Array]中object后面的那个字符串。
  ```javascript
  // 例一
  ({[Symbol.toStringTag]: 'Foo'}.toString())
  // "[object Foo]"

  // 例二
  class Collection {
    get [Symbol.toStringTag]() {
      return 'xxx';
    }
  }
  let x = new Collection();
  Object.prototype.toString.call(x) // "[object xxx]"
  ```
  ES6 新增内置对象的Symbol.toStringTag属性值如下:
    * JSON[Symbol.toStringTag]：'JSON'
    * Math[Symbol.toStringTag]：'Math'
    * Module 对象M[Symbol.toStringTag]：'Module'
    * ArrayBuffer.prototype[Symbol.toStringTag]：'ArrayBuffer'
    * DataView.prototype[Symbol.toStringTag]：'DataView'
    * Map.prototype[Symbol.toStringTag]：'Map'
    * Promise.prototype[Symbol.toStringTag]：'Promise'
    * Set.prototype[Symbol.toStringTag]：'Set'
    %TypedArray%.prototype[Symbol.toStringTag]：'Uint8Array'等
    * WeakMap.prototype[Symbol.toStringTag]：'WeakMap'
    * WeakSet.prototype[Symbol.toStringTag]：'WeakSet'
    * %MapIteratorPrototype%[Symbol.toStringTag]：'Map Iterator'
    * %SetIteratorPrototype%[Symbol.toStringTag]：'Set Iterator'
    * %StringIteratorPrototype%[Symbol.toStringTag]：'String Iterator'
    * Symbol.prototype[Symbol.toStringTag]：'Symbol'
    * Generator.prototype[Symbol.toStringTag]：'Generator'
    * GeneratorFunction.prototype[Symbol.toStringTag]：'GeneratorFunction'
* `Symbol.unscopables`
  指向一个对象。该对象指定了使用with关键字时，哪些属性会被with环境排除。


### Set 和 Map 数据结构

Map和Set是ES6提供的新的数据结构。

#### Set

* Set类似于数组，但是成员的值都是唯一的，没有重复的值。
* Set函数可以接受一个数组（或者具有 iterable 接口的其他数据结构）作为参数，用来初始化。Set本身是一个构造函数，用来生成 Set 数据结构。
* 向 Set 加入值的时候，不会发生类型转换，所以5和"5"是两个不同的值。Set 内部判断两个值是否不同，使用的算法叫做“Same-value-zero equality”，它类似于精确相等运算符（===），主要的区别是向 Set 加入值时认为NaN等于自身，而精确相等运算符认为NaN不等于自身。
* Set.prototype.size：返回Set实例的成员总数。
* Set.prototype.add(value)：添加某个值，返回 Set 结构本身。
* Set.prototype.delete(value)：删除某个值，返回一个布尔值，表示删除是否成功。
* Set.prototype.has(value)：返回一个布尔值，表示该值是否为Set的成员。
* Set.prototype.clear()：清除所有成员，没有返回值。
* Array.from方法可以将 Set 结构转为数组。
* Set.prototype.keys()：返回键名的遍历器
* Set.prototype.values()：返回键值的遍历器
* Set.prototype.entries()：返回键值对的遍历器
* Set.prototype.forEach()：使用回调函数遍历每个成员
* Set的遍历顺序就是插入顺序。

#### Map

* 类似于对象，也是键值对的集合，但是“键”的范围不限于字符串，各种类型的值（包括对象）都可以当作键。也就是说，Object 结构提供了“字符串—值”的对应，Map 结构提供了“值—值”的对应，是一种更完善的 Hash 结构实现。
* Map可以接受一个数组(任何具有 Iterator 接口、且每个成员都是一个双元素的数组的数据结构)作为参数。该数组的成员是一个个表示键值对的数组。
* Map.prototype.size： 返回 Map 结构的成员总数
* Map.prototype.set(key, value)： set方法设置键名key对应的键值为value，然后返回整个 Map 结构。如果key已经有值，则键值会被更新，否则就新生成该键。
* Map.prototype.get(key)： get方法读取key对应的键值，如果找不到key，返回undefined。
* Map.prototype.has(key)： has方法返回一个布尔值，表示某个键是否在当前 Map 对象之中。
* Map.prototype.delete(key)： delete方法删除某个键，返回true。如果删除失败，返回false。
* Map.prototype.clear()： clear方法清除所有成员，没有返回值。
* 只有对同一个对象的引用，Map 结构才将其视为同一个键,Map 的键实际上是跟内存地址绑定的，只要内存地址不一样，就视为两个键。
  ```javascript
  const map = new Map();
  map.set(['a'], 555);
  map.get(['a']) // undefined
  ```
* Map.prototype.keys()：返回键名的遍历器。
* Map.prototype.values()：返回键值的遍历器。
* Map.prototype.entries()：返回所有成员的遍历器。
* Map.prototype.forEach()：遍历 Map 的所有成员。
* Map 的遍历顺序就是插入顺序。

1. Map 转为数组最方便的方法，就是使用扩展运算符
    ```javascript
    const myMap = new Map()
      .set(true, 7)
      .set({foo: 3}, ['abc']);
    [...myMap] // [ [ true, 7 ], [ { foo: 3 }, [ 'abc' ] ] ]
    ```
2. 数组传入 Map 构造函数，就可以转为 Map。(`[[1,2], [3, 4]]`)
3. Map 转为对象
    ```javascript
    function strMapToObj(strMap) {
      let obj = Object.create(null);
      for (let [k,v] of strMap) {
        obj[k] = v;
      }
      return obj;
    }
    ```
    如果所有 Map 的键都是字符串，它可以无损地转为对象。
4. 对象转为 Map
    ```javascript
    let obj = {"a":1, "b":2};
    let map = new Map(Object.entries(obj));
    ```
5. Map 转为 JSON
    * Map 的键名都是字符串，这时可以选择转为对象 JSON。
    * Map 的键名有非字符串，这时可以选择转为数组 JSON。

6. JSON 转为 Map
    * JSON所有键名都是字符串，先转化为对象，然后对象转Map
    * 整个 JSON 就是一个数组，且每个数组成员本身，又是一个有两个成员的数组。先转化为数组，然后直接转Map

#### WeakSet和Set
WeakSet 结构与 Set 类似，也是不重复的值的集合。但是，它与 Set 有两个区别。
* WeakSet 的成员只能是对象（null除外），而不能是其他类型的值。
* WWeakSet 中的对象都是弱引用，即垃圾回收机制不考虑 WeakSet 对该对象的引用，也就是说，如果其他对象都不再引用该对象，那么垃圾回收机制会自动回收该对象所占用的内存，不考虑该对象还存在于 WeakSet 之中。
* WeakSet 的成员是不适合引用的，因为它会随时消失。另外，由于 WeakSet 内部有多少个成员，取决于垃圾回收机制有没有运行，运行前后很可能成员个数是不一样的，而垃圾回收机制何时运行是不可预测的，因此 ES6 规定 WeakSet 不可遍历。

##### WeakSet用处

1. 储存 DOM 节点，而不用担心这些节点从文档移除时，会引发内存泄漏。（存储的对象不会造成内存泄漏）

#### WeakMap和Map的区别，相比Object有什么优点

* WeakMap只接受对象作为键名（null除外），不接受其他类型的值作为键名。
* WeakMap键名所引用的对象都是弱引用，即垃圾回收机制不将该引用考虑在内。因此，只要所引用的对象的其他引用都被清除，垃圾回收机制就会释放该对象所占用的内存。也就是说，一旦不再需要，WeakMap 里面的键名对象和所对应的键值对会自动消失，不用手动删除引用。
* WeakMap既没有遍历操作（即没有keys()、values()和entries()方法），没有size属性，也无法清空，即不支持clear方法。WeakMap只有四个方法可用：get()、set()、has()、delete()。

WeakMap结构有助于防止内存泄漏。

##### WeakMap应用
1. WeakMap 应用的典型场合就是 DOM 节点作为键名。

    ```javascript
    let myWeakmap = new WeakMap();
    myWeakmap.set(
      document.getElementById('logo'),
      {timesClicked: 0})
    ;
    document.getElementById('logo').addEventListener('click', function() {
      let logoData = myWeakmap.get(document.getElementById('logo'));
      logoData.timesClicked++;
    }, false);
    ```
    上面代码中，document.getElementById('logo')是一个 DOM 节点，每当发生click事件，就更新一下状态。我们将这个状态作为键值放在 WeakMap 里，对应的键名就是这个节点对象。一旦这个 DOM 节点删除，该状态就会自动消失，不存在内存泄漏风险。

2. WeakMap 的另一个用处是部署私有属性
    ```javascript
    const _counter = new WeakMap();
    const _action = new WeakMap();
    class Countdown {
      constructor(counter, action) {
        _counter.set(this, counter);
        _action.set(this, action);
      }
      dec() {
        let counter = _counter.get(this);
        if (counter < 1) return;
        counter--;
        _counter.set(this, counter);
        if (counter === 0) {
          _action.get(this)();
        }
      }
    }
    const c = new Countdown(2, () => console.log('DONE'));
    c.dec();
    c.dec();
    ```
    上面代码中，Countdown类的两个内部属性_counter和_action，是实例的弱引用，所以如果删除实例，它们也就随之消失，不会造成内存泄漏。

**注**:
```javascript
const _map = new Map();
const _set = new Set();
const _weak_map = new WeakMap();
const _weak_set = new WeakSet();

typeof _map;  //object
typeof _set;  //object
typeof _weak_map;  //object
typeof _weak_set;  //object

Object.prototype.toString.call(_map);  //[object Map]
Object.prototype.toString.call(_set);  //[object Set]
Object.prototype.toString.call(_weak_map);  //[object WeakMap]
Object.prototype.toString.call(_weak_set);  //[object WeakSet]
```

###### 弱引用
弱引用是指垃圾回收的过程中不会将键名对该对象的引用考虑进去。