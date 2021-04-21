#### 1. 实现一个批量请求函数 multiRequest(urls, maxNum)

要求如下：

1. 要求最大并发数 maxNum
2. 每当有一个请求返回，就留下一个空位，可以增加新的请求
3. 所有请求完成后，结果按照 urls 里面的顺序依次打出

```javascript
function multiRequest(urls, maxNum) {
  return new Promise((resolve, reject) => {
    let i = 0;
    let result = [];
    let endRquest = 0;
    const len = urls.length;
    const fetchFn = (index) => {
      request(urls[index])
        .then(res => {
          result[index] = res;
        })
        .catch(err => {
          result[index] = err;
        })
        .finally(() => {
          endRquest++;
          if (i < len) {
            fetchFn(i);
          }
          if (endRquest === len) {
            resolve(result);
          }
        });
      i++;
      if (i < maxNum) {
        fetchFn(i);
      }
    }
    fetchFn(0);
  });
}

function request(url) {
  return new Promise((r) => {
    console.log(url);
    const time = Math.random() * 10000;
    setTimeout(() => r(url + ':' + time), time);
  });
}

// 测试
let arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 0];
multiRequest(arr, 4).then(res => {
  console.log(res);
});
```

#### 2. 实现一个 normalize 函数，能将输入的特定的字符串转化为特定的结构化数据

字符串仅由小写字母和 [] 组成，且字符串不会包含多余的空格。
示例一: 'abc' --> {value: 'abc'}
示例二：'[abc[bcd[def]]]' --> {value: 'abc', children: {value: 'bcd', children: {value: 'def'}}}

```javascript
const normalize = str => {
  let list = str.match(/\w+/g)
  let obj = {}
  let curr = obj
  while (key = list.shift()) {
      curr.value = key
      if (list.length === 0) break
      curr.children = {}
      curr = curr.children
  }
  return obj
}
```