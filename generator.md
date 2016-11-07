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

由于Generator函数就是遍历器生成函数，因此可以把Generator赋值给对象的`Symbol.iterator`属性，从而使得该对象具有Iterator接口

```javascript
var myIterable={};
myIterable[Symbol.iterator]=function* (){
    yield 1;
    yield 2;
    yield 3;
};

[...myIterable]// [1,2,3]
```

上面代码中，Generator函数赋值给`Symbol.iterator`属性，由于Generator返回一个遍历器对象，从而使得`myIterable`对象具有了Iterator接口，可以被`...`运算符遍历。

```javascript
function* gen(){}

var g=gen();

g[Symbol.iterator]()===g
```

##2. next方法的参数

`yield`本身没有返回值，`next`方法可以带一个参数，该参数就会被当做上一个`yield`的返回值。

```javascript

function* f(){
    for(var i=0;true;i++){
        var reset=yield i;
        if(reset) {i=-1;}
    }
}

var g=f();
g.next() //{value:0,done:false}
g.next() //{value:1,done:false}
g.next(true) //{value:0,done:false}
```

如果`next`方法没有参数，每次运行`yield`语句，变量`reset`的值总是`undefined`(因为yield在没参数的时候不返回值，但调用`next`的时候返回`yield`表达式后的值)。当`next`方法带一个参数`true`时，当前的变量`reset`就被重置为这个参数，因此`i`会等于-1。

这个功能有重要的语法意义：Generator函数从暂停状态到恢复运行，它的上下文状态是不变的。通过`next`方法的参数，就有办法在Generator函数开始运行之后继续向函数体内部注入参数从而调整函数行为。

##3. for...of循环

`for...of`循环可以自动遍历Generator函数时生成的`Iterator`对象，且此时不再需要调用`next`方法。

```javascript
function* foo(){
    yield 1;
    yield 2;
    return 3;
}

for(let v of foo()){ // foo() 返回一个Iterator对象
    console.log(v);
}
// 1 2 
```

>**Note: **一旦`next`方法的返回对象的`done`属性为`true`，`for...of`循环就会终止，且不包含返回对象，所以上面的`return`语句不包括在循环之中。

##4. Generator.prototype.throw()

Generator 函数返回的遍历器对象，都有一个`throw`方法，可以在函数体外抛出错误，然后在Generator函数体内捕获。

```javascript
var g=function* (){
    try{
        yield;
    } catch (e){
        console.log('内部捕获',e);
    }
};

var i=g();
i.next();

try{
    i.throw('a');
    i.throw('b');
} catch (e){
    console.log('外部捕获',e);
}

//内部捕获 a
//外部捕获 b
```

遇到错误先由内部捕获，第二次抛出的异常因为内部捕获已经执行过，所以由外部捕获。`throw`方法可以接受一个参数，该参数会被`catch`语句接收，建议抛出`Error`对象的实例。

>**Note: **不要混淆遍历器对象的`throw`方法和全局的`throw`命令。上面代码的错误，是用遍历器对象的`throw`方法抛出的，而不是用`throw`命令抛出的。throw命令抛出的只能被函数体外的`catch`捕获。而`throw`方法抛出的错误可以被函数体内部外部的`catch`捕获。

`throw`方法被捕获以后，会附带执行下一条`yield`语句。`throw`命令和`throw`方法无关相互不影响。因而`throw`命令不会影响Generator函数遍历器的状态。


>**Note: **一旦Generator执行过程中抛出错误，且没有被内部捕获，就不会再执行下去了。如果以后还调用`next`方法，将返回一个`{value:undefined,done:true}`对象，JS引擎认为这个Generator已经结束了。


##5. Generator.prototype.return()

Generator函数返回的遍历器对象，还有一个`return`方法，可以返回给定的值，并且终结遍历Generator函数。

```javascript
function* gen(){
    yield 1;
    yield 2;
    yield 3;
}

var g=gen();

g.next(); // {value:1,done:false}
g.return('foo'); // {value:'foo',done:true}
g.next(); // {value:undefined,done:true}
```

如果`return`调用，则返回`return`方法传入的参数作为`value`，并改状态`done`为`true`;
如果Generator函数内部有`try...finally`代码块，那么`return`方法会推迟到`finally`代码块执行完再执行。

```javascript
function* numbers(){
    yield 1;
    try{
        yield 2;
        yield 3;
    } finally {
        yield 4;
        yield 5;
    }
    yield 6;
}

var g=numbers();

g.next() //{done:false,value:1}
g.next() //{done:false,value:2}
g.return(7) //{done:false,value:4}
g.next()// {done:false,value:5}
g.next()// {done:true,value:7}
```

##6. yield* 语句

如果在Generator函数内部，调用另一个Generator函数，默认情况下是没有效果的。

```javascript

function* foo(){
    yield 'a';
    yield 'b';
}

function* bar(){
    yield 'x';
    foo();
    yield 'y'
}

for(let v of bar()){
    console.log(v);
}
//x
//y
```

这个时候就需要用到`yield*`语句在一个Generator函数里执行另一个Generator函数。

```javascript
function* bar(){
    yield 'x';
    yield* foo();
    yield 'y';
}
```

从语法角度看，如果`yield`命令后面跟的是一个遍历器对象，需要在`yield`命令后面加上星号，表明它返回的是一个遍历器对象。这被称为`yield*`语句。

>**实际上，任何数据结构只要有Iterator接口，就可以被`yield*`遍历。

下面试一个稍微复杂的栗子，使用`yield*`语句遍历完全二叉树。

```javascript
//二叉树构造函数
function Tree(left,label,right){
    this.left=left;
    this.label=label;
    this.right=right;
}

//下面是中序遍历函数
//由于返回的是一个遍历器，所以要用generator函数。
//函数体内采用递归算法，所以左树和右树要用`yield*`遍历
function* inorder(t){
    if(t){
        yield* inorder(t.left);
        yield t.label;
        yield* inorder(t.right);
    }
}

//下面生成二叉树
function make(array){
    //判断是否为叶节点
    if(array.length==1) return new Tree(null,array[0],nul);
    return new Tree(make(array[0]),array[1],make(array[2]));
}

let tree = make([[['a'], 'b', ['c']], 'd', [['e'], 'f', ['g']]]);

//遍历二叉树
var result=[];
for(let node of inorder(tree)){
    result.push(node);
}

result // ['a','b','c','d','e','f','g'];
```

##8. Generator函数的`this`

Generator函数总是返回一个遍历器，ES6规定这个遍历器是Generator函数的实例，也继承了Generator函数的`prototype`。

```javascript
function* g(){}

g.prototype.hello=function(){
    return 'hi!';
};

let obj=g();

obj instanceof g //true
obj.hello() // 'hi!'
```

如果把g当做普通的构造函数，并不会生效，因为Generator函数返回的是遍历器对象而不是`this`对象。

```javascript
function* g(){
    this.a=11;
}

let obj=g();
obj.a //undefined

new g() //报错
```

改造成一致构造函数的小技巧：

```javascript

function* gen(){
    this.a=1;
    yield this.b=2
    yield this.c=3;
}

function F(){
    return gen.call(gen.prototype);
}

var f=new F();

f.next();//{value:2,done:false}
f.next();//{value:3,done:false}
f.next();//{value:undefine,done:true}
f.a//1
f.b//2
f.c//3
```

##9. 含义

###Generator与状态机

Generator是实现状态机的最佳结构。如下栗子就是一个状态机：

```javascript
var ticking=true;
var clock=function(){
    if(ticking){
        console.log('Tick!');
    }else{
        console.log('Tock!');
    }
    ticking=!ticking;
}

//ES6
var colck=function*　(){
    console.log(1);
    yield;
    console.log(2);
    yield;
};
```

##10. 应用

Generator可以暂停函数执行，返回任意表达式的值。这种特点使得Generator有多种应用场景。

1. 异步操作的同步表达化
2. 控制流管理
3. 部署Iterator接口
4. 作为数据结构

