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

**缺陷**

Object.defineProperty能够劫持对象的属性，但是需要对对象的每一个属性进行遍历劫持；如果对象上有新增的属性，则需要对新增的属性再次进行劫持；如果属性是对象，还需要深度遍历。这也是为什么Vue给对象新增属性需要通过$set的原因，其原理也是通过Object.defineProperty对新增的属性再次进行劫持。

Object.defineProperty除了能够劫持对象的属性，还可以劫持数组；虽然数组没有属性，但是我们可以把数组的索引看成是属性，不过虽然监听到了数组中元素的变化，但是和监听对象属性面临着同样的问题，就是新增的元素并不会触发监听事件。

为此，Vue的解决方案是劫持Array.property原型链上的7个函数。

1. 无法检测到对象属性的添加或删除
2. 无法检测数组元素的变化，需要进行数组方法的重写
3. 无法检测数组的长度的修改

### 为什么Vue3.0弃用Object.defineProperty，而使用Proxy实现数据监听？

#### Object.defineProperty 和 Proxy 的优缺点对比

1. Object.defineProperty只能劫持对象的属性，而Proxy是直接代理对象。
  由于 Object.defineProperty 只能对属性进行劫持，需要遍历对象的每个属性。而 Proxy 可以直接代理对象。

2. Object.defineProperty对新增属性需要手动进行Observe
  由于 Object.defineProperty 劫持的是对象的属性，所以新增属性时，需要重新遍历对象，对其新增属性再使用 Object.defineProperty 进行劫持。
  也正是因为这个原因，使用vue给 data 中的数组或对象新增属性时，需要使用 vm.$set 才能保证新增的属性也是响应式的。
  如果采用 proxy 实现，Proxy 通过 set(target, propKey, value, receiver) 拦截对象属性的设置，是可以拦截到对象的新增属性的。
  不止如此，Proxy 对数组的方法也可以监测到，不需要像上面vue2.x源码中那样进行 hack。
3. Proxy支持13种拦截操作，这是defineProperty所不具有的
4. 新标准性能红利
  Proxy 作为新标准，长远来看，JS引擎会继续优化 Proxy ，但 getter 和 setter 基本不会再有针对性优化。
5. Proxy兼容性差

Object.defineProperty 和 Proxy 本质差别是，defineProperty 只能对属性进行劫持，新增属性需要手动 Observe 的问题。