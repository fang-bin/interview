#### 1.

```javascript
const value = { number: 10 };

const multiply = (x = { ...value }) => {
  console.log(x.number *= 2);
};

multiply();
multiply();
multiply(value);
multiply(value);
```

对象的扩展运算符

对象的解构赋值用于从一个对象取值，相当于将目标对象自身的所有可遍历的（enumerable）、但尚未被读取的属性，分配到指定的对象上面。所有的键和它们的值，都会拷贝到新对象上面。

对象的扩展运算符（...）用于取出参数对象的所有可遍历属性，拷贝（浅拷贝）到当前对象之中。

###### 答案: `20 20 20 40`

#### 2.

```javascript
const settings = {
  username: "lydiahallie",
  level: 19,
  health: 90
};

const data = JSON.stringify(settings, ["health", "level"]);
```

`JSON.stringify(value[, replacer[, space]])`

space 指定缩进用的空白字符串，用于美化输出（pretty-print）；如果参数是个数字，它代表有多少的空格；上限为10。该值若小于1，则意味着没有空格；如果该参数为字符串（当字符串长度超过10个字母，取其前10个字母），该字符串将被作为空格；如果该参数没有提供（或者为 null），将没有空格。

JSON.stringify的第二个参数是 替代者(replacer). 替代者(replacer)可以是个函数或数组，用以控制哪些值如何被转换为字符串。

如果替代者(replacer)是个 数组 ，那么就只有包含在数组中的属性将会被转化为字符串。成员的转换顺序与键在数组中的顺序一样。

而如果替代者replacer是个 函数`replacer((index, ele))`(函数应当返回JSON字符串中的value)，这个函数将被对象的每个属性都调用一遍。函数返回的值会成为这个属性的值，最终体现在转化后的JSON字符串中（译者注：Chrome下，经过实验，如果所有属性均返回同一个值的时候有异常，会直接将返回值作为结果输出而不会输出JSON字符串），而如果返回值为undefined，则该属性会被排除在外。

###### 答案 `"{"health":90, "level":19}"`

几种特殊情况:

```javascript
let obj = {a: 1, b: 2};
Object.defineProperty(obj, 'c', {
  configurable: true,
  writable: true,
  value: 3,
  enumerable: false,
});
obj[Symbol.for('fang')] = 1000;

console.log(JSON.stringify(obj));  //{"a":1,"b":2}
console.log(JSON.stringify(obj, [Symbol.for('fang'), 'c', 'b', 'a']));  //{"c":3,"b":2,"a":1}
console.log(JSON.stringify(obj, (index, ele) => 100));  //100
console.log(JSON.stringify(obj, (index, ele) => ele));  //{"a":1,"b":2} 如果返回的是undefined 则会直接过滤掉
```

#### 3.
