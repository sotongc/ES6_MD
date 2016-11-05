#Generator 函数

1. 简介
2. next方法的参数
3. for...of循环
4. Generator.prototype.throw()
5. Generator.prototype.return()
6. yield*语句
7. 作为对象属性的Generator函数
8. Generator函数的this
9. 含义
10. 应用

## 1. 简介

Generator函数是ES6提供的一种异步编程解决方案，语法行为与传统函数完全不同

Generator函数有多重理解角度。从语法上，可以理解成是一个状态机，封装了多个内部状态。

执行Generator函数会返回一个遍历器对象，也就是说，Generator函数除了状态机，还是一个遍历器对象生成函数。返回的遍历器对象可以一次遍历Generator函数内部的每一个状态。

Generator函数的特征：

1. `function`关键字与函数名之间有个`*`符号
2. 函数体内部使用`yield`语句，定义不同的内部状态

```javascript
function* helloWorldGenerator(){
    yield 'hello';
    yield 'world';
    return 'ending';
}

var hw=helloWorldGenerator();
```

上面的函数有 hello world 和 ending 三个状态

Genrator函数的调用方法与普通函数相同，不同的是：

- 调用后，函数并不执行也不反回预算结构
- 反回一个指向内部状态的指针兑现个，也就是Iterator对象
- 每次调用`next`就会从头或者上次停止的地方执行直到下一个`yield`或者`return`出现。

ES6 中一下写法都能过

```javascript
function * foo(){}

function *foo(){}

function* foo(){}

function*foo(){}
```

建议采用第三种方法。

###`yield`语句

遍历器对象的`next`方法的运行逻辑如下。

1. 遇到`yield`语句，就暂停执行后面的操作，并将紧跟在`yield`后面的那个表达式的值，作为返回的对象`value`属性值。
2. 下一次调用`next`方法时继续执行，直到遇到下一个`yield`或者`return`并将`return`后面表达式的值作为返回对象的`value`属性值 
3. 若没有`return`返回的值为`undefined`。

Generator函数可以不用`yield`语句，这时就变成了一个单纯的暂缓执行函数。

>yield 不能用在普通函数中，会报错。

###与Iterator接口的关系

任意对象的Symbol.iterator方法，等于该对象的遍历器生成函数，调用该函数会返回该对象的一个遍历器对象。


