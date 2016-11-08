#编程风格

1. 块级作用域
2. 字符串
3. 解构赋值
4. 对象
5. 数组
6. 函数
7. Map结构
8. Class
9. 模块
10. ESLint的使用

本章探讨如何将ES6的新语法，运用到编码实践之中，与传统的JavaScript语法结合在一起，写出合理的，易读的，可维护的代码。

-------------------------------------------------------------------------------

##1. 块级作用域

###（1）let 取代 var

1. let存在块级作用域
2. let 有暂时性死区
3. let 没有变量提升
4. 使用 let 遵循先声明后使用的原则

###（2）全局常量和线程安全

在`let`和`const`之间，建议优先使用`const`，尤其是在全局环境，不应该设置变量，只应该设置常量。这符合函数式编程思想，有利于将来的分布式运算。

`const`声明常量还有两个好处：

1. 阅读代码的人立刻会意识到不应该修改这个值
2. 防止无意间修改变量导致的错误

所有函数都应该设置为常量。

长远来看，JavaScript可能会有多线程的实现，这时`let`表示的变量，只应该出现在单线程运行的代码中，不能是多线程共享的，这样有利于保证线程安全。

-------------------------------------------------------------------------------

##2. 字符串

静态字符串一律使用单引号或反引号，不使用双引号。及动态字符串使用反引号。

```javascript
const a='foobar';
const b=`foo${a}bar`;
const c='foobar';
```

-------------------------------------------------------------------------------

##3. 解构赋值

使用数组成员对变量赋值时，优先使用解构赋值。

```javascript
const arr=[1,2,3,4];

const [first,second]=arr;
```

函数的参数如果是对象的成员，优先使用解构赋值。

```javascript
function getFullName({fname,lname}){
    return `${fname} ${lname}`;
}
```

如果函数返回多个值，优先使用对象的解构赋值，而不是数组的解构赋值。这样便于以后添加返回值，以及更改顺序。

```javascript
let input={right:0,left:0,top:0,bottom:0};
function processInput({left,right,top,bottom}){
    return {left,right,top,bottom};
}

const {left,right}=processInput(input);
```

-------------------------------------------------------------------------------

##4. 对象

单行定义的对象，最后一个成员不以逗号结尾。多行定义的对象，最后一个成员以逗号结尾。

```javascript
const a={k1:v1,k2:v2};
const b={
    k1:v1,
    k2:v2,
};
```

对象尽量静态化，一旦定义，就不得随意添加新的属性。如果添加属性不可避免，要使用`Object.assign`方法。

```javascript
const a={};
Object.assign(a,{x:3});
```

如果对象的属性名是动态的，可以在创造对象的时候，使用属性表达式定义。

```javascript
//目的在于将对象属性定义在一块，提高可读性
const obj={
    id:5,
    name:'San Francisco',
    [getKey('enable')]:true,
};
```

-------------------------------------------------------------------------------

##5. 数组

使用扩展运算符拷贝数组。

```javascript
const itemsCopy=[...items];
```

使用`Array.from`方法，将类似数组的对象转化为数组。

```javascript
const foo=document.querySelectorAll('.foo');
const nodes=Array.from(foo);
```

-------------------------------------------------------------------------------

##6. 函数

立即执行函数可以写成箭头函数的形式。

```javascript
(()=>{
    console.log(1)
})();
```

那些需要使用函数表达式的地方，尽量用箭头函数代替，这样更简洁，且绑定了`this`指向。

```javascript
[1,2,3].map(x=>x*x);
```

箭头函数取代`Function.prototype.bind`,不应该再用self/_this/that绑定`this`。简单的单行的不会复用的函数使用箭头函数，复杂的建议还是使用传统写法。

所有配置项都应该集中在一个对象，放在最后一个参数，布尔值不可直接作为参数。

```javascript
function divide(a,b,{option=false}={}){

}
```

不要再函数体内再使用`arguments`，使用reset运算符代替。因为`arguments`是个伪数组，而reset运算符可以提供一个真正的数组。

```javascript
function concatAll(...args){
    return args.join('');
}
```

使用默认语法设置参数的默认值，别再在函数体中使用类型初始化。

```javascript
function handleThings(opt={}){
    //而不是
    //opts=opts||{}；
}
```

-------------------------------------------------------------------------------

##7. Map结构

注意区分Object和Map,只有模拟现实世界实体对象时，才使用Object。如果只是需要哈希结构使用Map，因为Map内部返回Iterator接口。并且Map的键不局限于string可以是任意数据类型。

-------------------------------------------------------------------------------

##8. Class

总是用Class, 取代需要prototype的操作。因为Class的写法更简洁，易于理解。并使用`extends`继承。

-------------------------------------------------------------------------------

##9. 模块

使用原生替代CommonJS

-------------------------------------------------------------------------------

##10. ESLint 的使用

ESlint 是一个语法规则和代码风格的检查工具，可以用来保证写出语法正确，风格统一的代码。

首先安装ESLint

```bash
$ npm i -g eslint
```

然后，安装Airbnb语法规则

```bash
$ npm i -g eslint-config-airbnb
```

最后，在项目的根目录下新建一个`.eslintrc`文件，配置ESLint。

```json
{
    "extends":"eslint-config-airbnb"
}
```

现在就可以检查，当前代码是否符合预设的规则。

`index.js`文件的代码如下

```javascript
var unusued = 'I have no purpose!';

function greet() {
    var message = 'Hello, World!';
    alert(message);
}

greet();
```

使用ESLint检查

```bash
$ eslint index.js
index.js
  1:5  error  unusued is defined but never used                 no-unused-vars
  4:5  error  Expected indentation of 2 characters but found 4  indent
  5:5  error  Expected indentation of 2 characters but found 4  indent

✖ 3 problems (3 errors, 0 warnings)
```


