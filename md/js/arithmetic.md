## 排序算法

#### 冒泡排序
    function bubbleSort (arr){
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

#### 选择排序 
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

#### 插入排序
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

这其中时间复杂度依次是 **插入排序 < 选择排序 < 冒泡排序**

#### 快速排序

[关于快速排序速度分析](http://mindhacks.cn/2008/06/13/why-is-quicksort-so-quick/)

快速排序是分治策略的经典实现，分治的策略如下：
* 分解(Divide)步骤：将问题划分未一些子问题，子问题的形式与原问题一样，只是规模更小
* 解决(Conquer)步骤：递归地求解出子问题。如果子问题的规模足够小，则停止递归，直接求解
* 合并(Combine)步骤：将子问题的解组合成原问题的解

实现方式：

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

上面算法的问题主要是

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
        const pivot = partition(arr, s, e);
        quick_sort(arr, s, pivot);
        quick_sort(arr, pivot + 1, e);
      }
      quick_sort(arr, start, end);
    }

**需要注意:** 在v8中splice是时间复杂度为O(n)，这就意味着每次递归取中间值的时候都需要一个O(n)的运算，这就像极了我们见过的循环套循环的情况，时间复杂度会变成O(n²);

快速排序中选取的基准点可以使用数组中任一元素，最简单直接取第一个。

**快速排序用于大型数据集合进行排序远比上面的排序方式要快，小数据集合时反而比较慢**

#### 归并排序

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

#### 堆排序

#### 基数排序

**在大多数情况下，原生sort方法一直都是最优解**

## 随机算法（洗牌）

#### 原生sort方法

    function outOfOrder (arr){
      arr.sort(() => Math.random() > 0.5 ? -1 : 1);
      return arr;
    }

这种方法比较低效

#### 随机更换位置

    function shuffle (arr) {
      let i = arr.length;
      while(i) {
        const j = ~~(Math.random() * i--);
        [arr[i], arr[j]] = [arr[j], arr[i]];
      }
      return arr;
    }

