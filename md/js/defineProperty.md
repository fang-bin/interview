## defineProperty 与 defineProperties
对象是由多个名/值对组成的无序的集合。对象中每个属性对应任意类型的值。

Object.defineProperty定义新属性或修改原有的属性。

**`Object.defineProperty(obj, prop, descriptor)`**

在ECMAScript5中，对每个属性都添加了几个属性类型，来描述这些属性的特点。

* configurable: 表示该属性是否能被delete删除。当其值为false时，其他的特性也不能被改变。默认值为true
* enumerable: 是否能枚举。也就是是否能被for-in遍历。默认值为true
* writable: 是否能修改值。默认为true
* value: 该属性的具体值是多少。默认为undefined
* get: 当我们通过`person.name`访问name的值时，get将被调用。该方法可以自定义返回的具体值是多少。get默认值为undefined
* set: 当我们通过`person.name = 'Jake'`设置name的值时，set方法将被调用。该方法可以自定义设置值的具体方式。set默认值为undefined

**需要注意的是，不能同时设置value、writable 与 get、set的值。**
**请尽量同时设置get、set。如果仅仅只设置了get，那么我们将无法设置该属性值。如果仅仅只设置了set，我们也无法读取该属性的值。**

Object.defineProperty只能设置一个属性的属性特性。当我们想要同时设置多个属性的特性时，需要使用我们之前提到过的Object.defineProperties

```javascript
var dog = {};

Object.defineProperty(dog, 'num', {
  configurable: true,
  value: 'hahha',
  enumerable: true,
  writeable: true,
});

var person = {};
// 通过get与set自定义访问与设置name属性的方式
Object.defineProperty(person, 'name', {
  get: function() {
    // 一直返回fangbin
    return 'fangbin';
  },
  set: function(value) {
    // 设置name属性时，返回该字符串，value为新值
    console.log(value + ' in set');
  }
});


var son = {}
Object.defineProperties(son, {
    name: {
        value: 'lala',
        configurable: true
    },
    age: {
        get: function() {
            return this.value || 3
        },
        set: function(value) {
            this.value = value
        }
    }
})
```