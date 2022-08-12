# sort（js）

本文主要用于记录阅读红宝书以及各路大佬对于sort函数的见解和阐述，由于本人水平有限，文中如若出现错误和不足还请大佬们不吝珠玉。

## sort函数介绍

在实际开发中，如果我们调用sort函数，sort()默认会按照升序重新排列数组元素，但在使用中我哦们经常会遇见类似下面这中问题

```js
let arr = [10,5,23,4,3]
console.log(arr.sort());
//[10, 23, 3, 4, 5]
```

此时我们在浏览器控制台可以看到给出的结果是[10, 23, 3, 4, 5]，而不是按照升序排列的[3, 4, 5，10, 23]，像是升序了，但又没完全升序，这是为什么呢？

我们可以在红宝书里找到答案。

原来，sort()会在数组每一项上调用String()转型函数，然后通过比较字符串来决定顺序，就算数组元素都是number类型，也会把数组转换成字符串再进行排序比较。所以在上面的例子中，即使5小于10和23，但依旧会排在最后面。

如何解决这个问题呢？

sort()可以接收一个函数，我们一般称之为`compareFunction`(比较函数)，用于判断应该依照什么样的规则来对数组进行排序。这里给出红宝书对于compareFunction给出的实例：

```js
function compare(value1, value2) {
        if (value1 < value2) {
            return -1
        } else if (value1 > value2) {
            return 1
        } else {
            return 0
        }
    }
//不难看出，当升序排列时，compareFunction用于接收两个参数，如果value1应该排在前面，就返回负值，反之则返回正值，如果两个参数值相等，就返回0
```

这个函数有什么用呢，我们来试一下。

```js
 let arr = [10, 5, 23, 4, 3]
    function compare(value1, value2) {
        if (value1 < value2) {
            return -1
        } else if (value1 > value2) {
            return 1
        } else {
            return 0
        }
    }
    console.log(arr.sort(compare));
	//[3, 4, 5, 10, 23]
```

我们将compare函数传入sort(),此时浏览器输出的结果就和我们想要的结果是一致的，当然，如果我们想让数组降序排列，也可以通过操作compare函数的返回值来实现：

```js
 let arr = [10, 5, 23, 4, 3]
    function compare(value1, value2) {
        if (value1 < value2) {
            return 1
        } else if (value1 > value2) {
            return -1
        } else {
            return 0
        }
    }
    console.log(arr.sort(compare));
	//[23, 10, 5, 4, 3]
```

只是简单的将函数返回值进行调换，我们便可以实现降序排列的需求。

当然，如果在实际开发中每次都需要书写compare函数，代码未免太过繁杂，所以在es6之后，我们可以将这个函数简写成一个箭头函数：

```js
 	let arr = [10, 5, 23, 4, 3]
    const ans1 = arr.sort((a, b) => a < b ? -1 : a > b ? 1 : 0)
    console.log(ans1);//[3, 4, 5, 10, 23]
	//同样的，交换返回值的位置就可以实现降序排列
	const ans2 = arr.sort((a, b) => a < b ? 1 : a > b ? -1 : 0)
    console.log(ans2);//[23, 10, 5, 4, 3]
```

如果数组的元素都是数值可以进一步简化：

```js
	function compare (value1, value2) {
    	return value1 - value2
	}
	let arr = [10, 5, 23, 4, 3]
	arr.sort(compare)
	console.log(arr) //[3, 4, 5, 10, 23]
	//同理，交换函数返回值可以实现降序排列
```

总的来说，compareFunction函数的作用可以总结如下：

- 如果 `compareFunction（a, b）`小于 `0`，那么 `a` 会被排列到 `b` 之前；
- 如果 `compareFunction（a, b）`等于 `0`，`a`和 `b` 的相对位置不变；
- 如果 `compareFunction（a, b）`大于 `0`，`b`会被排列到 `a`之前。

## sort()底层原理

 通过上面对sort函数的简单介绍，相信大家对`JS`数组的 `sort`方法已经不陌生了。那么它的内部是如何实现的呢？为什么在调用sort函数之后，数组就可以按照我们给出的条件进行排列呢，我们在这里简单结合浏览器V8引擎源码来分析一下。

通过阅读源码，可以知道，sort在进行排序时有两种方法：`InsertionSort(插入排序)`和`QuickSort(快速排序)`

sort大概的源码构架：

```js
function InnerArraySort(array, length, comparefn) {
  // In-place QuickSort algorithm.
  // For short (length <= 10) arrays, insertion sort is used for efficiency.
	
  //判断是否传入compare函数
  if (!IS_CALLABLE(comparefn)) {
    comparefn = function (x, y) {
      if (x === y) return 0;
      if (%_IsSmi(x) && %_IsSmi(y)) {
        return %SmiLexicographicCompare(x, y);
      }
      x = TO_STRING(x);
      y = TO_STRING(y);
      if (x == y) return 0;
      else return x < y ? -1 : 1;
    };
  }
    //插入排序
  function InsertionSort(a, from, to) {
    for (var i = from + 1; i < to; i++) {
      var element = a[i];
      for (var j = i - 1; j >= from; j--) {
        var tmp = a[j];
        var order = comparefn(tmp, element);
        if (order > 0) {
          a[j + 1] = tmp;
        } else {
          break;
        }
      }
      a[j + 1] = element;
    }
  };
	//寻找哨兵元素位置
  function GetThirdIndex(a, from, to) {
    var t_array = new InternalArray();
    // Use both 'from' and 'to' to determine the pivot candidates.
    var increment = 200 + ((to - from) & 15);//每隔 200~215 个元素挑出一个元素
    var j = 0;
    from += 1;
    to -= 1;
    for (var i = from; i < to; i += increment) {
      t_array[j] = [i, a[i]];
      j++;
    }
    t_array.sort(function(a, b) {
      return comparefn(a[1], b[1]);
    });
    var third_index = t_array[t_array.length >> 1][0];
    return third_index;
  }

    
    //快速排序
  function QuickSort(a, from, to) {
    var third_index = 0;
    while (true) {
      // Insertion sort is faster for short arrays.
      if (to - from <= 10) {//当 n<=10 时，采用插入排序
        InsertionSort(a, from, to);
        return;
      }
      if (to - from > 1000) {//当n>1000时，获取哨兵元素
        third_index = GetThirdIndex(a, from, to);
      } else {//当10<n<1000时，取中位数作为哨兵元素
        third_index = from + ((to - from) >> 1);
      }
      // Find a pivot as the median of first, last and middle element.	
      var v0 = a[from];
      var v1 = a[to - 1];
      var v2 = a[third_index];
      var c01 = comparefn(v0, v1);
      if (c01 > 0) {
        // v1 < v0, so swap them.
        var tmp = v0;
        v0 = v1;
        v1 = tmp;
      } // v0 <= v1.
      var c02 = comparefn(v0, v2);
      if (c02 >= 0) {
        // v2 <= v0 <= v1.
        var tmp = v0;
        v0 = v2;
        v2 = v1;
        v1 = tmp;
      } else {
        // v0 <= v1 && v0 < v2
        var c12 = comparefn(v1, v2);
        if (c12 > 0) {
          // v0 <= v2 < v1
          var tmp = v1;
          v1 = v2;
          v2 = tmp;
        }
      }
      // v0 <= v1 <= v2
      a[from] = v0;
      a[to - 1] = v2;
      var pivot = v1;
      var low_end = from + 1;   // Upper bound of elements lower than pivot.
      var high_start = to - 1;  // Lower bound of elements greater than pivot.
      a[third_index] = a[low_end];
      a[low_end] = pivot;

      // From low_end to i are elements equal to pivot.
      // From i to high_start are elements that haven't been compared yet.
      partition: for (var i = low_end + 1; i < high_start; i++) {
        var element = a[i];
        var order = comparefn(element, pivot);
        if (order < 0) {
          a[i] = a[low_end];
          a[low_end] = element;
          low_end++;
        } else if (order > 0) {
          do {
            high_start--;
            if (high_start == i) break partition;
            var top_elem = a[high_start];
            order = comparefn(top_elem, pivot);
          } while (order > 0);
          a[i] = a[high_start];
          a[high_start] = element;
          if (order < 0) {
            element = a[i];
            a[i] = a[low_end];
            a[low_end] = element;
            low_end++;
          }
        }
      }
        //递归调用快排方法
      if (to - high_start < low_end - from) {
        QuickSort(a, high_start, to);
        to = low_end;
      } else {
        QuickSort(a, from, low_end);
        from = high_start;
      }
    }
  };
```

通过对源码的大致理解，我们可以知道，对于需要进行排序的数组元素个数n，大概有以下几种情况：

- 当 n<=10 时，采用`插入排序`；
- 当 n>10 时，采用`三路快速排序`；
- 10<n <=1000，采用中位数作为哨兵元素；
- n>1000，每隔 200~215 个元素挑出一个元素，放到一个新数组中，然后对它排序，找到中间位置的数，以此作为中位数。

## sort()的应用场景

由于sort可以根据字符串或者数值进行排序的特性，在应用中也是大显身手，下面简单列举几个sort函数的应用场景。

### 根据唯一数字标识进行排序(id ,key等)

```js
 const userList = [
        { id: 5, name: "" },
        { id: 3, name: "" },
        { id: 2, name: "" },
        { id: 4, name: "" },
        { id: 1, name: "" }
    ]
    let list = userList.sort((a, b) => a.id - b.id)
    console.log(list);
//[{id: 1, name: ''},{id: 2, name: ''},{id: 3, name: ''},{id: 4, name: ''},{id: 5, name: ''},]
//是否降序升序可根据返回值调整
```

### 根据时间进行排序

```js
 const list = [
        { id: 2, date: '2004-1-20 12:00:00', },
        { id: 1, date: '2000-1-20 12:00:00', },
        { id: 4, date: '2003-1-20 12:00:00', },
        { id: 3, date: '2001-1-20 12:00:00', },
        { id: 5, date: '2008-1-20 12:00:00', }
    ]
    let dataList = list.sort((a, b) => Date.parse(a.date) - Date.parse(b.date))
    console.log(dataList);
    // [
    //     { id: 1, date: '2000-1-20 12:00:00', },
    //     { id: 3, date: '2001-1-20 12:00:00', },
    //     { id: 4, date: '2003-1-20 12:00:00', },
    //     { id: 2, date: '2004-1-20 12:00:00', },
    //     { id: 5, date: '2008-1-20 12:00:00', }
    // ]
	//是否降序升序可根据返回值调整
```

### 根据中文排序（常见于将多个人物名称按照拼音顺序进行排序）

```js
		const list = [
            { id: 2, name: "李华" },
            { id: 1, name: "小明" },
            { id: 4, name: "刘豪" },
            { id: 3, name: "秦秦" },
            { id: 5, name: "张张" }
        ]
        let dataList = list.sort((a, b) => a.name.localeCompare(b.name))
        console.log(dataList);
  		// [{ id: 2, name: '李华' }
        // { id: 4, name: '刘豪' }
        // { id: 3, name: '秦秦' }
        // { id: 1, name: '小明' }
        // { id: 5, name: '张张' }]
		//是否降序升序可根据返回值调整
```

**以及在对数组进行的一系列操作中使用**