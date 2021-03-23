### 浮点数

双精度浮点数（8个字节，64位）、单精度浮点数（4个字节，32位）、半精度浮点数（2个字节，16位，比较新）都是计算机使用的二进制浮点数据类型。

IEEE二进制浮点数算术标准（IEEE 754）是20世纪80年代以来最广泛使用的浮点数运算标准，其规定了四种浮点数值的方式：单精确度、双精确度、延伸单精确度、延伸双精确度。

#### 单精度浮点数

单精度浮点数使用32位（4个字节）来存储一个浮点数。

![单精度浮点数](https://upload.wikimedia.org/wikipedia/commons/thumb/d/d2/Float_example.svg/590px-Float_example.svg.png)

第1位表示正负，中间8位表示指数，后23位储存有效数位（有效数位是24位）

第一位的正负号0代表正，1代表负。

中间八位共可表示28=256个数，指数可以是二补码；或0到255，0到126代表-127到-1，127代表零，128-255代表1-128。

有效数位最左手边的1并不会储存，因为它一定存在（二进制的第一个有效数字必定是1）。换言之，有效数位是24位，实际储存23位。

#### 双精度浮点数

双精度浮点数使用64位（8个字节）来存储一个浮点数。

![双精度浮点数](https://upload.wikimedia.org/wikipedia/commons/7/76/General_double_precision_float.png)

对于64位的浮点数，最高的1位是符号位S，接着的11位是指数E，剩下的52位为有效数字M。

1. E全为1。这时，如果有效数字M全为0，表示±无穷大（正负取决于符号位s）；如果有效数字M不全为0，表示这个数不是一个数（NaN）。
2. E全为0。这时，浮点数的指数E等于1-127（或者1-1023），有效数字M不再加上第一位的1，而是还原为0.xxxxxx的小数。这样做是为了表示±0，以及接近于0的很小的数字。

3. E不全为0或不全为1。这时，浮点数就采用上面的规则表示，即指数E的计算值减去127（或1023），得到真实值，再将有效数字M前加上第一位的1。

#### 浮点数在计算机中的科学表示方法

![浮点数表示方式](http://chart.googleapis.com/chart?cht=tx&chl=V%20%3D%20(-1)%5Es%5Ctimes%20M%5Ctimes%202%5EE&chs=45)

IEEE 754规定，在计算机内部保存M时，默认这个数的第一位总是1，因此可以被舍去，只保存后面的xxxxxx部分。

E为一个无符号整数（unsigned int）。这意味着，如果E为8位，它的取值范围为0~255；如果E为11位，它的取值范围为0~2047。但是，我们知道，科学计数法中的E是可以出现负数的，所以IEEE 754规定，E的真实值必须再减去一个中间数，对于8位的E，这个中间数是127；对于11位的E，这个中间数是1023。

因为 E 固定长度是 52 位，再加上省略的一位，最多可以表示的数是 2^53=9007199254740992，对应科学计数尾数是 9.007199254740992，这也是 JS 最多能表示的精度。它的长度是 16，所以可以使用 toPrecision(16) 来做精度运算，超过的精度会自动做凑整处理。

#### 大数危机

JavaScript 所有数字都保存成 64 位浮点数，这给数值的表示带来了两大限制。一是数值的精度只能到 53 个二进制位（相当于 16 个十进制位），大于这个范围的整数，JavaScript 是无法精确表示的，这使得 JavaScript 不适合进行科学和金融方面的精确计算。二是大于或等于2的1024次方的数值，JavaScript 无法表示，会返回Infinity。

由于 E 最大值是 1023，所以最大可以表示的整数是 2^1024 - 1，这就是能表示的最大整数。但你并不能这样计算这个数字，因为从 2^1024 开始就变成了 Infinity。

对于 (2^53, 2^63) 之间的数

* (2^53, 2^54) 之间的数会两个选一个，只能精确表示偶数
* (2^54, 2^55) 之间的数会四个选一个，只能精确表示4个倍数
* ... 依次跳过更多2的倍数

ES2020 引入了一种新的数据类型 BigInt（大整数），来解决这个问题，这是 ECMAScript 的第八种数据类型。BigInt 只用来表示整数，没有位数的限制，任何位数的整数都可以精确表示。

BigInt有以下的特点:

1. 为了与 Number 类型区别，BigInt 类型的数据必须添加后缀n。
2. BigInt 与普通整数是两种值，它们之间并不相等。
3. typeof运算符对于 BigInt 类型的数据返回bigint。
4. BigInt 可以使用负号（-），但是不能使用正号（+），因为会与 asm.js 冲突。
5. BigInt 不能与普通数值进行混合运算。
6. 几乎所有的数值运算符都可以用在 BigInt，但是有两个例外。
    * 不带符号的右移位运算符>>>
    * 一元的求正运算符+
7. BigInt构造函数必须有参数，而且参数必须可以正常转为数值，并且不能使用new来初始化新的大数对象

#### toPrecision vs toFixed

数据处理时，这两个函数很容易混淆。它们的共同点是把数字转成字符串供展示使用。注意在计算的中间过程不要使用，只用于最终结果。

1. toPrecision 是处理精度，精度是从左至右第一个不为0的数开始数起。
2. toFixed 是小数点后指定位数取整，从小数点开始数起。

两者都能对多余数字做凑整处理，也有些人用 toFixed 来做四舍五入，但一定要知道它是有 Bug 的。

**如**：1.005.toFixed(2) 返回的是 1.00 而不是 1.01。

**原因**： 1.005 实际对应的数字是 1.00499999999999989，在四舍五入时全部被舍去！

**解法**：使用专业的四舍五入函数 Math.round() 来处理。但 Math.round(1.005 * 100) / 100 还是不行，因为 1.005 * 100 = 100.49999999999999。还需要把乘法和除法精度误差都解决后再使用 Math.round。可以使用后面介绍的 number-precision#round 方法来解决。

### 解决浮点误差

理论上用有限的空间来存储无限的小数是不可能保证精确的，但我们可以处理一下得到我们期望的结果。

##### 数据展示类

当你拿到 1.4000000000000001 这样的数据要展示时，建议使用 `toPrecision` 凑整并 `parseFloat` 转成数字后再显示，如下：

`parseFloat(1.4000000000000001.toPrecision(12)) === 1.4  // True`

封装成方法就是：

```javascript
function strip(num, precision = 12) {
  return +parseFloat(num.toPrecision(precision));
}
```

为什么选择 12 做为默认精度？这是一个经验的选择，一般选12就能解决掉大部分0001和0009问题，而且大部分情况下也够用了，如果你需要更精确可以调高。

##### 数据运算类

对于运算类操作，如 `+-*/`，就不能使用 `toPrecision` 了。正确的做法是把小数转成整数后再运算。以加法为例：

```javascript
/**
 * 精确加法
 */
function add(num1, num2) {
  const num1Digits = (num1.toString().split('.')[1] || '').length;
  const num2Digits = (num2.toString().split('.')[1] || '').length;
  const baseNum = Math.pow(10, Math.max(num1Digits, num2Digits));
  return (num1 * baseNum + num2 * baseNum) / baseNum;
}
```

以上方法能适用于大部分场景。遇到科学计数法如 2.3e+1（当数字精度大于21时，数字会强制转为科学计数法形式显示）时还需要特别处理一下。

[贴一个很小的兼容浮点数精度的库](https://github.com/nefe/number-precision)


### 位运算

| 运算符 | 用法 | 描述 |
| --- | --- | --- |
| 按位与 `&` | `a & b` | 对于每一个比特位，只有两个操作数相应的比特位都是 1 时，结果才为 1，否则为 0。 |
| 按位或 `|` | `a | b` | 对于每一个比特位，当两个操作数相应的比特位至少有一个 1 时，结果为 1，否则为 0。 |
| 按位异或 `^` | `a ^ b` | 对于每一个比特位，当两个操作数相应的比特位有且只有一个 1 时，结果为 1，否则为 0。 |
| 按位非 `~` | `~a` | 反转操作数的比特位，即 0 变成 1，1 变成 0。 |
| 左移 `<<` | `a << b` | 将 a 的二进制形式向左移 b (< 32) 比特位，右边用 0 填充。 |
| 有符号右移 `>>` | `a >> b` | 将 a 的二进制表示向右移 b (< 32) 位，丢弃被移出的位。(新的最左侧的位总是和以前相同，符号位没有被改变。) |
| 无符号右移 `>>>` | `a >>> b` | 将 a 的二进制表示向右移 b (< 32) 位，丢弃被移出的位，并使用 0 在左侧填充。(即便右移 0 个比特，结果也是非负的) |

在JavaScript内部，数值都是以64位浮点数的形式储存，但是做**位运算**的时候，是以 **32位带符号的整数(会舍弃小数)** 进行运算的，并且返回值也是一个32位带符号的整数。

#### 位运算在js中的妙用

1. 使用&运算符判断一个数的奇偶

    ```javascript
    // 偶数 & 1 = 0
    // 奇数 & 1 = 1
    console.log(2 & 1)    // 0
    console.log(3 & 1)    // 1
    ```

2. 使用~, >>, <<, >>>, |来取整

    ```javascript
    console.log(~~ 6.83)    // 6
    console.log(6.83 >> 0)  // 6
    console.log(6.83 << 0)  // 6
    console.log(6.83 | 0)   // 6
    //以上的向下取整仅仅针对于正数，如果是负数则算是向上取整

    // >>>不可对负数取整
    console.log(6.83 >>> 0)   // 6
    ```

3. 使用^来完成值交换

    ```javascript
    var a = 5
    var b = 8
    a ^= b
    b ^= a
    a ^= b
    console.log(a)   // 8
    console.log(b)   // 5
    ```

4. 实现取绝对值

    ```javascript
    function abs(a) {
      const b = a >> 31;// 如果a为正数或0则b为0，如果a是负数则b为-1
      return (a ^ b) - b;
    }
    ```

5. 使用&, >>, |来完成rgb值和16进制颜色值之间的转换

```javascript
/**
 * 16进制颜色值转RGB
 * @param  {String} hex 16进制颜色字符串
 * @return {String}     RGB颜色字符串
 */
  function hexToRGB(hex) {
    var hexx = hex.replace('#', '0x')
    var r = hexx >> 16
    var g = hexx >> 8 & 0xff
    var b = hexx & 0xff
    return `rgb(${r}, ${g}, ${b})`
}

/**
 * RGB颜色转16进制颜色
 * @param  {String} rgb RGB进制颜色字符串
 * @return {String}     16进制颜色字符串
 */
function RGBToHex(rgb) {
    var rgbArr = rgb.split(/[^\d]+/)
    var color = rgbArr[1]<<16 | rgbArr[2]<<8 | rgbArr[3]
    return '#'+ color.toString(16)
}
// -------------------------------------------------
hexToRGB('#ffffff')               // 'rgb(255,255,255)'
RGBToHex('rgb(255,255,255)')      // '#ffffff'
```

#### 权限系统

在权限系统中使用位运算：
* `|`赋予权限
* `&`校验权限
* `^`无责增，有则减
* `&(~code)`删除权限

由于：每种权限码都是唯一的，且每个权限码的二进制数形式，有且只有一位值为 1（2^n），所以权限码只能是在`[-(2^53-1), 2^53-1]`之间的2^n(例如1, 2, 4, 8,...,1024,...)，同一个应用下可用的权限数就非常有限了，这也是该方案的局限性。

为了突破这个限制，引入一个“权限空间”的概念:

1. **权限 code**，字符串，形如 index,pos。其中 pos 表示 32 位二进制数中 1 的位置（其余全是 0）； index 表示权限空间，用于突破 JavaScript 数字位数的限制，是从 0 开始的正整数，每个权限code都要归属于一个权限空间。index 和 pos 使用英文逗号隔开。
2. **用户权限**，字符串，形如 1,16,16。英文逗号分隔每一个权限空间的权限值。例如 1,16,16 的意思就是，权限空间 0 的权限值是 1，权限空间 1 的权限值是 16，权限空间 2 的权限是 16。


几道相关的二进制位运算的算法题:

[二进制中1的个数](https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/solution/jian-dan-yi-dong-wei-yun-suan-js-cpython-by-azl397/)

`n &= (n - 1)`可以消去二进制n中最右边的1

[数值的整数次方](https://leetcode-cn.com/problems/shu-zhi-de-zheng-shu-ci-fang-lcof/)

一定要注意，js的位运算是以32位带符号的整数进行运算的，所以超过Math.pow(2, 31)的数字进行位运算，都可能有问题。