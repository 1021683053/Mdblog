# JavaScript Array常用操作以及容易被遗忘的操作，与排序部分排序算法

本文章列举的属性与特性，大部分属于`ES5`属性，`ES6/7` 的特性有更好的解决方案，由于前一段时间面试大多数公司的面试题
都有包含，Array 方法、排序、算法 等问题，有些知识长时间不用很容易被遗忘，也用来对面试前复习与平时工作记忆性查找。

***

### 文章参考
[MDN JavaScript 标准库 Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)

### `Array` 常用实例（所有数组实例继承自Array.prototype。Array构造函数的原型对象是可修改的，其会影响所有的数组实例）

- 属性
    - **Array.prototype.constructor** 构造函数
    - **Array.prototype.length** 数组长度
- `Mutator` 赋值函数 (会改变自身值)
    - **pop()** 删除数组的最后一个元素，并返回这个元素。
    - **push()** 在数组的末尾增加一个或多个元素，并返回数组的新长度。
    - **reverse()** 颠倒数组中元素的排列顺序，即原先的第一个变为最后一个，原先的最后一个变为第一个。
    - **shift()** 删除数组的第一个元素，并返回这个元素。
    - **sort()** 对数组元素进行排序，并返回当前数组
    - **splice()** 在任意的位置给数组添加或删除任意个元素。
    - **unshift()** 在数组的开头增加一个或多个元素，并返回数组的新长度。
- `Accessor` 访问函数 （不会改变自身值）
    - **concat()** 返回一个由当前数组和其它若干个数组或者若干个非数组值组合而成的新数组。
    - **join()** 连接所有数组元素组成一个字符串。
    - **slice()** 抽取当前数组中的一段元素组合成一个新数组。
    - **toString()** 返回一个由所有数组元素组合而成的字符串。
    - **toLocaleString()** 返回一个由所有数组元素组合而成的本地化后的字符串。
    - **indexOf()** 返回数组中第一个与指定值相等的元素的索引，如果找不到这样的元素，则返回-1。
    - **lastIndexOf()** 返回数组中最后一个（从右边数第一个）与指定值相等的元素的索引，如果找不到这样的元素，则返回-1。
- `Iteration` 迭代器函数
    - **forEach()** 为数组中的每个元素执行一次回调函数。
    - **every()** 如果数组中的每个元素都满足测试函数，则返回 true，否则返回 false。
    - **some()** 如果数组中至少有一个元素满足测试函数，则返回 true，否则返回 false。
    - **filter()** 将所有在过滤函数中返回 true 的数组元素放进一个新数组中并返回。
    - **map()** 返回一个由回调函数的返回值组成的新数组。
    - **reduce()** 从左到右为每个数组元素执行一次回调函数，并把上次回调函数的返回值放在一个暂存器中传给下次回调函数，并返回最后一次回调函数的返回值。
    - **reduceRight()** 从右到左为每个数组元素执行一次回调函数，并把上次回调函数的返回值放在一个暂存器中传给下次回调函数，并返回最后一次回调函数的返回值。
    - **values()** 返回一个数组迭代器对象，该迭代器会包含所有数组元素的值。

*具体用法请参考原文 本文的所有函数归属于ES5，兼容IE9+*

### 常用的 Array技巧

**1. 在第N个位置添加一个值**
```javascript
var arr = [1,2,3,4];
arr.splice(1, 0, 'This is a new value!');
console.log(arr);
//=> [1,'This is a new value!', 2, 3, 4];
```

**2. Array的复制**
```javascript
//1. 方法 slice
var arr = [1,2,3];
var newarr = arr.slice(0);

//2. 方法 concat
var arr = [1,2,3,4];
var newarr = arr.concat();
```
以上两个方法的缺点很明显，数组的复制只能停留在第一层，如果一个值为Object，复制的属性，Object会被引用。

```javascript
//3. JSON.parse(JSON.stringify(  ))
var arr = [{a:1},2,3,4];
var newarr = JSON.parse(JSON.stringify( arr ));
var newarr[0].a = 2;
console.log(arr[0].a);
//=> 1
```

```javascript
//4. 对象的克隆，例如 underscore 的 _.clone(),_.extend() jQuery 的 $.extend()。
```

### 排序、去重问题

**1. 去重问题**

正常去重，完整去掉重复的元素
```javascript
//1. 使用排序，然后循环判断当前值是否为上一个元素相等
var arr = [1,33,44,33,22,33,4455,44];
arr.sort();
var newarr = [arr[0]];
for( var i = 1; i<arr.length; i++ ){
    if( arr[i] !== newarr[newarr.length-1] ){
        newarr.push(arr[i]);
    }
}
console.log(newarr);
//=> [1, 22, 33, 44, 4455]
```
```javascript
//2. 使用indexOf() 方法判断是否在已有对象内
var arr = [1,33,44,33,22,33,4455,44];
var newarr = [];
for( var i = 0; i<arr.length; i++ ){
    if( newarr.indexOf( arr[i] ) < 0 ){
        newarr.push(arr[i]);
    }
}
console.log(newarr);
//=> [1, 22, 33, 44, 4455]
```
去重，去掉包含某个元素的内容的其中一个
```javascript
//3. 只保留aa，bb，cc开始的任意一个
var arr = ['aa1234567','bb1234567','aa333222','bb3431233','cc123321','bb123321'];
var newarr = arr.sort();
var newarr = [arr[0]];
for( var i = 1; i<arr.length; i++ ){
    if( newarr[newarr.length-1].indexOf(arr[i].slice(0,2)) ){
        newarr.push(arr[i]);
    }
}
console.log(newarr);
//=> ["aa1234567", "bb123321", "cc123321"]
```
**2. 排序问题**
sort() 默认是先对比第一个元素，在对比第二个元素 的升序排列，也就是 `[11,2,22,111]` 排序结果为 `[11,111,2,22]`
```javascript
// 1. 基础排序方法
var arr = [1,23,4,2,1,3];
arr.sort(function(a, b){
    return a-b;
})
// 理解：a，b   a(对比数据的前者), b(对比数据的后者) return true(替换位置)，false(不替换位置)
```

**3. 归纳函数**
```javascript
//去重复
Array.prototype.unique = function(){
    this.sort();
    var re=[this[0]];
    for(var i = 1; i < this.length; i++)
    {
        if( this[i] !== re[re.length-1])
        {
            re.push(this[i]);
        }
    }
    return re;
}

//并集
Array.prototype.union = function(a){
    return this.concat(a).unique();
}

//差集
Array.prototype.minus = function(a){
    var result =[];
    var clone = this;
    for(var i=0; i < clone.length; i++){
        var flag = true;
        for(var j=0; j < a.length; j++){   
            if(clone[i] == a[j])   
            flag = false;   
        }   
        if(flag)   
        result.push(clone[i]);
    }
    return result.unique();
}

// 交集
Array.prototype.intersect = function(b) {
    var result = [];
    var a = this;
    for(var i = 0; i < b.length; i ++) {
        var temp = b[i];
        for(var j = 0; j < a.length; j ++) {
            if(temp === a[j]) {
                result.push(temp);
                break;
            }
        }
    }
    return result.unique();
}
```
