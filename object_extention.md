#对象的扩展

1. 属性的简洁表示法 --纯粹的语法糖
2. 属性名表达式
3. 方法的name属性
4. Object.is()
5. Object.assign()
6. 属性的可枚举性
7. 属性的遍历
8. __proto__属性，Object.setPrototypeOf(), Object.getPrototypeOf()
9. Object.values(),Object.entries()---ES7 提案
10. 对象的扩展运算符
11. Object.getOwnPropertyDescriptors()

##1. 属性的简洁表示法

ES6 允许直接写入变量和函数，作为对象的属性和方法。这样的书写更加简洁

```javascript
var foo='bar';
var baz={foo};
baz //{foo: "bar"}

//equals
var baz={foo:foo};
```

上面代码表明，ES6允许在对象之中，只写属性名，不写属性值。这时属性值等于属性名所代表的变量。

除了属性简写，方法也可以简写

```javascript
var o={
    method(){
        return "Hello!";
    }
};

//等同于

var o={
    method:function(){
        return "Hello!";
    }
};
```

CommonJS模块输出变量，就非常合适使用简洁写法。

```javascript
module.exports={getItem,setItem,clear};
```

##2. 属性名表达式

JavaScript语言定义对象的属性，有两种方法。`obj.foo` 和 `obj['f'+'oo']`。但是如果使用对象字面量定义对象，ES5中只能使用标识符来定义。

```javascript
var obj={
    foo:true
};
```

ES6允许字面量定义对象时，用表达式作为对象属性名来定义。

```javascript
let propKey='foo';
var obj={
    [propKey]:true
};
```

>Note: 表达式与简洁表示法不能同时使用，否则会报错。

##3. 方法的name属性

函数的`name`属性，返回函数名。对象方法也是函数，因此也有`name`属性。

1. 方法`name`属性返回函数名。
2. 使用取值或者存值函数，方法名前面会加上`'get'`或`'set'`
3. `bind`方法创造的函数，方法名前加上`'bound'` 
4. `Function`构造函数创造的函数，返回 `'anonymous'` 
5. 如果对象的方法是一个Symbol值，那么返回时这个Symbol值的描述

```javascript
const key1=Symbol('des');
const key2=Symbol();

let obj={
    [key1]() {},
    [key2]() {}
};

obj[key1].name //"[des]"
obj[key2].name //""
```


##4. Object.is()

ES5比较两个值是否相等，只有两个运算符：相等`==`和严格相等`===`。这两种运算符都有缺点。

1. `==`会隐式转换数据类型
2. `===`对于`NaN`不等于自身，以及`+0===-0`。

JavaScript 缺乏这样一种运算：在所有环境中，只要两个值是一样的，它们就应该相等。

ES6提出 "Same-value equality" 算法，用来解决这个问题。 `Object.is`就是用来比较两个值是否严格相等，于`===`运算符行为基本一致；

```javascript
//true
Object.is('foo','foo')
Object.is({},{})
Object.is(NaN,NaN)

//false
Object.is(+0,-0)
```

##5. Object.assign()

`Object.assign`方法用于对象的合并，将源对象的所有可枚举属性，复制到目标对象；方法的第一个参数是目标对象，后面的参数都是源对象；用法类似于jQuery中的`$.extend()`。

>**Note:** 如果目标对象与源对象有同名属性，或者多个源对象有同名属性，则后面的会覆盖前面的属性。如果只有一个参数，则返回该参数，如果该参数不是对象，会先隐式转换为对象再返回。由于`undefined`和`null`无法转换为对象，所以会报错。

```javascript
var target={};
Object.assign(target,src1,src2);
```

>**Note: **如果非对象参数出现在源对象的位置，那么处理规则有所不同。首先，这些参数都会转成对象，如果无法转成对象就会跳过但不报错。其他类型的值不再首参数也不会报错。但是除了字符串会以数组形式，拷贝入目标对象其他值都不会产生效果。

```javascript
var v1='abc';
var obj=Object.assign({},v1);//{0:'a',1:'b',2:'c'}
```

`Object.assign`拷贝的属性是有限制的，只拷贝源对象的自身属性，不拷贝继承属性，也补考呗不可枚举的属性。

#### 注意点

`Object.assign`方法实行的是浅拷贝，而不是深拷贝。也就是说，如果源对象某个属性的值是对象，那么目标对象拷贝得到的是这个对象的引用。在使用`Object.assign`的时候尽量确保对象值是简单类型。

##6. 属性的可枚举性

对象的每个属性都有一个描述对象（Descriptor），用来控制该属性的行为。`Object.getOwnPropertyDescriptor`方法可以获取该属性的描述对象。

```javascript
let obj={foo:123};
Object.getOwnPropertyDescripor(obj,'foo');
/*
    {
        value:123,
        writable:true,
        enumerable:true,
        configurable:true
    }
*/
```

`enumerable`可枚举属性为`false`的属性会被某些操作忽略。

ES5有三个操作会忽略：

1. `for in`循环：只遍历对象自身的和继承的可枚举的属性
2. `Object.keys()`：返回对象自身的所有可枚举的属性的键名
3. `JSON.stringify()`:只串行化对象自身的可枚举的属性

ES6新增了一个操作

- `Object.assign()`:只拷贝对象自身的可枚举的属性。

这四个操作中，只有`for in`循环会返回继承的属性。实际上，引入`enumerable`的最初目的，就是让某些属性可以规避掉`for in`操作。比如对象原型的`toStirng`方法，以及数组的`length`属性。

>**Note: **ES6规定，所有Class的原型的方法都是不可枚举的。

```javascript
Object.getOwnPropertyDescriptor(class {foo() {}}.prototype,'foo').enumerable
```

操作中引入继承的属性会让问题复杂化，大多数时候，我们只关心对象自身的属性。所以，尽量不要用`for in`循环，而用`Object.keys()`代替。

##7. 属性的遍历

ES6中一共有5中方法可以遍历对象的属性。

1. `for in`
2. `Object.keys()`: 返回一个数组，包括对象自身的所有可枚举属性（不包含继承属性和Symbol属性）
3. `Object.getOwnPropertyNames(obj)`: 返回以个数组，包含对象自身的所有属性（不包含Symbol属性，但包含不可枚举属性）
4. `Object.getOwnPropertySymbols()`: 返回一个数组，包含对象自身的所有Symbol属性。
5. `Reflect.ownKeys(obj)`: 返回一个数组，包含对象的所有属性，包括不可枚举和Symbol。

以上的遍历方法都遵循同样的属性遍历的次序规则。

-首先遍历所有属性名为数值的属性，按照数字排序
-其次遍历所有属性名为字符串的属性，按照生成时间排序
-随后遍历所有属性名为Symbol的属性，按照生成时间排序

##8. `__proto__`属性，Object.setPrototypeOf(),Object.getPrototypeOf()

###(1) `__proto__`属性

`__proto__`属性，用来读取或者设置当前对象的`prototype`对象。目前所有浏览器都部署了这个属性。

```javascript
//ES6
var obj={
    method:function(){}
};
obj.__proto__=someOtherObj;
```

该属性没有写入ES6的正文，而是写入了附录，原因是`__proto__`前后的双下划线，说明它本质上是一个内部属性，而不是一个正式的对外的API，只是由于浏览器广泛支持，才被加入了ES6。从语义和兼容性角度来讲都最好当这个属性不存在。

###（2）Object.setPrototypeOf()

`Object.setPrototypeOf`方法的作用与`__proto__`相同，用来设置一个对象的`prototype`对象。这是ES6正式推荐的设置原型对象的方法。

```javascript
//格式
Object.setPrototypeOf(object,prototype)

//用法
var o=Object.setPrototypeOf({},null);
```

###(3) Object.getPrototypeOf(), Object.setPrototypeOf()

```javascript
function Rectangle(){}

var rec=new Rectangle();

Object.getPrototypeOf(rec)===Rectangle.prototype //true

Object.setPrototypeOf(rec,Object.prototype);
Object.getPrototypeOf(rec)===Rectangle.prototype //false
```

##10. 对象的扩展运算符

目前，ES7有一个提案，将Rest运算符引入对象。Babel转码器已经支持这项功能。

###(1) 解构赋值

```javascript
let {x,y,...z}={x:1,y:2,a:3,b:4};
x //1
y //2
z //{a:3,b:4}
```

上面代码中，变量z是解构赋值所在的对象。它获取等号右边的所有尚未读取的键，将它们联通值一起拷贝过来。

由于解构赋值要求等号右边是一个对象，所以如果等号右边是`undefined`或`null`，就会报错，因为它们无法转换为对象。

同时解构赋值必须是最后一个参数，否则也会报错

```javascript
let{x,y,...z}=null;//runtime error
let{x,y,...z}=undefined;//runtime error

let{...x,y,z}=obj;//syntax error
let{x,...y,...z}=obj;//syntax error
```

>**Note: **解构赋值的拷贝是浅拷贝；解构赋值不会拷贝继承自原型对象的属性。

```javascript
let o1={a:1};
let o2={b:2};
o2.__proto__=o1;
let o3={...o2} //{b:2}
```

解构赋值的翼个用处是扩展某个函数的参数，引入其他操作

```javascript
function baseFunc({a,b}){}
function WarpperFunction({x,y,...restConfig}){
    return baseFunction(restConfig);
}
```

###(2) 扩展运算符

扩展运算符`...`用于去除参数对象的所有可遍历属性，拷贝到当前对象中。这等同于使用`Object.assign`方法。

```javascript
//对象拷贝
let z={a:3,b:4};
let n={...z};

//对象合并和Object.assign情况完全一样
let ab={...a,...b};
```

用来修改现有对象部分的部分属性就很方便了

```javascript
let newVersion={
    ...previousVersion,
    name:'New Name' // Override the name property
};
```

##11. Object.getOwnPropertyDescriptors()

返回一个对象，所有原对象的属性名都是该对象的属性名，对应的属性值就是该属性的描述对象。