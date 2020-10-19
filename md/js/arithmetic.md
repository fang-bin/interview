## 排序算法

![排序算法总结](https://github.com/fang-bin/interview/blob/master/image/sort.jpg)

#### 冒泡排序

```javascript
function bubbleSort1 (arr){
  const len = arr.length;
  for (let i = 0; i < len; i++){
    let end = true;
    for (let j = 0; j < len - i; j++) {
      if(arr[j] > arr[j + 1]) {
        arr[j] = arr[j] + arr[j + 1];
        arr[j+1] = arr[j] - arr[j + 1];
        arr[j] = arr[j] - arr[j + 1];
        end = false;
      }
    }
    if (end) break;
  }
  return arr;
}
```

还可以对冒泡排序进行优化

```javascript
function bubbleSort1 (arr) {
  if (!Array.isArray(arr)) return;
  const len = arr.length;
  if (len < 2) return arr;
  let i = len - 1;
  let pos = 0;
  while (i > 0) {
    pos = 0;
    for (let j = 0; j < i; j++) {
      if (arr[j + 1] - arr[j] < 0) {
        [arr[j + 1], arr[j]] = [arr[j], arr[j + 1]];
        pos = j;
      }
    }
    i = pos;
  }
  return arr;
}

```

#### 选择排序 

```javascript
function selectSort (arr){
  const len = arr.length;
  let minIndex = 0;
  for (let i = 0; i < len - 1; i++){
    minIndex = i;
    for (let j = i + 1; j < len; j++){
      if (arr[j] < arr[minIndex]){
        minIndex = j;
      }
    }
    if(i !== minIndex) {   //注意使用这种方法替换要注意 i=== minIndex的情况
      arr[i] = arr[i] + arr[minIndex];
      arr[minIndex] = arr[i] - arr[minIndex];
      arr[i] = arr[i] - arr[minIndex];
    }
  }
  return arr;
}
```

#### 插入排序

```javascript
function insertSort (arr){
  let temp = undefined;
  let j = 0;
  for (let i = 1; i < arr.length; i++){
    temp = arr[i];
    j = i;
    while(j > 0 && arr[j - 1] > temp) {
      arr[j] = arr[j - 1];
      j--;
    }
    arr[j] = temp;
  }
  return arr;
}
```

插入排序还可以使用二分法对其优化
```javascript
function insertSort (arr){
  if (!Array.isArray(arr)) return;
  const len = arr.length;
  if (len < 2) return arr;
  let temp = 0,
    left = 0,
    right = len - 1;
  let mid = 0;
  let j = 0;
  for (let i = 1; i < len; i++) {
    temp = arr[i]
    left = 0;
    right = i - 1;
    while(left <= right) {
      mid = ~~((left + right) / 2);
      if (temp < arr[mid]) {
        right = mid - 1;
      }else {
        left = mid + 1;
      }
    }
    j = i;
    while (j > left) {
      arr[j] = arr[j - 1];
      j--;
    }
    arr[left] = temp;
  }
  return arr;
}
```

这其中时间复杂度依次是 **插入排序 < 选择排序 < 冒泡排序**

#### 希尔排序 （Shell Sort）

希尔排序，也称递减增量排序算法，是插入排序的一种更高效的改进版本。但希尔排序是非稳定排序算法。

希尔排序的基本思想是：先将整个待排序的记录序列分割成为若干子序列分别进行直接插入排序，待整个序列中的记录"基本有序"时，再对全体记录进行依次直接插入排序。

注意: 增量元素不互质时，小增量可能不起作用

可以通过以下公式对希尔排序用到的间隔序列进行动态计算

```javascript
const len= this.dataStore.length;   //数组长度
let gap= 1;         //  初始间隔
while (gap< len/3) {     // 动态设置
  gap= 3 * gap + 1;
}
```

```javascript
function shellSort (arr){
  if (!Array.isArray(arr)) return;
  const len = arr.length;
  if (len < 2) return arr;
  let temp = 0,
    j = 0,
    gap = 1;
  while (gap < len / 5) {
    gap = gap * 5 + 1;
  }
  while (gap > 0) {
    for (let i = gap; i < len; i++) {
      temp = arr[i];
      j = i - gap;
      while (j >= 0 && temp - arr[j] < 0) {
        arr[j + gap] = arr[j];
        j -= gap;
      }
      arr[j + gap] = temp;
    }
    gap = ~~(gap / 5);
  }
  return arr;
}
```

#### 

#### 快速排序

[关于快速排序速度分析](http://mindhacks.cn/2008/06/13/why-is-quicksort-so-quick/)

快速排序是分治策略的经典实现，分治的策略如下：
* 分解(Divide)步骤：将问题划分未一些子问题，子问题的形式与原问题一样，只是规模更小
* 解决(Conquer)步骤：递归地求解出子问题。如果子问题的规模足够小，则停止递归，直接求解
* 合并(Combine)步骤：将子问题的解组合成原问题的解

实现方式：

```javascript
function quickSort (arr){
  if (arr.length === 0){
    return [];
  }
  let smallArr = [];
  let pivot = arr[0];  //尽量不要使用splice来。
  let bigArr = [];
  for (let i = 1; i < arr.length; i++){
    if (arr[i] > pivot){
      bigArr.push(arr[i]);
    }else {
      smallArr.push(arr[i]);
    }
  }
  return [...quickSort(smallArr), pivot, ...quickSort(bigArr)];
}
```
看了不少博客，发现ruan大佬的这版本快排有些问题，主要是没有采用原地交换，导致空间复杂度暴涨到O(n㏒n)，下面是改进版：

```javascript
function quickSort (arr){
  const start = 0;
  const end = arr.length -1;
  const swap = (arr, s, e) => {
    [arr[s], arr[e]] = [arr[e], arr[s]];
  }
  const compare = (a, b) => a - b;
  const partition = (arr, s, e) => {
    const pivot = arr[s];
    while(true) {
      while(compare(pivot, arr[s]) > 0) {
        s++;
      }
      while(compare(arr[e], pivot) > 0) {
        e--;
      }
      if (s === e){
        return s;
      }else if (s > e){
        return s - 1;
      }
      swap(arr, s, e);
      s++;
      e--;
    }
  }
  const quick_sort = (arr, s, e) => {
    if (s >= e){
      return;
    }
    const pivotIndex = partition(arr, s, e);
    quick_sort(arr, s, pivotIndex);
    quick_sort(arr, pivotIndex + 1, e);
    return arr;
  }
  return quick_sort(arr, start, end);
}
```

**需要注意:** 在v8中splice是时间复杂度为O(n)，这就意味着每次递归取中间值的时候都需要一个O(n)的运算，这就像极了我们见过的循环套循环的情况，时间复杂度会变成O(n²);

快速排序中选取的基准点可以使用数组中任一元素，最简单直接取第一个。

**快速排序用于大型数据集合进行排序远比上面的排序方式要快，小数据集合时反而比较慢**

#### 归并排序

和选择排序一样，归并排序的性能不受输入数据的影响，但表现比选择排序好的多，因为始终都是O(n log n）的时间复杂度。代价是需要额外的内存空间。

归并排序是采用分治法（Divide and Conquer）的一个非常典型的应用。

```javascript
function mergeSort (arr){
  const merge = (left, right) => {
    let result = [];
    let il = 0;
    let ir = 0;
    while (il < left.length && ir < right.length) {
      if (left[il] < right[ir]){
        result.push(left[il++]);
      }else {
        result.push(right[ir++]);
      }
    }
    il < left.length && (result = result.concat(left.slice(il)));
    ir < right.length && (result = result.concat(right.slice(ir)));
    return result;
  }
  const mergeSortSplit = array => {
    if (array.length === 1){
      return array;
    }
    const midIndex = ~~(array.length / 2);
    const left = array.slice(0, midIndex);
    const right = array.slice(midIndex);
    return merge(mergeSortSplit(left), mergeSortSplit(right));
  }
  return mergeSortSplit(arr);
}
```

#### 堆排序

堆排序（Heapsort）是指利用堆这种数据结构所设计的一种排序算法

```javascript
function heapSort (arr){
  if (!Array.isArray(arr)) return;
  let len = arr.length;
  if (len < 2) return arr;
  const heapify = (array, index, leng) => {
    let left = 2 * index + 1;
    let right = 2 * index + 2;
    let largest = index;
    if (left < leng && array[left] > array[largest]) {
      largest = left;
    }
    if (right < leng && array[right] > array[largest]) {
      largest = right;
    }
    if (largest !== index) {
      [array[index], array[largest]] = [array[largest], array[index]];
      heapify(array, largest, leng);
    }
  }
  for (let i = ~~(len / 2); i >= 0 ; i--) {
    heapify(arr, i, len);
  }

  for (let i = len - 1; i > 0; i--) {
    [arr[0], arr[i]] = [arr[i], arr[0]];
    len--;
    heapify(arr, 0, len);
  }
  return arr;
}
```

#### 计数排序

计数排序的核心在于将输入的数据值转化为键存储在额外开辟的数组空间中。作为一种线性时间复杂度的排序，计数排序要求输入的数据必须是有确定范围的整数。

计数排序不是比较排序，排序的速度快于任何比较排序算法。

计数排序是用来排序0到100之间的数字的最好的算法，但是它不适合按字母顺序排序人名。但是，计数排序可以用在基数排序中的算法来排序数据范围很大的数组。

```javascript
function countingSort (arr){
  if (!Array.isArray(arr)) return;
  const len = arr.length;
  if (len < 2) return arr;
  let min = arr[0],
    max = arr[0],
    counter = [];
  for (let i = 1; i < len; i++) {
    max = max - arr[i] > 0 ? max : arr[i];
    min = arr[i] - min > 0 ? min : arr[i];
    if (!counter[arr[i]]) {
      counter[arr[i]] = 0;
    }
    counter[arr[i]]++;
  }
  let pos = 0;
  for (let j = min; j <= max; j++) {
    if(counter[j]) {
      arr[pos++] = j;
      counter[j]--;
    }
  }
  return arr;
}
```

#### 桶排序

桶排序是计数排序的升级版。

```javascript
function insertSort (arr){
  if (!Array.isArray(arr)) return;
  const len = arr.length;
  if (len < 2) return arr;
  let cur = 0;
  let j = 0;
  for (let i = 1; i < len; i++) {
    cur = arr[i];
    j = i - 1;
    while (j >= 0 && arr[j] - cur > 0) {
      arr[j + 1] = arr[j];
      j--;
    }
    arr[j + 1] = cur;
  }
  return arr;
}

function bucketSort (arr){
  if (!Array.isArray(arr)) return;
  const len = arr.length;
  if (len < 2) return arr;
  const bucketSize = 10;
  let max = arr[0],
    min = arr[0];
  for (let x = 1; x < len; x++) {
    max = max - arr[x] > 0 ? max : arr[x];
    min = arr[x] - min > 0 ? min : arr[x];
  }
  const bucketCount = ~~((max - min) / bucketSize) + 1;
  let bucket = [];
  for (let i = 0; i < bucketCount; i++) {
    bucket[i] = [];
  }
  for (let j = 0; j < len; j++) {
    bucket[~~((arr[j] - min) / bucketSize)].push(arr[j]);
  }
  let pos = 0;
  for (let k = 0; k < bucketCount; k++) {
    insertSort(bucket[k]);
    for (let l = 0; l < bucket[k].length; l++) {
      arr[pos++] = bucket[k][l];
    }
  }
  return arr;
}
```

#### 基数排序

```javascript
function radixSort (arr){
  if (!Array.isArray(arr)) return;
  const len = arr.length;
  if (len < 2) return arr;
  let mod = 10,
    dev = 1;
  let counter = [],
    index = 0;
  let pos = 0,
    value = 0;
  const maxNumberLength = arr.reduce((total, cur) => (total - cur > 0 ? total : cur), -Infinity).toString().length;
  for (let i = 0; i < maxNumberLength; i++, dev *= 10, mod *= 10) {
    for (let j = 0; j < len; j++) {
      index = ~~((arr[j] % mod) / dev);
      if (!counter[index]) {
        counter[index] = [];
      }
      counter[index].push(arr[j]);
    }
    pos = 0;
    for (let k = 0; k < counter.length; k++) {
      if (counter[k]) {
        while (value = counter[k].shift()) {
          arr[pos++] = value;
        }
      }
    }
  }
  return arr;
}
```

**注意** 计数排序、桶排序、基数排序都用了桶的概念，但对桶的使用方法有明显差异

* 基数排序根据键值的每位数字来分配桶
* 计数排序的每个桶只存储单一键值
* 桶排序的每个桶储存一定范围的数值

**在大多数情况下，原生sort方法一直都是最优解**

这个就要引入v8中Array.prototype.sort所使用的算法。

深扒 Array.prototype.sort 算法则发现在v8的 7 版本之前，其使用的是快排和插入排序结合的排序方法，在排序数组长度少于10的时候使用插入排序，其他情况则使用快速排序，

## 字符串匹配算法

```javascript
// KMP算法
var strStr = function(haystack, needle) {
    const len1 = haystack.length;
    const len2 = needle.length;
    if (len2 === 0) return 0;
    let i = 0,
        j = 0,
        curLen = 0;
    while (i <= len1 - len2) {
        while (i <= len1 - len2 && haystack.charAt(i) !== needle.charAt(0)) i++;
        j = 0;
        curLen = 0;
        while(j < len2 && i < len1 && haystack.charAt(i) === needle.charAt(j)) {
            i++;
            j++;
            curLen++;
        }
        if (curLen === len2) return i - curLen;
        i = i - curLen + 1;
    }
    return - 1;
};
```

参考: 
[字符串匹配的KMP算法--阮一峰](http://www.ruanyifeng.com/blog/2013/05/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm.html)
[javascript实现KMP算法--阳光](https://juejin.im/post/6844903910688161806)




## 随机算法（洗牌）

#### 原生sort方法

```javascript
function outOfOrder (arr){
  arr.sort(() => Math.random() > 0.5 ? -1 : 1);
  return arr;
}
```

这种方法比较低效

#### 随机更换位置

```javascript
function shuffle (arr) {
  let i = arr.length;
  while(i) {
    const j = ~~(Math.random() * i--);
    [arr[i], arr[j]] = [arr[j], arr[i]];
  }
  return arr;
}
```


## 计算数组交集

```javascript
function getIntersect (...arrs){
  const baseArr = [...new Set(arrs.splice(0, 1)[0])];
  return arrs.reduce((total, arr) => {
    return total.filter(e => arr.includes(e));
  }, baseArr);
}
```

## 二分法

## 贪心算法



