#变量的解构赋值

1. 数组的解构赋值
2. 对象的解构赋值
3. 字符串的解构赋值
4. 数值和布尔值的解构赋值
5. 函数参数的解构赋值
6. 圆括号问题
7. 用途

-----------------------------------

##1. 数组的解构赋值

def “解构赋值”：按照一定模式，从数组和对象中提取值，对变量进行赋值。

```javascript
var [a,b,c]=[1,2,3]; //a:1,b:2,c:3
```
本质上，这种写法属于 “模式匹配”，只要等式两边的模式相同，左边的变量就会被赋予对应的值。如果解构不成功，变量的值就等于`undefined`，不完全解构一样可以成功。

如果等号的右边是不可遍历解构，那么将会报错。

```javascript
//报错
let [foo]=1;
let [foo]=false
let [foo]={}//左右模式不同
```
解构同时适用于`var`,`let`,`const`，命令。

对于Set结构，也可以使用数组的解构赋值。

```javascript
let [x,y,z]=new Set(["a","b","c"]);
x//"a"
```
实际上，只要某种数据结构具有iterator接口，都可以采用数组形式的解构赋值。


```javascript

function* fibs(){
	var a=0,b=1;
	while(true){
		yield a;
		[a,b]=[b,a+b];
	}
}

var [first,second,third,fourth,fifth,sixth]=fibs();
sixth//5
```

上面的代码中`fibs`是一个Generator函数，原生具有Iterator接口。解构赋值会依次从这个接口获取值。

###默认值
解构赋值允许指定默认值。

```javascript
var [foo=true]=[];
foo //true

[x,y='b']=['a']; // x='a', y='b'
[x,y='b']=['a',undefined]; // x='a', y='b'
```

>Note: ES6nebula使用(`===`)，判断一个位置是否有值。所以，如果一个数组成员不严格等于`undefined`，默认值是不会生效的。

```javascript
var [x=1]=[undefined];//严格等于所以默认值生效

var [x=1]=[null];//不严格等于，所以默认值不生效
```

如果一个默认值是一个表达式，那么这个表达式是惰性求值的，即只有在用到的时候，才会求值。

```javascript
function f(){
	console.log('aaaa');
}

let[x=f()]=[1];
```

上面的代码因为`x`能取到值，所以函数`f`不会执行。

默认值可以引用解构赋值变量的其他变量，但该变量必须已经声明。（暂时性死区）

##2.对象的解构赋值

解构不仅可以用于数组还可用于对象。对象的解构与数组不同，数组是按元素次序排列的，而对象由键名决定。

```javascript
var {bar,foo}={foo:'aaa',bar:'bbb'};

var {baz}={foo:'aaa',bar:'bbb'};
baz // undefined
```

如果变量名与属性名不一致，必须写成下面这样。

```javascript
var {foo:baz}={foo:'aaa',bar:'bbb'};
baz //"aaa"

let obj={first:'hello',last:'world'};
let {first:f,last:l}=obj;
f+' '+l// 'hello world'
```

>Note: 这实际说明对象的解构赋值是下面形式的简写.
对象的解构赋值内部机制，真正被赋值的是后者，不是前者。

```javascript
var {foo:foo,bar:bar}={foo:'a',bar:'b'};
```


>Note：采用这种写法时，变量的声明和赋值是一体的。对于`let`和`const`来说，变量不能重新声明，所以一旦赋值的变量以前声明过，就会报错

```javascript
let foo;
let{foo}={foo:1};

let baz;
let {bar:baz}={bar:1};
//重复声明报错
```

对象的解构也可以指定默认值

```javascript 
var {x=3}={};
x//3

var{x,y=5}={x:1};
x,y//1,5

var {x:y=3}={};
x,y//undefined,3

var {x:y=3}={x:5};
x,y//undefined,5

var {message:msg='Something went wrong'}={};
msg//"Something went wrong"
```
默认对象生效的条件是，对象属性严格等于`undefined`。如果解构失败，变量的值等于`undefiend`

如果要将一个已经声明的变量解构，必须非常小心。

```javascript
var x;
{x}={x:1};//SyntaxError

({x}={x:1})
```

圆括号的作用是防止解析器把花括号解析为块级作用域的开始。

对象的解构赋值，可以很方便的将现有对象的方法，赋值到某个变量。

```javascript
let {log,sin,cos}=Math;//方便使用
```
由于数组本质是对象，因此可以对数组进行对象属性的解构。

```javascript
var arr=[1,2,3];
var {0:first,[arr.length-1]:last}=arr;
// 用变量代替关键位置方便使用。
```
>Note: 对象解构赋值时，对象左边的是右边的模式匹配，而不是变量。

##3.字符串的解构赋值

字符串也可以解构赋值。这是因为此时，字符串被转换成了一个类似数组的对象。

```javascript
const [a,b,c,d,e]='hello';
a-e //h-o
```
类数组对象都有`length`属性，因此还可以对这个属性解构赋值。

```javascript
let {length:len}='hello';
len//5
```

##4. 数值和布尔值的解构赋值
解构赋值时，如果等号右边是数值和布尔值，则会先转为对象。

```javascript
let {toString:s}=123;
s===Number.prototype.toString//true

let {toString:s}=true;
s===Boolean.prototype.toString//true

```

上面的栗子中，数值和布尔值的包装对象都有`toString`属性，因此变量`s`都能取到值。

解构赋值的规则是，只要等号右边的值不是对象，就先将其转为对象。由于`undefined`和`null`无法转为对象，所以对于这两个值解构赋值都会报错。

```javascript
let {prop:x}=undefined;
let {prop:y}=null
//TypeError
```

##5. 函数参数的解构赋值

```javascript
[[1,2],[3,4]].map([a,b]=>a+b);

[[1,2],[3,4]].map(function([a,b]){
	return a+b;
});
```

函数参数的解构也可以使用默认值。

```javascript
function move({x=0,y=0}={}){
	return [x,y];
}

move({x:3,y:8});//[3,8]
move({});//[0,0]
move();//[0,0]
```

下面的写法会触发不同的结果
```javascript
function move({x,y}={x:0,y:0}){
	return [x,y];
}

move({x:3,y:8});//[3,8]
move({});//[undefined,undefined]
move();//[0,0]

```
上面是对参数对象指定默认值，而不是对单一变量。所以会出现不同的结果。

##6. 圆括号问题

解构赋值方便，但对编译器来说一个式子是模式还是表达式不好解析。

由此带来的问题, ES6中规定，只要可能导致解构的歧义，就不要使用圆括号。实际情况更麻烦，所以尽量不要在解构中放置圆括号。

##7. 变量解构的用途

####1. 交换变量的值 

```javascript 
[x,y]=[y,x];// 很方便，不解释
```

####2. 从函数返回多个值

```javascript
//返回以个数组
function example(){
	return [1,2,3];
}
var [a,b,c]=example();

//返回一个对象
function example(){
	return {foo:1,bar:2};
}
var {foo,bar}=example();
```
####3. 函数参数定义

```javascript
//有序定义
function f([x,y,z]){}

//无序定义
function f({x,y,z}){}
```
####4. 提取JSON数据

```javascript
let {id,satus,data:number}=jsonData;
```
简化赋值，至少不用做长长的链式索引了。

####5. 参数的默认值

不解释，前面好多例子了

####6. 输入模块的指定方法
```javascript
const {SourceMapConsumer,SourceNode}=require('source-map');
```
真实情况是这个模块返回了一个对象 →_→;(又TMD是个简化语法的语法糖)







