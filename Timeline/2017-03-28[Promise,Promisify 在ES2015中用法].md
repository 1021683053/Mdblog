# yield, async/await, promsie, promisify

## 阮一峰的相关文章
- [Generator 函数的含义与用法](http://www.ruanyifeng.com/blog/2015/04/generator.html)
- [co 函数库的含义和用法](http://www.ruanyifeng.com/blog/2015/05/co.html)
- [Thunk 函数的含义和用法](http://www.ruanyifeng.com/blog/2015/05/thunk.html)
- [async 函数的含义和用法](http://www.ruanyifeng.com/blog/2015/05/async.html)

## GeneratorFunction(生成器函数)与 Generator对象(生成器)

** 参考文章 **
- [GeneratorFunction](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/function*)
- [Generator](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Generator)

** 官方描述 **
>生成器是一种可以从中退出并在之后重新进入的函数。生成器的环境（绑定的变量）会在每次执行后被保存，下次进入时可继续使用。

>调用一个生成器函数并不马上执行它的主体，而是返回一个这个生成器函数的迭代器（iterator）对象。当这个迭代器的next()方法被调用时，生成器函数的主体会被执行直至第一个yield表达式，该表达式定义了迭代器返回的值，或者，被 yield*委派至另一个生成器函数。next()方法返回一个对象，该对象有一个value属性，表示产出的值，和一个done属性，表示生成器是否已经产出了它最后的值。

官方描述很直白，基本功能和用法很简单，基本参考官方实例就可以写出对应的内容
