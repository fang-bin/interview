# JS面向对象

### ES5如何实现继承（原型链）

* 每个对象都有__proto__属性，该属性指向其原型对象，在调用实例的方法和属性时，如果在实例对象上找不到，就会往原型对象上找
* 构造函数的prototype属性也指向实例的原型对象
* 原型对象的constructor属性指向构造函数

![avatar](https://user-gold-cdn.xitu.io/2020/4/4/17144d68b7d0eea1?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

ES5实现继承的方法:

#### 原型链继承

原理： 让子类的原型对象指向父类的实例。（注意要把子类原型对象的构造函数重新指回子类，要不然子类的构造函数也会指向父类）

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

缺点：
* 由于所有Child实例原型都指向同一个Parent实例, 因此对某个Child实例的父类引用类型变量修改会影响所有的Child实例
* 在创建子类实例时无法向父类构造传参, 即没有实现super()的功能

#### 构造函数继承

    function Parent(name) { this.name = name; }
    Parent.prototype.getName = function() { return this.name; }
    function Child() {
        Parent.call(this, 'fangbin')   // 执行父类构造方法并绑定子类的this, 使得父类中的属性能够赋到子类的this上
    }

缺点:

* 继承不到父类原型上的属性和方法

#### 组合式继承
结合原型链继承和构造函数继承

    function Parent(name) { this.name = name; }
    Parent.prototype.getName = function() { return this.name; }
    function Child() {
        // 构造函数继承
        Parent.call(this, 'fangbin') 
    }
    //原型链继承
    Child.prototype = new Parent();
    Child.prototype.constructor = Child;

缺点: 每次创建子类实例都执行了两次构造函数(Parent.call()和new Parent())，虽然这并不影响对父类的继承，但子类创建实例时，原型中会存在两份相同的属性和方法，这并不优雅。

#### 寄生式组合继承

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

寄生组合式继承，是目前最成熟的继承方式，babel对ES6继承的转化也是使用了寄生组合式继承。

![avatar](https://user-gold-cdn.xitu.io/2020/4/6/1714fd86c8983189?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)