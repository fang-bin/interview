# JS面向对象

### ES5如何实现继承（原型链）

* 每个对象都有__proto__属性，该属性指向其原型对象，在调用实例的方法和属性时，如果在实例对象上找不到，就会往原型对象上找
* 构造函数的prototype属性也指向实例的原型对象
* 原型对象的constructor属性指向构造函数

![原型链](https://github.com/fang-bin/interview/blob/master/image/prototype.png)

创建对象的方式中还包括工厂模式：
```javascript
var createPerson = function(name, age) {
    // 声明一个中间对象，该对象就是工厂模式的模子
    var o = new Object();

    // 依次添加我们需要的属性与方法
    o.name = name;
    o.age = age;
    o.getName = function() {
        return this.name;
    }
    return o;
}
```
缺点: 1. 没有办法识别对象实例的类型(使用instanceof无法识别该对象的类型)

ES5实现继承的方法:

#### 原型链继承

原理： 让子类的原型对象指向父类的实例。（注意要把子类原型对象的构造函数重新指回子类，要不然子类的构造函数也会指向父类）

```javascript
function Parent(name) { this.name = name; }
// 父类的原型方法
Parent.prototype.sleep = function () {}
// 子类
function Child() {}

Child.prototype = new Parent(); // 让子类的原型对象指向父类实例, 这样一来在Child实例中找不到的属性和方法就会到原型对象(父类实例)上寻找
Child.prototype.constructor = Child;
const a = new Child();
console.log(a.getName);
console.log(a.constructor);
```

缺点：
* 由于所有Child实例原型都指向同一个Parent实例, 因此对某个Child实例的父类引用类型变量修改会影响所有的Child实例
* 在创建子类实例时无法向父类构造传参, 即没有实现super()的功能

#### 构造函数继承

```javascript
function Parent(name) { this.name = name; }
Parent.prototype.getName = function() { return this.name; }
function Child() {
    Parent.call(this, 'fangbin')   // 执行父类构造方法并绑定子类的this, 使得父类中的属性能够赋到子类的this上
}
```

缺点:

* 继承不到父类原型上的属性和方法

#### 组合式继承
结合原型链继承和构造函数继承

```javascript
function Parent(name) { this.name = name; }
Parent.prototype.getName = function() { return this.name; }
function Child() {
    // 构造函数继承
    Parent.call(this, 'fangbin') 
}
//原型链继承
Child.prototype = new Parent();
Child.prototype.constructor = Child;
```

缺点: 每次创建子类实例都执行了两次构造函数(Parent.call()和new Parent())，虽然这并不影响对父类的继承，但子类创建实例时，原型中会存在两份相同的属性和方法，这并不优雅。

#### 寄生式组合继承

```javascript
function Parent (name){
  this.name = name;
}
Parent.prototype.sayName = function (){
  return this.name;
}
function Child (name){
  Parent.call(this, name);
}
Child.prototype = Object.create(Parent.prototype); //通过浅拷贝将`指向父类实例`改为`指向父类原型`，这样既减去一次构造函数的执行，又解决了对子类原型的操作会影响到父类原型的问题
Child.prototype.constructor = Child;
```

寄生组合式继承，是目前最成熟的继承方式，babel对ES6继承的转化也是使用了寄生组合式继承。

![面向对象](https://github.com/fang-bin/interview/blob/master/image/oop.jpeg)

#### ES5的继承和ES6中的继承有什么区别
ES5的继承时通过prototype或构造函数机制来实现。即（将将子类的prototype等于父类的实例，再在上面挂在方法）**ES5的继承实质上是先创建子类的实例对象，然后再将父类的方法添加到this上**

ES6的继承机制完全不同，实质上是先创建父类的实例对象this（所以必须先调用父类的super()方法），然后再用子类的构造函数修改this。

具体的：ES6通过class关键字定义类，里面有构造方法，类之间通过extends关键字实现继承。子类必须在constructor方法中调用super方法，否则新建实例报错。因为子类没有自己的this对象，而是继承了父类的this对象，然后对其进行加工。如果不调用super方法，子类得不到this对象。

ES6的继承实现

```javascript
class Father{
  constructor(name){
    this.name = name;
  }
  getName(){
    console.log(this.name);
  }
  //  这里是父类的f方法
  f(){
    console.log('fffffffffffffffffffffff');
  }
}
class Son extends Father{
  constructor(name,age){
    super(name); // HACK: 这里super()要在第一行
    this.age = age;
  }
  getAge(){
    console.log(this.age);
  }
  //  子类的f方法
  f(){
    console.log('sssssssssssssssssssssss');
  }
}
var s1 = new Son('张一',12);
s1.getName();
s1.getAge();
console.log(s1.__proto__); //  为Son，不用修正
s1.f(); //  打印ssssssssssssss
s1.__proto__ = new Father();  //  改变s1的原型指向，改为Father
s1.f();  // 打印ffffffffffffff
console.log(s1.__proto__);  // 为Father
```

ES5的继承实现，上面有说明

##### 封装、继承、多态
###### 封装：把客观事物封装成抽象的类，隐藏属性和方法的实现细节，仅对外公开接口。（将属性和方法组成一个类的过程就是封装。）

###### 继承：子类可以使用父类的所有功能，并且对这些功能进行扩展。继承的过程，就是从一般到特殊的过程。

###### es6和es5继承的区别

大多数浏览器的ES5实现之中，每一个对象都有__proto__属性，指向对应的构造函数的prototype属性。而ES6中Class作为构造函数的语法糖，同时有prototype属性和__proto__属性，因此同时存在两条继承链。
1. 子类的__proto__属性，表示构造函数的继承，总是指向父类。
2. 子类prototype属性的__proto__属性，表示方法的继承，总是指向父类的prototype属性。

```javascript
class A {}
class B extends A {}
console.log(B.__proto__ === A);  //true
console.log(B.prototype.__proto__ === A.prototype);  //true

//这样的结果是因为，类的继承是按照下面的模式实现的:
class A {}

class B {}

// B的实例继承A的实例
Object.setPrototypeOf(B.prototype, A.prototype);

// B继承A的静态属性
Object.setPrototypeOf(B, A);

Object.setPrototypeOf的简单实现如下：

Object.setPrototypeOf = function (obj, proto) {
  obj.__proto__ = proto;
  return obj;
}
```
###### 多态（Polymorphism）按字面的意思就是“多种状态”。在面向对象语言中，接口的多种不同的实现方式即为多态。

#### 面向对象开发的优点

* 代码开发模块化，更易维护和修改。
* 代码复用。
* 增强代码的可靠性和灵活性。
* 增加代码的可理解性。
* 面向对象编程有很多重要的特性，比如：封装，继承，多态和抽象。

#### 面向对象开发的缺点

* 相对于传统的JS面向过程开发具有一定难度

#### 面向对象设计的七大原则 （这个其实和现在前端的组件化封装思想差不多）

1. **开闭原则** 对扩展开放，支持方便的扩展，对修修改关闭，严格限制对已有的内容修改。提高系统的灵活性、可复用性和可维护性。
2. **单一职责原则** 单一职责是高内聚低耦合的一个体现，通俗的讲就是一个类只能负责一个职责,修改一个类不能影响到别的功能,也就是说只有一个导致该类被修改的原因，低耦合性，影响范围小，降低类的复杂度，职责分明，提高了可读性，变更引起的风险低，利于维护。
3. **里士替换原则** 强调设计和实现要依赖于抽象而非具体，子类只能去扩展基类，而不是隐藏或者覆盖基类它包含以下4层含义:
    * 子类可以实现父类的抽象方法，但不能覆盖父类的非抽象方法。
    * 子类中可以增加自己特有的方法。
    * 当子类的方法重载父类的方法时，方法的前置条件（即方法的形参）要比父类方法的输入参数更宽松。
    * 当子类的方法实现父类的抽象方法时，方法的后置条件（即方法的返回值）要比父类更严格。
    
    开闭原则的体现，约束继承泛滥,提高系统的健壮性、扩展性和兼容性。
4. **依赖倒置原则** 程序要依赖于抽象接口，不要依赖于具体实现。 简单的说就是要求对抽象进行编程，不要对实现进行编程，这样就降低了客户与实现模块间的耦合。通俗讲就是引用基类的地方必须能透明地使用其子类的对象，反之则不行（类似，我喜欢动物，那么我喜欢狗。而如果我喜欢狗，则我喜欢动物不一定正确）。减少类间的耦合性，提高系统的稳定性，降低并行开发引起的风险，提高代码的可读性和可维护性。
5. **接口隔离原则** 一个类对另一个类的依赖应该建立在最小的接口上,通俗的讲就是需要什么就提供什么，不需要的就不要提供。高内聚，低耦合，可读性高，易于维护。
6. **迪米特法则/最少知道原则** 每个单元对于其他的单元只能拥有有限的知识：只是与当前单元紧密联系的单元。使得软件更好的可维护性与适应性
对象较少依赖其它对象的内部结构，可以改变对象容器（container）而不用改变它的调用者（caller）。
7. **合成/聚合复用原则** 尽量采用组合(contains-a)、聚合(has-a)的方式而不是继承(is-a)的关系来达到软件的复用目的。如果新对象的某些功能在别的已经创建好的对象里面已经实现，那么应当尽量使用别的对象提供的功能，使之成为新对象的一部分，而不要再重新创建。可以降低类与类之间的耦合程度，提高了系统的灵活性。

#### 非构造函数的继承

要让一个普通对象继承另一个普通对象

```javascript
var Chinese = {
  nation: '中国',
}
function extendObj(o) {
  function F() {}
  F.prototype = o;
  return new F();
}
var Doctor = extendObj(Chinese);
Doctor.career = '医生';
```
除了利用prototype链之外，把父对象的属性全部拷贝给子对象也行（分为深拷贝和浅拷贝）

#### 通过prototype构造函数实现的继承的对象的验证方法

* isPrototypeOf() 用来判断某个proptotype对象和某个实例之间的关系(通过ES6方法实现的继承没办法检验)
* hasOwnProperty() 用来判断该实例对象中的某个属性是本地属性还是继承自prototype对象的属性(ES6中通过super继承的属性，也属于本地属性)
* in 某个实例是否含有某个属性，不管是不是本地属性


#### new实现

```javascript
function New(func) {

    // 声明一个中间对象，该对象为最终返回的实例
    var res = {};
    if (func.prototype !== null) {

        // 将实例的原型指向构造函数的原型
        res.__proto__ = func.prototype;
    }

    // ret为构造函数执行的结果，这里通过apply，将构造函数内部的this指向修改为指向res，即为实例对象
    var ret = func.apply(res, Array.prototype.slice.call(arguments, 1));

    // 当我们在构造函数中明确指定了返回对象时，那么new的执行结果就是该返回对象
    if ((typeof ret === "object" || typeof ret === "function") && ret !== null) {
        return ret;
    }

    // 如果没有明确指定返回对象，则默认返回res，这个res就是实例对象
    return res;
}
```
流程:
1. 声明一个中间对象；
2. 将该中间对象的原型指向构造函数的原型；
3. 将构造函数的this，指向该中间对象；
4. 返回该中间对象，即返回实例对象。


##### jQuery内部有关原型的实现

```javascript
(function(ROOT) {
  var jQuery = function(selector) {
    return new jQuery.fn.init(selector);
  }
  jQuery.fn = jQuery.prototype = {
    constructor: jQuery,
    version: '1.0.0',
    init: function(selector) {
      var ele = document.querySelector(selector);
      this[0] = ele;
      return this;
    }
  }
  jQuery.fn.init.prototype = jQuery.fn;
  jQuery.extend = jQuery.fn.extend = function (options) {
    var target = this;
    for (lef fn in options) {
      target[fn] = options[fn];
    }
    return target;
  }
  ROOT.jQuery = ROOT.$ = jQuery;
})(window);
```

###### 更改原型链的一些骚操作

`Object.setPrototypeOf(obj, obj.\__proto__)`
