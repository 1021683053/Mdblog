# JavaScript正则表达式
网络部分借鉴只用于个人理解或者以后查阅
本文参考地址

* [MDN JavaScript正则表达式](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Regular_Expressions)
* [精通 JavaScript中的正则表达式](http://www.iteye.com/topic/481228)

***

## 创建正则表达式
**构造函数创建**
```javascript
var reg = new RegExp(); //创建空的正则表达式
var reg = new RegExp('a'); //创建表达式匹配a
var reg = new RegExp('a','i'); //带修饰创建（不匹配大小写）
```
_注：构造函数第一个参数为正则表达式文本内容，第二个参数为二选标志，**标志可以组合使用**_

**正则表达式字面量创建**
```javascript
var reg = /a/gi;
```
_注：这个更常用，但是不能用做于未知的正则，比如，这个正则需要判断段，拼接，或者修饰符未知_

## JavaScript正则表达式函数
|函数名|使用方法|
|-----|-------
|`exec`|一个在字符串中执行查找匹配的RegExp方法，它返回一个数组（未匹配到则返回null）。
|`test`|一个在字符串中测试是否匹配的RegExp方法，它返回true或false。
|`match`|一个在字符串中执行查找匹配的String方法，它返回一个数组或者在未匹配到时返回null。
|`search`|一个在字符串中测试匹配的String方法，它返回匹配到的位置索引，或者在失败时返回-1。
|`replace`|一个在字符串中执行查找匹配的String方法，并且使用替换字符串替换掉匹配到的子字符串。
|`split`|一个使用正则表达式或者一个固定字符串分隔一个字符串，并将分隔后的子字符串存储到数组中的String方法。

>当你想要知道在一个字符串中的一个匹配是否被找到，你可以使用test或search方法；想得到更多的信息（但是比较慢）则可以使用exec或match方法。如果你使用exec或match方法并且匹配成功了，那么这些方法将返回一个数组并且更新相关的正则表达式对象的属性和预定义的正则表达式对象（详见下）。如果匹配失败，那么exec方法返回null（也就是false）。

## 特殊字符
特殊字符参照表
* [特殊字符参照表](../HTML/正则表达式中的特殊字符.md)
