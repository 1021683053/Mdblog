# JavaScript语言精粹

只是概括本人认为的重要或者容易被遗忘的只是点，如果有条件请购买原版书籍进行阅读。

***

**1.块级作用于的危险性**
```javascript
/*
  var md = /a*/.metch(s);
*/
```
因为正则原因无法注释正则后面的内容。

**2.typeof 所有的运算结果**
```javascript
  'number' 'string' 'boolean' 'undefined' 'function' 'object'
```
_(注：以上所有结果都为字符串)_

**3.简单数据类型**

JavaScript简单数据类型有 `数字(Number)`,`字符串(String)`,`布尔值(Boolean)`,`null`,`undefined`

**4.对象的Key值**

`"first-name"`必须用引号括起来而 `first_name`不需要

**5.函数理解**
>函数是对象，所以他可以向任何其他的值一样被使用，函数可以保存变量、对象和数组中。函数可以被当做参数传递给其他的函数，函数也可以返回函数。而且，因为函数是对象，所以也可以拥有方法。

**6.函数的`this`,`arguments`与4中调用模式**

_(注：每个函数出了形参还会接收到两个参数 `this`&`arguments` )_  
1）方法调用模式  
2）函数调用模式  
3）构造器调用模式  
4）apply调用模式  
- 函数的方式调用时，`this`会绑定到全局作用域上面（包括在对象内部）`this`也指向全局

```javascript
"use strict"
// 此段代码只适用于 Browser
var scope = 'window';

var obj = {
	scope: "local",
	double: function(){
		var helper = function(){
			console.log(this.scope);
		}
		helper();
	}
}

obj.double();
//输出结果 window

//解决方案
var self = this; OR var that = this; //对象冒充
```
- 扩充类型功能

```javascript
"use strict"

//对函数对象原型进行扩充
Function.prototype.method = function(name, func){

    //自己对本身进行扩充
	this.prototype[name] = func;
	return this;
}

Function.method("liweifeng", function(){
	console.log("Liweifeng");
});

var test = function(){

}
test.liweifeng();
```
