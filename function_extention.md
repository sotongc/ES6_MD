#函数的扩展

1. 函数参数的默认值
2. rest参数
3. 扩展运算符
4. 严格模式
5. name属性
6. 箭头函数
7. 绑定 this
8. 尾调用优化
9. 函数参数的尾逗号

--------------------------------------

##1. 函数参数的默认值

在ES6之前，函数的参数默认值只能采用类型检测和函数内部参数初始化的方式。

ES6允许为函数的参数设置默认值，即直接写在参数定义的后面。

```javascript
function log(x,y='World'){
	console.log(x,y);
}

log('Hello')//Hello,World
log('Hello','China')//Hello,China
log('Hello','')//Hello
```
参数变量默认是声明的，所以不能用`let`和`const`在内部再次声明。

### 函数的length属性

指定了默认值后，函数的`length`属性，将返回米有指定默认值的参数个数。

```javascript
(function(a){}).length//1
(function(a=5){}).length//0
(function(a,b,c=5){}).length//2
```

因为`length`属性的含义是该函数预期传入的参数个数。某个参数指定默认值以后，预期传入的参数个数就不包括这个参数了。同理，reset参数也不会计入`length`属性。

```javascript
(function(a=0,b,c){}).length//0
(function(a,b=1,c){}).length//1
```
如果设置了默认值的参数不是尾参数，那么`length`属性也不再计入后面的参数了。

###作用域

如果参数默认值是一个变量，则该变量所处的作用域与其他变量的作用域相同，先当前函数作用域，然后才是全局作用域。


如果参数的默认值是一个函数，该函数的作用域是其声明时所在的作用域。

```javascript

let foo='outer';
function bar(func=x=>foo){
	let foo='inner';
	console.log(func());//outer
}
bar();
```

一个复杂的栗子

```javascript
var x=1;
function foo(x,y=function(){x=2;}){
	var x=3;
	y();
	console.log(x);
}
foo()//3

var x=1;
function foo(x,y=function(){x=2;}){
	x=3;
	y();
	console.log(x);
}
foo()//2
```
两个x不是一个作用域下的。参数默认函数可以讲变量的默认值设为异常抛出函数。

##2. rest参数

ES6引入rest参数（形式为 "...变量名"），用于获取函数的多余参数，这样就不用使用`arguments`对象了。

rest参数将多余的参数放入数组中。

```javascript
function add(...values){
	let sum=0;
	for(var val of values){
		sum+=val;
	}
	return sum;
}
add(2,5,3) //10
```
下面是一个rest参数代替arguments变量的栗子

```javascript
function sortNum(){
	return Array.slice.call(arguments).sort();
}

//rest参数的写法
const sortNum=(...numbers)=>numbers.sort();
```

>Note: rest 参数之后不能再有其他参数，否则会报错。 函数的length属性不包括rest参数。

##3. 扩展运算符

**Def:** 扩展运算符(...) 好比rest参数的逆运算，将一个数组转为用逗号分隔的参数序列。

该运算符主要用于函数调用。

```javascript
function push(array, ...items){
	array.push(...items);
	//or 
	array.push.apply(array,items);
}

//等价的两种找最大值的写法
//目的都是调用 Math.max(1,2,3)这个接口
Math.max(...[1,2,3]);
Math.max.apply(Math,[1,2,3]);

function add(x,y){
	return x+y;
}

var numbers=[4,38];
add(...numbers)
```

###扩展运算符的应用

####1. 合并数组

```javascript
//ES5
arr1.concat(arr2,arr3);

//ES6
[...arr1,...arr2,...arr3];
```
####2. 与解构赋值结合

```javascript
const [first,...rest]=[1,2,3,4,5];
```
####3. 函数的返回值
####4.字符串
```javascript
[...'hello'];
```

####5. 实现了Iterator接口的对象

任何Iterator接口的对象，都可以用扩展原酸符转为真正的数组。

```javascript
var nodeList=document.querySelectorAll('div');
var array=[...nodeList];
```

###4. 严格模式

ES6规定，只要函数参数使用了默认值，解构赋值，或者扩展运算符，那么函数内部就不能显式设定为严格模式，否则会报错。

原因在于参数先执行，但严格模式的定义在函数体内部。为了规避麻烦，索性直接如此定义。

###5. name属性

函数的`name`属性，返回该函数的函数名。

```javascript
function foo(){}
foo.name //"foo"
```
如果将一个匿名函数赋值给一个变量，ES5中`name`会返回空字符串，ES6会返回实际函数名。

`Function`构造函数返回的函数实例，`name`属性的值为"anonymous"。

```javascript
(new Function).name //"anonymous"

function foo(){}
foo.bind({}).name //"bound foo"

(function(){}).bind({}).name //"bound "
```

`bind`返回的函数，`name`属性值会加上"bound"前缀

##6. 箭头函数

ES6允许使用 `=>`定义函数。

```javascript
var f=v=>v;

var f=function(v){
	return v;
};
```

如果箭头函数不需要参数或者需要多个参数，就使用一个圆括号代表参数部分。

```javascript
var f=()=>5;
//equals
var f=function(){return 5};

var sum=(num1,num2)=>num1+num2;
//equals
var sum=function(num1,num2){
	return num1+num2;
};
```

如果箭头函数的代码块超过一条语句，需要将它们括起来

```javascript
var sum=(num1,num2)=>{return num1+num2;}

var getTempItem=id=>({id:id,name:'temp'});
```

和变量解构结合使用。

```javascript
const full=({first,last})=>first+' '+last;

// 等同于
function full(person){
	return person.first+' 'person.last;
}
```

箭头函数简化了语法，方便定义工具函数和回调函数

###使用注意点

1. 函数体内的`this`对象，就是定义时所在的对象，而不是使用时所在的对象。
2. 不可以当做构造函数。
3. 不可以使用`arguments`对象。可用rest参数代替
4. 不可以使用`yield`命令，因此箭头函数不能用作Generator函数。

`this`对象的指向是可变的，但是在箭头函数中，它是固定的。

```javascript
function foo(){
	setTimeout(()=>{
		console.log('id:',this.id);
	},100);
}

var id=21;

foo.call({id:42});
//id:42
```

 箭头函数导致`this`总是指向函数定义生效时所在的对象，而非执行时所在的对象。此时箭头函数的生成时间是在foo函数执行时。所以`this`指向 `{id:42}`

 箭头函数可以让`setTimeout`里面的`this`,绑定定义时所在的作用域，而不是指向运行时所在的作用域。

 ```javascript
 function Timer(){
 	this.s1=0;
 	this.s2=0;
 	//箭头函数
 	setInterval(()=>this.s1++,1000);
 	//普通函数
 	setInterval(function(){
 		this.s2++;
 	},1000);
 }

 var timer=new Timer();

 setTimeout(()=>console.log('s1: ',timer.s1),3100);
 setTimeout(()=>console.log('s2: ',timer.s2),3100);
 //s1:3
 //s2:0
 ```

上面两种绑定方式，箭头函数的`this`指向定义时的`timer`对象；而普通函数的`this`指向运行时的`window`对象。所以普通函数的循环累加是针对`window.s12`；

箭头函数可以让`this`指向固定化，这种特性有利于封装回调函数。

```javascript
var handler={
	id:'123456',
	init:function(){
		document.addEventListener('click',event=>this.doSomething(event.type),false);
	},
	doSomething:function(type){
		console.log(this.id);
	}
}
```

若果不用箭头函数固定`this`为`handler`对象，那么`this`的指向将成为执行环境中的dom对象。

>`this`指向固定化，并不是因为箭头函数中由内部绑定`this`的机制，实际原因是箭头函数根本没有自己的`this`,导致内部的`this`就是外层代码块的this.正因为它没有`this`所以不能用于构造函数。


>除了`this`,以下三个变量在箭头函数之中也不存在，指向外层函数的对应变量：`arguments`,`super`, `new.target`。

####嵌套的箭头函数

箭头函数的内部可以再使用箭头函数。下面是一个部署管道机制(pipeline)的例子，即前一个函数的输出是后一个函数的输入。

```javascript
const pipeline=(...funcs)=>
	val=>functions.reduce((a,b)=>b(a),val);
```

##7. 绑定 this （ES7 提案）

##8. 尾调用优化

**Def: **【尾调用】-就是指某个函数的最后一步是调用另一个函数。

Simple Case:

```javasript
function f(x){
	return g(x);
}
```

以下三种情况，都不属于尾调用。

```javascript
//case 1
function f(x){
	let y=g(x);
}

//case 2
function f(x){
	return g(x)+1;
}

//case 3
function f(x){
	g(x);
}
```

情况一，二调用后都有别的操作，情况三等同于`return undefined`。尾调用不一定出现在函数尾部，只要是最后一步操作即可。（例如 if 控制）

####尾调用优化

尾调用之所以与其他调用不同，就在于它的特殊的调用位置。

我们知道，函数调用会在内存形成一个“调用记录”，又称“调用帧”，保存调用位置和内部变量等信息。

>*Note:* 如果在函数A的内部调用函数B，那么在A的调用帧上方，还会形成一个B的调用帧。等到B运行结束，将结果返回到A，B的调用帧才会消失。如果函数B还调用函数C，那就还有一个C的调用，以此类推，所有调用帧会形成一个调用栈。


尾调用由于是函数的最后一步操作，所以不需要保留外层函数的调用帧，**因为调用位置，内部变量等信息不会再用到**，只要直接用内层函数的调用帧渠道外层函数的调用帧就可以了。

```javascript
functon f(){
	let m=1,n=2;
	return g(m+n);
}
f();

//尾调用，不再保留调用帧所以等同于
function f(){
	return g(3);
}

//等同于
g(3)；
```

上面的栗子就叫做“尾调用优化”。

**Def:**【尾调用优化】-对于尾调用函数只保留内层函数的调用帧，这样可以大大节省内存。

>**Note: **只有不再用到外层函数的内部变量，内层函数的调用帧才会取代外层函数的调用帧，否则就无法进行 “尾调用优化”。

```javascript
function addOne(a){
	var one=1;
	function inner(b){
		return b+one;
	}
	return inner(a);
}
```

上面的函数不会进行尾调用优化，因为内层函数 `inner`用到了外层函数`addOne`的内部变量`one`。如此，大多数闭包函数均不会触发尾调用优化。

####尾递归

函数调用自身，称为递归。如果尾调用自身，就称为尾递归。

递归非常耗内存，因为需要保存大量的调用帧，很容易发生 "栈溢出"。但对于尾递归来说，由于只存在一个调用帧，所以永远不会发生 "栈溢出" 错误。

下面的栗子：

**1. 阶乘计算**

```javascript
//递归
function factorial(n){
	if(n===1) return 1;
	return n*factorial(n-1);
}

//尾递归--把所有函数中的变量在尾调用中都利用起来（传参）
function factorial(n,total){
	if(n===1) return total;
	return factorial(n-1,n*total);
}
factorial(5,1)
```

**2. 斐波那契数列**

```javascript
//递归
function Fibonacci(n){
	if(n<=1){return 1};
	return Fibonacci(n-1)+Fibonacci(n-2);
}

//尾递归
function Fibonacci(n,ac1=1,ac2=1){
	n<=1 && (return ac2);
	return Fibonacci(n-1,ac2,ac1+ac2);
}
```


#### 递归函数的改写

>*Note:* 尾递归的实现，往往需要改写递归函数，确保最后一步只调用自身。

做法是： 把所有用到的内部变量改写成函数的参数。

函数式编程有一个概念就啊哦做柯里化，意思是将多参数的函数转换成单参数的形式。

**Def:** 【函数柯里化】-将多参数的函数转换成单参数的形式。

```javascript

function currying(fn,n){
	return function(m){
		return fn.call(this,m,n);
	}
}

function tailFactorial(n,total){
	if(n===1) return total;
	return tailFactorial(n-1,n*total);
}

const factorial=currying(tailFactorial,1);

//ES6
function factorial(n,total=1){
	if(n===1) return total;
	return factorial(n-1,n*total);
}
```

>**Summary: **递归本质上是一种循环操作，纯粹的函数式编程语言没有循环操作，所以要通过递归来实现。这也是尾递归对这些语言重要的原因。

####严格模式

ES6的尾调用优化只在严格模式下开启，正常模式是无效的，这是因为正常模式下，函数内部有两个变量用于跟踪调用栈

- `func.arguments`:返回调用时函数的参数。
- `func.caller`: 返回调用当前函数的那个函数。

尾调用发生时，函数的调用栈会改写，所以这两个变量会失真，这是为何尾函数调用只在严格模式下生效。


