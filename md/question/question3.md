1. ts中 public private protected 的区别

* 在TypeScript里，成员都默认为 public。
* 当成员被标记成 private时，它就不能在声明它的类的外部访问。(**在派生类中也不能访问基类的私有属性**)
* protected修饰符与 private修饰符的行为很相似，但有一点不同， protected成员在派生类中仍然可以访问。构造函数也可以被标记成 protected。 这意味着这个类不能在包含它的类外被实例化，但是能被继承。

当比较带有 private或 protected成员的类型的时候，情况就不同了。 如果其中一个类型里包含一个 private成员，那么只有当另外一个类型中也存在这样一个 private成员， 并且它们都是来自同一处声明时，我们才认为这两个类型是兼容的。 对于 protected成员也使用这个规则。