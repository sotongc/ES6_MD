#let和const命令

>1. let命令
2. 块级别作用域
3. const命令
4. 顶层对象属性
5. 顶层对象.



##let命令

ES6新增了`let`命令,用于来声明变量。它的用法类似`var`但可以创建块级作用域。

```javascript
{
	let a=10;
	var b=1;
}

a // ReferenceError: a is not defined.
b // 1
```

`for`循环的计数器，就很适合使用`let`。

```javascript
for(let i=0;i<10;i++){}
```
不同于`var`变量声明会污染全局作用域，`let`只在块级作用域中起效。下面的两个例子用来说明这个问题;

```javascript
var a=[];
for(var i=0;i<10;i++){
	a[i]=function(){
		console.log(i);
	};
}
a[6](); //10
```
若果使用`var`声明`console.log(i)`在实际执行的时候，循环已结束，所以`i=6`;

```javascript 
var a=[];
for(let i=0;i<10;i++){
	a[i]=function(){
		console.log(i);
	};
}
a[6](); //6
```
使用`let`声明因为块级作用域的存在就不存在这个问题。 同闭包有着类似的效果。

>Note: `let`不像`var`那样有着变量提升的现象。所以变量一定要在声明后使用否则会报错;

```javascript
console.log(foo) // undefined
console.log(bar) // ReferenceError

var foo=2;
let bar=2;

```

###暂时性死区

只要块级作用域内存在`let`，它所声明的变量就绑定在这个区域，不再受外部的影响。

```javascript

var tmp=123;

if(true){
	tmp='abc'; // RferenceError
	let tmp;
}
```
同作用域链的查找顺序，块级作用域为最优先位置。

>暂时性死区：ES6明确规定，如果区块中存在`let`和`const`命令，这个区块对这些命令声明的变量，从一开始就形成了封闭性作用域。


>ES6规定暂时性死区不出现变量提升，是为了减少运行时的错误，防止在变量声明前就使用这个变量。

下面的栗子是一个比较隐蔽的死区

```javascript
function bar(x=y,y=2){
	return [x,y];
}

bar(); //报错，x=y 触发了一个死区
```

###不允许重复声明

`let`不允许在相同作用域内，重复声明同一个变量。

###块级作用域

没有块级作用域带来的不合理：

- 内层变量覆盖外层变量（变量提升导致的）
- 用于计数的变量泄漏

块级作用域允许任意嵌套，外层作用域无法访问内层作用域变量，反之可行。块级作用域的出现，实际上使得广泛应用的立即执行匿名函数不再必要。

>块级作用域允许声明函数的规则，只在使用大括号的情况下成立，如果没有使用大括号，就会报错。

###`const`关键字

`const`声明一个只读的常量。一旦声明，常量的值就不能改变。

```javascript
const PI=3.141592654;
PI //

PI=3;// TypeError: Assignment to constant variable.
```

`const`声明的变量不得改值，意味着，`const`一旦声明变量，就必须立即初始化，不能留到以后赋值。和`let`相同，`const`有块级作用域，同样存在暂时性死区，不可重复声明。

```javascript 
const foo={};
foo.prop=123;

foo.prop // 123

foo={}; // TypeError: "foo" is read-only
```

上面`foo`常量存储的是一个指向内存的地址，这意味着该常量不可更改地址，但是地址指向的对象本身是可变的。

如果真想将对象冻结，应该使用`Object.freeze`方法。

```javascript
const foo=Object.freeze({});

//常规模式下，下面一行不起效
//严格模式下，报错
foo.prop=123;
```

###顶层对象的属性

ES5中，顶层对象和全局变量是等价的。这被认为是javasscript设计最大的败笔之一。首先在编译时没法检查变量未声明的错误。其次，容易意外的创建全局变量。最后，顶层对象的属性是到处可读写的，不利于模块化编程。


ES6为了改变这一点，一方面规定，为了保持兼容性，`var`和`function`声明的全局变量，依旧是顶层对象的属性；另一方面规定，`let`、`const`、`class`声明的全局变量，不属于顶层对象的属性。也就是说，从ES6开始，全局变量将逐步与顶层对象的属性脱钩。

```javascript
// Node 下： global.a
// 或 this.a
var a=1;

window.a //1

let b=1;
window.b //undefined
```

###顶层对象

ES5的顶层对象本身也是一个问题，源于它在各种实现里的不统一。

- 浏览器里面，顶层对象是`window`,但Node和Web Worker 没有`window`.
- 浏览器和Web Worker里面，`self`也指向顶层对象，但Node里面没有`self`
- Node里面，顶层对象是`global`，但其他环境都不支持。

同一段代码为了兼容各个环境，都能取到顶层对象，现在一般是使用`this`变量，但有局限性。


- 全局环境中，`this`会返回顶层对象。但是，Node和ES6中返回的是当前模块。
- 函数里面的`this`，如果函数不是作为对象的方法运行，而是单纯作为函数运行，`this`会指向顶层对象。但是，严格模式下，这时`this`会返回`undefined`。
- 不管是严格模式，还是普通模式`new Function('return this')()`，总是会返回全局对象。如果浏览器采用了CSP内容安全策略，那么`eval`、`new Function`这些方法都可能无法使用。

综上所述，很难找到一种方法，可以在所有情况下，都取到顶层对象。

现在有一个提案，引入一个`global`作为顶层对象独立于不同环境下。



