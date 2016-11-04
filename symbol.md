#Symbol

1. 概述
2. 作为属性名的Symbol
3. 实例：消除魔术字符串
4. 属性名的遍历
5. Symbol.for(),Symbol.keyFor()
6. 实例：模块的Singleton模式
7. 内置的Symbol值

##1. 概述

ES5的对象属性名都是字符串，这容易产生同名属性的冲突，为了从根本上防止属性名的冲突，ES6引入了Symbol。


**Def: **Symbol是ES6引入的新的原始数据类型，表示独一无二的值。

前六种原始数据类型是：

1. undefine
2. null
3. boolean
4. string
5. number
6. object
7. Symbol

Symbol 值通过`Symbol` 函数生成。现在对象的属性名可以有两种类型，一种是原来就有的字符串，另一种就是新增的Symbol类型。

```javascript
typeof Symbol('bar'); // symbol
```

Symbol是一种类似于字符串的数据类型，但不是对象,可以接受一个字符串参数作为对实例的表述

> **Note: ** `Symbol`函数的参数只是表示对当前Symbol值的描述，因此相同参数的`Symbol`函数返回的值是不相等的。

Symbol 不能与其他类型的值运算，但是可以显示转换为字符串`.toString()`,也可以显示转换为布尔值`Boolean()`但不能转换为数值。

##2. 作为属性名的Symbol

```javascript
var s=Symbol();
var obj={
    [s]:'Hello'
};
```

>**Note: **Symbol不能用`.`标识符来索引，只能用方括号`[]`

常量使用Symbol值最大的好处，就是其他任何值都不可能有相同的值了，因此可以保证下面的`switch`语句会按设计的方式工作。

```javascript
const RED=Symbol();
const GREEN=Symbol();

function getComplement(color){
    switch(color){
        case RED:
            return GREEN;
            break;
        case GREEN:
            return RED;
            break;
        default:
            throw new Error("Undefined color");
            break;
    }
}
```

##3. 实例：消除魔术字符串

**Def: **【魔术字符串】--代码中多次出现，与代码形成强耦合的某一个具体的字符串或者数值。风格良好的代码应该尽量消除魔术字符串，改由含义清晰的变量代替。

下面是一个魔术字符串的栗子：

```javascript
function getArea(shape,options){
    var area=0;

    swtich(shape){
        case 'triangle': //魔术字符串
            area=.5*options.w*options.h;
            break;
        /*more*/
    }
    return area;
}

getArea('triangle',{width:100,height:100});
```

常用的而消除魔术字符串的方法，就是把它写成一个变量

```javascript
var shapeType={
    triangle:'triangle'
};

function getArea(shape,options){
    var area=0;
    switch(shape){
        case shapeType.triangle:
    }
    return area
}
```

其实`shapeType.triangle`等于哪个值并不重要，只要确保不会跟其他`shapeType`冲突即可。所以可以改用symbol值。

```javascript
const shapeType={
    triangle:Symbol()
};
```

##4. 属性名的遍历

Symbol 作为属性名不会被下列语句遍历：

1. `for in`
2. `for of`
3. `Object.keys()` // 返回对象自身的所有可枚举的属性的键名
4. `Object.getOwnPropertyNames()`

`Object.getOwnPropertySymbols`可以返回symbol类型的属性，返回一个数组，成员是当前对象的所有用作属性名Symbol的值。

##5. Symbol.for(), Symbol.keyFor()

`Symbol.for` 方法接收一个字符串为参数，然后搜索有没有以该参数作为名称的Symbol值。如果有，就返回这个Symbol值，否则就新建并返回一个以该字符串为名称的Symbol。

```javascript
var s1=Symbol.for('foo');//找不到，新建一个并返回
var s2=Symbol.for('foo');//找到了刚才新建的那个

s1===s2 //true
```

`Symbol.for()`与`Symbol()`这两种写法，都会生成新的Symbol。区别在于，前者会被登记在全局环境中供搜索，后者不会。

例如，调用`Symbol.for()`30次，每次都返回相同的值，但`Symbol()`30次调用只会返回30个不同的值。

`Symbol.keyFor`方法返回一个已登记的Symbol类型值的key。

```javascript
var s1=Symbol.for("foo");
Symbol.keyFor(s1) // "foo"

var s2=Symbol("foo");
Symbol.keyFor(s2) //undefined
```

上面 `s2`属于未登记的Symbol值，所以返回undefined

>**Note: **`Symbol.for`为Symbol值登记的名字，是全局环境的，可以在不同的iframe或service worker中取到同一个值。

##6.实例：模块的 Singleton 模式

Singleton模式指的是调用一个类，任何时候返回的都是同一个实例。

较为可靠的解决方法：可防覆盖，但防不了复写。

```javascript

const FOO_KEY=Symbol.for('foo');

function A(){
    this.foo='hello';
}

if(!global[FOO_KEY]){
    global[FOO_KEY]=new A();
}

module.exports=global[FOO_KEY];

var a=require('./mod.js');
global[Symbol.for('foo')];//'hello'
```

##7. 内置的Symbol值

除了定义自己使用的Symbol值以外，ES6还提供了11个内置的Symbol值，指向语言内部使用的方法。
