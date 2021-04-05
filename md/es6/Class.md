## Class

[JavaScript 语言中，生成实例对象的传统方法是通过构造函数。](../js/oop.md)这种写法跟传统的面向对象语言（比如 C++ 和 Java）差异很大，很容易让新学习这门语言的程序员感到困惑。

ES6 提供了更接近传统语言的写法，引入了 Class（类）这个概念，作为对象的模板。通过class关键字，可以定义类。

基本上，ES6 的class可以看作只是一个语法糖，它的绝大部分功能，ES5 都可以做到，新的class写法只是让对象原型的写法更加清晰、更像面向对象编程的语法而已。

**class写法里面定义的方法，方法与方法之间不需要逗号分隔，加了会报错。**

**构造函数的prototype属性，在 ES6 的“类”上面继续存在。事实上，类的所有方法都定义在类的prototype属性上面。**

**类的内部所有定义的方法，都是不可枚举的（non-enumerable）。这一点与 ES5 的行为不一致。**

```javascript
// ES6
class Point {
  constructor(x, y) {
    // ...
  }

  toString() {
    // ...
  }
}

Object.keys(Point.prototype)
// []
Object.getOwnPropertyNames(Point.prototype)
// ["constructor","toString"]


// ES5
var Point = function (x, y) {
  // ...
};
Point.prototype.toString = function () {
  // ...
};

Object.keys(Point.prototype)
// ["toString"]
Object.getOwnPropertyNames(Point.prototype)
// ["constructor","toString"]
```

#### 1. constructor 方法

constructor()方法是类的默认方法，通过new命令生成对象实例时，自动调用该方法。一个类必须有constructor()方法，如果没有显式定义，一个空的constructor()方法会被默认添加。

constructor()方法默认返回实例对象（即this），完全可以指定返回另外一个对象。

```javascript
class Foo {
  constructor() {
    return Object.create(null);
  }
}

new Foo() instanceof Foo
// false
```

它跟普通构造函数的一个主要区别，后者不用new也可以执行。

#### 2. 类的实例

与 ES5 一样，实例的属性除非显式定义在其本身（即定义在this对象上），否则都是定义在原型上（即定义在class上）。

与 ES5 一样，类的所有实例共享一个原型对象。

#### 3. 取值函数(getter)和存值函数(setter) 

与 ES5 一样，在“类”的内部可以使用get和set关键字，对某个属性设置存值函数和取值函数，拦截该属性的存取行为。

```javascript
class MyClass {
  constructor() {
    // ...
  }
  get prop() {
    return 'getter';
  }
  set prop(value) {
    console.log('setter: '+value);
  }
}

let inst = new MyClass();

inst.prop = 123;
// setter: 123

inst.prop
// 'getter'
```

存值函数和取值函数是设置在属性的 Descriptor 对象上的。

```javascript
class CustomHTMLElement {
  constructor(element) {
    this.element = element;
  }

  get html() {
    return this.element.innerHTML;
  }

  set html(value) {
    this.element.innerHTML = value;
  }
}

var descriptor = Object.getOwnPropertyDescriptor(
  CustomHTMLElement.prototype, "html"
);

"get" in descriptor  // true
"set" in descriptor  // true
```

#### 4. 类的表达式

与函数一样，类也可以使用表达式的形式定义。

```javascript
const MyClass = class Me {
  getClassName() {
    return Me.name;
  }
};

let inst = new MyClass();
inst.getClassName() // Me
Me.name // ReferenceError: Me is not defined
```

上面代码使用表达式定义了一个类。需要注意的是，这个类的名字是Me，但是Me只在 Class 的内部可用，指代当前类。在 Class 外部，这个类只能用MyClass引用。

如果类的内部没用到的话，可以省略Me，也就是可以写成`const MyClass = class { /* ... */ };`这种形式。

#### 5. 注意点

1. 严格模式
    类和模块的内部，默认就是严格模式，所以不需要使用use strict指定运行模式。
2. 不存在提升
    类不存在变量提升（hoist），这一点与 ES5 完全不同。
3. name 属性
    由于本质上，ES6 的类只是 ES5 的构造函数的一层包装，所以函数的许多特性都被Class继承，包括name属性。
    **name属性总是返回紧跟在class关键字后面的类名。**
4. Generator 方法
    如果某个方法之前加上星号（*），就表示该方法是一个 Generator 函数。
5. this指向
    类的方法内部如果含有this，它默认指向类的实例。但是，必须非常小心，一旦单独使用该方法，很可能报错。
    ```javascript
    class Logger {
      printName(name = 'there') {
        this.print(`Hello ${name}`);
      }

      print(text) {
        console.log(text);
      }
    }

    const logger = new Logger();
    const { printName } = logger;
    printName(); // TypeError: Cannot read property 'print' of undefined
    ```
    解决方法
    * 在构造方法中绑定this
    * 使用箭头函数
      箭头函数内部的this总是指向定义时所在的对象。
      箭头函数位于构造函数内部，它的定义生效的时候，是在构造函数执行的时候。这时，箭头函数所在的运行环境，肯定是实例对象，所以this会总是指向实例对象。
    * 使用Proxy，获取方法的时候，自动绑定this

#### 6. 静态方法

类相当于实例的原型，所有在类中定义的方法，都会被实例继承。如果在一个方法前，加上static关键字，就表示该方法不会被实例继承，而是直接通过类来调用，这就称为“静态方法”。

**如果静态方法包含this关键字，这个this指的是类，而不是实例。**

**静态方法可以与非静态方法重名。**

**父类的静态方法，可以被子类继承。**

**静态方法也是可以从super对象上调用的。**

```javascript
class Foo {
  static classMethod() {
    return 'hello';
  }
}

class Bar extends Foo {
  static classMethod() {
    return super.classMethod() + ', too';
  }
}

Bar.classMethod() // "hello, too"
```

#### 7. 实例属性的新写法

实例属性除了定义在constructor()方法里面的this上面，也可以定义在类的最顶层。

#### 8. 静态属性

静态属性指的是 Class 本身的属性，即Class.propName，而不是定义在实例对象（this）上的属性。

```javascript
class Foo {
}

Foo.prop = 1;
Foo.prop // 1
```

目前，只有这种写法可行，因为 ES6 明确规定，Class 内部只有静态方法，没有静态属性。现在有一个提案提供了类的静态属性，写法是在实例属性的前面，加上static关键字。

```javascript
class MyClass {
  static myStaticProp = 42;

  constructor() {
    console.log(MyClass.myStaticProp); // 42
  }
}
```

#### 9. 私有属性和私有方法

私有方法和私有属性，是只能在类的内部访问的方法和属性，外部不能访问。

1. 在命名上加以区别。（这种命名是不保险的，在类的外部，还是可以调用到这个方法。）
2. 将私有方法移出类，因为类内部的所有方法都是对外可见的。
    ```javascript
    class Widget {
      foo (baz) {
        bar.call(this, baz);
      }

      // ...
    }

    function bar(baz) {
      return this.snaf = baz;
    }
    ```
3. 利用Symbol值的唯一性，将私有方法的名字命名为一个Symbol值。
    ```javascript
    const bar = Symbol('bar');
    const snaf = Symbol('snaf');

    export default class myClass{

      // 公有方法
      foo(baz) {
        this[bar](baz);
      }

      // 私有方法
      [bar](baz) {
        return this[snaf] = baz;
      }

      // ...
    };
    ```
    一般情况下无法获取到它们，因此达到了私有方法和私有属性的效果。但是也不是绝对不行，Reflect.ownKeys()依然可以拿到它们。
4. 目前，有一个提案，为class加了私有属性。方法是在属性名之前，使用#表示。只能在类的内部使用（this.#count）。如果在类的外部使用，就会报错。
  之所以要引入一个新的前缀#表示私有属性，而没有采用private关键字，是因为 JavaScript 是一门动态语言，没有类型声明，使用独立的符号似乎是唯一的比较方便可靠的方法，能够准确地区分一种属性是否为私有属性。另外，Ruby 语言使用@表示私有属性，ES6 没有用这个符号而使用#，是因为@已经被留给了 Decorator。
  这种写法不仅可以写私有属性，还可以用来写私有方法。
  另外，私有属性也可以设置 getter 和 setter 方法。
  私有属性和私有方法前面，也可以加上static关键字，表示这是一个静态的私有属性或私有方法。

#### 10. new.target 属性

ES6 为new命令引入了一个new.target属性，该属性一般用在构造函数之中，**返回new命令作用于的那个构造函数**。

如果构造函数不是通过new命令或Reflect.construct()调用的，new.target会返回undefined，因此这个属性可以用来确定构造函数是怎么调用的。

子类继承父类时，生成子类实例时，new.target会返回子类。

## Class 的继承

Class 可以通过extends关键字实现继承，这比 ES5 的通过修改原型链实现继承，要清晰和方便很多。

```javascript
class ColorPoint extends Point {
  constructor(x, y, color) {
    super(x, y); // 调用父类的constructor(x, y)
    this.color = color;
  }

  toString() {
    return this.color + ' ' + super.toString(); // 调用父类的toString()
  }
}
```

**子类必须在constructor方法中调用super方法，否则新建实例时会报错。这是因为子类自己的this对象，必须先通过父类的构造函数完成塑造，得到与父类同样的实例属性和方法，然后再对其进行加工，加上子类自己的实例属性和方法。如果不调用super方法，子类就得不到this对象。**

**ES5 的继承，实质是先创造子类的实例对象this，然后再将父类的方法添加到this上面（Parent.apply(this)）。ES6 的继承机制完全不同，实质是先将父类实例对象的属性和方法，加到this上面（所以必须先调用super方法），然后再用子类的构造函数修改this。**

**另一个需要注意的地方是，在子类的构造函数中，只有调用super之后，才可以使用this关键字，否则会报错。这是因为子类实例的构建，基于父类实例，只有super方法才能调用父类实例。**

**父类的静态方法，也会被子类继承。**

#### Object.getPrototypeOf()

Object.getPrototypeOf方法可以用来从子类上获取父类。

```javascript
class A {}
class B extends A {}

Object.getPrototypeOf(B) === A;  //true
Object.getPrototypeOf(B.prototype) === A.prototype;  //true
```

可以使用这个方法判断，一个类是否继承了另一个类。

### super 关键字

super这个关键字，既可以当作函数使用，也可以当作对象使用。

#### 1. super作为函数调用时，代表父类的构造函数。ES6 要求，子类的构造函数必须执行一次super函数。

```javascript
class A {}

class B extends A {
  constructor() {
    super();
  }
}
```

super虽然代表了父类A的构造函数，但是返回的是子类B的实例，即super内部的this指的是B的实例，因此super()在这里相当于`A.prototype.constructor.call(this)`。

作为函数时，super()只能用在子类的构造函数之中，用在其他地方就会报错。

#### 2. super作为对象时，在普通方法中，指向父类的原型对象；在静态方法中，指向父类。

```javascript
class A {
  p() {
    return 2;
  }
}

class B extends A {
  constructor() {
    super();
    console.log(super.p()); // 2
  }
}

let b = new B();
```

**由于super指向父类的原型对象，所以定义在父类实例上的方法或属性，是无法通过super调用的。**

```javascript
class A {
  constructor() {
    this.p = 2;
  }
}

class B extends A {
  get m() {
    return super.p;
  }
}

let b = new B();
b.m // undefined
```

如果属性定义在父类的原型对象上，super就可以取到。

```javascript
class A {}
A.prototype.x = 2;

class B extends A {
  constructor() {
    super();
    console.log(super.x) // 2
  }
}

let b = new B();
```

**ES6 规定，在子类普通方法中通过super调用父类的方法时，方法内部的this指向当前的子类实例。**

由于this指向子类实例，所以如果通过super对某个属性赋值，这时super就是this，赋值的属性会变成子类实例的属性。

```javascript
class A {
  constructor() {
    this.x = 1;
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
    super.x = 3;
    console.log(super.x); // undefined
    console.log(this.x); // 3
  }
}

let b = new B();
```

**在子类的静态方法中通过super调用父类的方法时，方法内部的this指向当前的子类，而不是子类的实例。**

```javascript
class A {
  constructor() {
    this.x = 1;
  }
  static print() {
    console.log(this.x);
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
  }
  static m() {
    super.print();
  }
}

B.x = 3;
B.m() // 3
```

**使用super的时候，必须显式指定是作为函数、还是作为对象使用，否则会报错。**

```javascript
class A {}

class B extends A {
  constructor() {
    super();
    console.log(super.valueOf() instanceof B); // true
  }
}

let b = new B();
```

上面代码中，super.valueOf()表明super是一个对象，因此就不会报错。同时，由于super使得this指向B的实例，所以super.valueOf()返回的是一个B的实例。

#### 类的 prototype 属性和__proto__属性

大多数浏览器的 ES5 实现之中，每一个对象都有__proto__属性，指向对应的构造函数的prototype属性。Class 作为构造函数的语法糖，同时有prototype属性和__proto__属性，因此同时存在两条继承链。

1. 子类的__proto__属性，表示构造函数的继承，总是指向父类。
2. 子类prototype属性的__proto__属性，表示方法的继承，总是指向父类的prototype属性。

```javascript
class A {
}

class B extends A {
}

B.__proto__ === A // true
B.prototype.__proto__ === A.prototype // true
```

这样的结果是因为，类的继承是按照下面的模式实现的。

```javascript
class A {
}

class B {
}

// B 的实例继承 A 的实例
Object.setPrototypeOf(B.prototype, A.prototype);

// B 继承 A 的静态属性
Object.setPrototypeOf(B, A);

const b = new B();
```

Object.setPrototypeOf方法的实现:

```javascript
Object.setPrototypeOf = function (obj, proto) {
  obj.__proto__ = proto;
  return obj;
}
```

这两条继承链，可以这样理解：作为一个对象，子类（B）的原型（__proto__属性）是父类（A）；作为一个构造函数，子类（B）的原型对象（prototype属性）是父类的原型对象（prototype属性）的实例。

##### 讨论两种情况。

###### 第一种，子类继承Object类。

```javascript
class A extends Object {
}

A.__proto__ === Object // true
A.prototype.__proto__ === Object.prototype // true
```

###### 第二种情况，不存在任何继承。

```javascript
class A {
}

A.__proto__ === Function.prototype // true
A.prototype.__proto__ === Object.prototype // true
```

这种情况下，A作为一个基类（即不存在任何继承），就是一个普通函数，所以直接继承Function.prototype。但是，A调用后返回一个空对象（即Object实例），所以A.prototype.__proto__指向构造函数（Object）的prototype属性。

#### 原生构造函数的继承

原生构造函数是指语言内置的构造函数，通常用来生成数据结构。

* Boolean()
* Number()
* String()
* Array()
* Date()
* Function()
* RegExp()
* Error()
* Object()

ES5 是先新建子类的实例对象this，再将父类的属性添加到子类上，由于父类的内部属性无法获取，导致无法继承原生的构造函数。

ES6 允许继承原生构造函数定义子类，因为 ES6 是先新建父类的实例对象this，然后再用子类的构造函数修饰this，使得父类的所有行为都可以继承。

继承Object的子类，有一个行为差异。

无法通过super方法向父类Object传参。这是因为 ES6 改变了Object构造函数的行为，一旦发现Object方法不是通过new Object()这种形式调用，ES6 规定Object构造函数会忽略参数。

#### Mixin 模式的实现

Mixin 指的是多个对象合成一个新的对象，新对象具有各个组成成员的接口。

将多个类的接口“混入”（mix in）另一个类的方法实现：

```javascript
function mix(...mixins) {
  class Mix {
    constructor() {
      for (let mixin of mixins) {
        copyProperties(this, new mixin()); // 拷贝实例属性
      }
    }
  }

  for (let mixin of mixins) {
    copyProperties(Mix, mixin); // 拷贝静态属性
    copyProperties(Mix.prototype, mixin.prototype); // 拷贝原型属性
  }

  return Mix;
}

function copyProperties(target, source) {
  for (let key of Reflect.ownKeys(source)) {
    if ( key !== 'constructor'
      && key !== 'prototype'
      && key !== 'name'
    ) {
      let desc = Object.getOwnPropertyDescriptor(source, key);
      Object.defineProperty(target, key, desc);
    }
  }
}
```

上面代码的mix函数，可以将多个对象合成为一个类。使用的时候，只要继承这个类即可。

```javascript
class DistributedEdit extends mix(Loggable, Serializable) {
  // ...
}
```
