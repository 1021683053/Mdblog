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

****

#### Generator使用方法

**1.执行次数**
```javascript
let gen = function * (){
    yield 1;
    yield 2;
};

let g = gen();
console.log( gen.next() );  //  { value: 1, done: false }
console.log( gen.next() );  //  { value: 1, done: false }
console.log( gen.next() );  //  { value: undefined, done: true }
```
如果函数里有 n 个 yield 至少执行 n+1 次 才能执行完毕

**2.执行yield返回值**
```javascript
let gen = function * (){
    var a = yield 1;
    var b = yield 2;
    console.log(a, b)
};

let g = gen();
console.log( gen.next() );  //=>  { value: 1, done: false }
console.log( gen.next() );  //=>  { value: 2, done: false }
                            //=>  undefind,undefind
console.log( gen.next() );  //=>  { value: undefined, done: true }
```
1. yield默认返回值为 undefined。
2. 返回值时机是 下一个 next() 开始执行，返回上一个 yield 值。

**3.执行 next(value)**
```javascript
let gen = function * (){
    // 这里是 第一个 next 执行的时机，包括定义一个 yield，但是不包括 赋值操作;
    var a = yield 1;
    // 第二个 next 执行时机，next(3) 中的‘3’被赋值于 上一个yield返回值，所以 a = 3;
    var b = yield 2;
    // 第三个 next 执行时机，next(4) 中的‘4’被赋值于 上一个yild的返回值，所以 b = 4;
    console.log(a, b)
};
let g = gen();
g.next();   
g.next(3);
g.next(4);
//=>  3,4
```
next(value)的传参被作为上一个 yield 的返回值。*当然第一个 next() 以上无 yield 所以传参无意义*

**4.执行 return(value)**
```javascript

let gen = function *(){
    var a = yield 1; //会被执行
    console.log(a);  //会被执行
    var b = yield 2; //会被执行但是 b 不会被赋值 因为下一步 是return() 执行到赋值前一完成，到此终止
    console.log(b);  //不会被执行 因为下一步是return() 后面的next()讲不在起作用
    var c = yield 3; //不会被执行
    console.log(c);  //不会被执行
}
let g = gen();
console.log( g.next() );    //=>    { value: 1, done: false }
console.log( g.next(6) );   //=>    { value: 2, done: false }
console.log( g.return(7) ); //=>    { value: 7, done: true }
console.log( g.next(8) );   //=>    { value: undefined, done: true }
```

#### co库实现
**1. co库的实现**
主要实现方式是 Promise 配合 generator.next(value) *next(value)后面会讲到*
```javascript
function run(gen){
    var g = gen();
    function next(data){
        var result = g.next(data);
        if (result.done) return result.value;
        result.value.then(function(data){
            next(data);
        });
    }
    next();
}
run(gen);
```
