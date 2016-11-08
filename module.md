#Module

1. 严格模式
2. export命令
3. import命令
4. 模块的整体加载
5. export default命令
6. 模块的继承
7. ES6模块加载的实质
8. 循环加载
9. 跨模块常量
10. ES6模块的转码

ES6模块的设计思想，是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。CommonJS和AMD模块，都只能在运行时确定这些东西。这导致完全没有办法在编译时做 “静态优化”。

ES6模块不是对象，而是通过`export`命令显式指定输出的代码，输入时也采用静态命令的形式。

```javascript
import {stat,exists,readFile} from 'fs';
```

上面代码的实质是从`fs`模块加载3个方法，其他方法不加载。这种加载称为“编译时加载”，效率要比执行时加载方式高，当然也导致了没法引用ES6模块本身。

除了静态加载带来的各种好处，ES6模块还有以下好处。

- 不再需要UMD模块格式了
- 将来浏览器新的API就能用模块格式提供，不再必要做成全局变量或者`navigator`对象的属性。
- 不再需要对象作为命名空间，未来这些可以通过模块提供。

浏览器使用ES6模块的语法如下。

```html
<script type="module" src="foo.js"></script>
```

##1. 严格模式

ES6的模块自动采用严格模式。严格模式主要有以下限制：

- 变量必须声明后再使用
- 函数的参数不能有同名属性，否则报错
- 不能使用`with`语句
- 不能对只读属性报错
- 不能使用前缀0表示八进制数
- 不能删除不可删除的属性
- 不能删除变量`delete prop`
- `eval`不会再它的外层作用域引入变量
- `eval`和`arguments`不能被重新赋值
- `arguments`不会自动反应函数参数的变化
- 不能使用`arguments.callee`
- 不能使用`arguments.caller`
- 禁止`this`指向全局对象
- 不能使用`fn.caller`和`fn.arguments`获取函数调用的堆栈
- 增加了保留字

上面的限制模块都必须遵守。

##2. export 命令

模块功能主要由两个命令构成：`export`和`import`。

- `export`：用于规定模块的对外接口
- `import`：用于输入其他模块提供的功能

一个模块就是一个独立的问题件。该文件内部的所有变量，外部无法获取。如果希望外部能读取模块内部的变量需要用`export`向外暴漏。

```javascript
var fname="Michael",
    lname="Jackson",
    birth=1958;

function multiply(x,y){
    return x*y;
}

export {fname,lname,birth,multiply};
```

通常情况下,`export`输出的变量就是本来的名字，但是可以使用`as`关键字重命名。

```javascript
export{
    v1 as streamV1,
    v2 as streamV2,
    v2 as streamLatestVersion
};
```

需要注意的是，`export`命令规定的是对外的接口，必须与模块内部的变量建立一一对应的关系

```javascript
export 1; //报错

var m=1;
export m; //报错
```

上面两个都不是接口，正确写法如下:

```javascript
//写法一
export var m=1;

//写法二
var m=1;
export {m};

//写法三
var n=1;
export {n as m};
```

同样的，`function`和`class`的输出，也必须遵守这样的写法。另外，`export`语句输出的借口，与其对应的值是动态绑定关系，即通过该接口，可以取到模块内部实时的值。

```javascript
export var foo='bar';
setTimeout(()=>foo='baz',500);
```

上面代码输出变量`foo`，值为`bar`，500毫秒之后变成`baz`。这一点与CommonJS规范完全不同。CommonJS模块输出的是值的缓存，不存在动态更新。

最后，`export`命令可以出现在模块的任何位置，只要处于模块顶级作用域就可以。如果处于局部作用域就会报错，这是因为处于局部代码块中无法做静态优化，违背了ES6模块的设计初衷。

##3. import命令

使用`export`命令定义了模块的对外接口以后，其他JS文件就可以通过`import`命令加载这个模块。

```javascript
import {fname,lname,birth} from './profile';

function setName(element){
    element.textContent=`${fname} ${lname}`;
}
```

>**Note: **`import`命令接收的变量名必须和`export`对外导出的接口名相同。

如果想为输入的变量重新取一个名字，import命令要使用`as`关键字，将输入的变量重命名。

```javascript
import {lname as surname} from './profile';
```

>**Note: **`import`命令具有提升效果，会提升到整个模块的头部，首先执行。

##4. 模块的整体加载

除了制定加载某个输出值，还可以使用整体加载，即用`*`指定一个对象，所有输出值都加载这个对象上面。

```javascript
import * as circle from './circle';

console.log(circle.area(4));
console.log(circle.circumference(14));
```

##5. export default命令

`export default`命令用于为模块指定默认输出。

```javascript 
export default function(){
    console.log('foo');
}
```
上面代码是一个模块文件`export-default.js`，它的默认输出是一个函数。其他模块加载该模块时，`import`命令可以为该匿名函数指定任意名字。

```javascript
import customName from './export-default';
customName();
```

`export default`也可以用在具名函数的前面，视同匿名函数。这个命令用于指定模块的默认输出，当然一个模块也只能由一个默认输出。

本质上，`export default`就是输出一个叫做`default`的变量或方法，然后系统允许你为它取任意名字。正因为`export default`命令其实只是输出一个叫做`default`的变量，所以它后面不能跟变量声明语句。

```javascript
export var a=1;

var a=1;
export default a;//相当于 export default=a;

//错误
export default var a=1;
```

##6. 模块的继承

模块之间也可以继承。

```javascript
export * from 'circle';
export var e=2.71828182846;
export default function(x){
    return Math.exp(x);
}
```

加载模块

```javascript
import * as math from 'circleplus';
import exp from 'circleplus';
console.log(exp(math.e));
```

##7. ES6模块加载的实质

ES6模块加载的机制，与CommonJs模块完全不同。CommonJS模块输出的是一个值的拷贝，而ES6模块输出的是值的引用。

- CommonJS: 一旦输出一个值，模块内部的变化不会影响到这个值。除非加一个取值函数
- ES6 Module: 不会执行模块，之后生成一个动态的只读引用，不会缓存值。

由于ES6输入的模块变量，只是一个只读引用，所以任何重新赋值会报错。

```javascript
export let obj={};

import {obj} from './lib';

obj.prop=123; //OK
obj={};//报错
```

##8. 循环依赖

**Def: **【循环依赖】--a脚本依赖b,b的执行反过来又依赖a;

通常，循环依赖表示存在强耦合，如果处理不好，还可能导致递归依赖，使程序无法执行。实际上，这很难避免，模块加载机制必须考虑循环依赖的情况。

### CommonJS 模块的加载原理

CommonJS的一个模块，就是一个脚本文件。`require`命令第一次加载该脚本，就会执行整个脚本，然后再内存生成一个对象。

```javascript
{
    id:'...',
    exports:{},
    loaded:true,
    ...
}
```

上面的代码就是Node内部加载模块后生成的一个对象。以后需要用到这个模块的时候，就会到`export`属性上面取值。即使再次执行`require`命令，也不会再次执行该模块，而是到缓存中取值。除非手动清除缓存。

###CommonJS模块循环依赖

CommonJS模块的重要特性是加载时执行，即脚本代码在`require`的时候，就会全部执行。一旦出现某个模块被“循环加载”，就只输出已经执行的部分，还未执行的部分不会输出。

```javascript
//a.js
exports.done=false;
var b=require('./b.js');
console.log('在 a.js 之中，b.done=%j',b.done);
exports.done=true;
console.log('a.js 执行完毕');
```

```javascript
//b.js
exports.done=false;
var a=require('./a.js');
console.log('在 b.js 之中，a.done=%j',a.done);
exports.done=true;
console.log('b.js 执行完毕')；
```

验证：

```javascript
var a=require('./a.js');
var b=require('./b.js');
console.log('在 main.js 之中，a.done=%j, b.done=%j',a.done,b.done);
```

执行 main.js，运行结果如下。

```bash
$ noe main.js

在 b.js 之中，a.done=false
b.js 执行完毕
在 a.js 之中，b.done=true
a.js 执行完毕
在 main.js 之中，a.done=true, b.done=true
```

###ES6模块的循环加载

ES6处理“循环加载”与CommonJS有本质不同。ES6模块时动态引用，如果使用`import`从一个模块加载变量，那些变量不会被缓存，而是成为一个指向被加载模块的引用，需要开发者自己保证真正取值的时候能够成功。

```javascript
//a.js
import {bar} from './b.js'
console.log('a.js');
console.log(bar);
export let foo='foo';

//b.js
import {foo} from './a.js';
console.log('b.js');
console.log(foo);
export let bar='bar';
```

执行结果如下：

```bash
$ babel-node a.js
b.js
undefined
a.js
bar
```

再看一个稍微复杂的栗子：

```javascript
//a.js
import {bar} from './b.js';
export function foo(){
    console.log('foo');
    bar();
    console.log('执行完毕');
}
foo();

//b.js
import {foo} from './a.js'
export function bar(){
    console.log('bar');
    if(Math.random()>0.5){
        foo();
    }
}
```

```bash
$ babel-node a.js
foo
bar
foo
bar
执行完毕
执行完毕
```

上面的代码能执行，是因为ES6加载的变量，都是动态引用其所在的模块。只要引用存在，代码就能执行。

##9. 跨模块常量

上面说过，`const`声明的常量只在当前代码块有效。如果想设置跨模块的常量，可以采用下面的写法

```javascript
// constants.js 模块
export const A = 1;
export const B = 3;
export const C = 4;

// test1.js 模块
import * as constants from './constants';
console.log(constants.A); // 1
console.log(constants.B); // 3

// test2.js 模块
import {A, B} from './constants';
console.log(A); // 1
console.log(B); // 3
```




