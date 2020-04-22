# JS面向对象

### ES5如何实现继承（原型链）

* 每个对象都有__proto__属性，该属性指向其原型对象，在调用实例的方法和属性时，如果在实例对象上找不到，就会往原型对象上找
* 构造函数的prototype属性也指向实例的原型对象
* 原型对象的constructor属性指向构造函数

![avatar](https://user-gold-cdn.xitu.io/2020/4/4/17144d68b7d0eea1?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

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

![avatar](https://user-gold-cdn.xitu.io/2020/4/6/1714fd86c8983189?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

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

#### 面向对象开发的优点

* 代码开发模块化，更易维护和修改。
* 代码复用。
* 增强代码的可靠性和灵活性。
* 增加代码的可理解性。
* 面向对象编程有很多重要的特性，比如：封装，继承，多态和抽象。

#### 面向对象开发的缺点

* 相对于传统的JS面向过程开发具有一定难度

#### 面向对象设计的七大原则

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
