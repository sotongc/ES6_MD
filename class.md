#Class

1. Class基本语法
2. Class的继承
3. 原生构造函数的继承
4. Class的取值函数（getter) 和存值函数（setter)
5. Class的Generator方法
6. Class的静态方法
7. Class的静态属性和实例属性
8. new.target属性
9. Mixin模式的实现

##1. Class基本语法

新的Class只是一个语法糖

```javascript
class Point{
    constructor(x,y){
        this.x=x;
        this.y=y;
    }
    toString(){
        return '('+this.x+','+this.y+')';
    }
}
```

ES6的类，完全可以看作构造函数的另一种写法。

```javascript
class Point{}

typeof Point //"function"
Point === Point.prototype.constructor //true
```

构造函数的`prototype`属性，在ES6的 "类" 上面继续存在。事实上，类的所有方法都定义在类的`prototype`属性上面。

由于类的方法都定义在`prototype`对象上面，所以累的新方法可以添加在`prototype`对象上面。`Object.assign`方法可以很方便地一次向类添加多个方法。

```javascript
class Point{
    constructor(){

    }
}

Object.assign(Point.prototype,{
    toString(){},
    toValue(){}
});
```

另外，类的内部所有定义的方法，都是不可枚举的。这一点和ES5行为不一致。同时类的属性名可以采用表达式。

###类的实例对象

与ES5一样，实例的属性除非显示定义在其本身（`this`对象），否则都是定义在原型上

###不存在变量提升

Class不存在变量提升，这一点与ES5完全不同。

```javascript
new Foo();//报错
class Foo{}
```

###this的指向
和ES5相同

##2. Class的继承

Class之间可以通过`extends`关键字实现继承，这比ES5的通过修改原型链实现继承，要清晰和方便的多。

```javascript
class ColorPoint extends Point{
    constructor(x,y,color){
        super(x,y);
        this.color=color;
    }
    toString(){
        return this.color+' '+super.toString();
    }
}
```

>- ES5的继承，实质是先创造子类的实例对象`this`，然后再将父类的方法添加到`this`上面。 
>- ES6的继承机制完全不同，实质是先创造父类的实例对象`this`,然后再用子类的构造函数修改`this`。

如果子类没有定义`constructor`方法，这个方法会被默认添加，在子类的构造函数中，只有调用`super`之后，才可以使用`this`关键字，否则会报错。这是因为子类实例的构建，是基于对父类实例加工，只有`super`方法才能返回父类实例。

###super关键字

`super`这个关键字，有两种用法，含义不同。

1. 作为函数调用时，`super`代表父类的构造函数。
2. 作为对象调用时,即`super.prop`代表父类。此时`super`既可以引用父类实例的属性和方法，也可以引用父类的静态方法。

##3. 原生构造函数的继承

原生构造函数是指语言内置的构造函数，通常用来生成数据结构。ECMAScript的原生构造函数大致有下面这些。

- `Boolean()`
- `Number()`
- `String()`
- `Array()`
- `Date()`
- `Function()`
- `RegExp()`
- `Object()`

以前，这些原生构造函数是无法继承的.ES5先新建子类的实例对象`this`,再将父类的属性添加到子类上，由于父类内部属性无法获取，导致无法继承原生的构造函数。ES6允许继承原生构造函数定义子类，因为ES6是先新建父类的实例对象`this`，然后再用子类的构造函数修饰`this`,使得父类的所有行为都可以继承。

##4. Class的取值函数（getter)和存值函数(setter)

与ES5一样，在Class内部可以使用`get`和`set`关键字，对某个属性设置存值函数和取值函数，拦截该属性的存取行为。

```javascript
class MyClass{
    constructor(){

    }
    get prop(){
        return 'getter'
    }
    set prop(value){
        console.log('setter: '+value);
    }
}

let inst=new MyClass();
inst.prop=123;
inst.prop
```

存值和取值是定义在属性的descriptor对象上的。

##5. Class的Generator方法

```javascript
class Foo{
    constructor(...args){
        this.args=args;
    }
    * [Symbol.iterator](){
        for(let arg of this.agrs){
            yield arg;
        }
    }
}

for(let x of new Foo('hello','world')){
    console.log(x);
}

//hello
//world
```

##6. Class的静态方法

类相当于实例的原型，所有在类中定义的方法，都会被实例继承。如果在一个方法前，加上`static`关键字，就表示该方法不会被**实例**继承，而是直接通过类来调用，这就是 “静态方法”。

```javascript
class Foo{
    static classMethod(){
        return 'hello';
    }
}

Foo.classMethod() //"hello"

var foo=new Foo();
foo.classMethod() //TypeError:foo.classMethod is not a function
```

父类的静态方法可以被子类继承

```javascript
class Bar extends Foo{}

Bar.classMethod(); //'hello'
```

静态方法也是可以从`super`对象上调用的

```javascript
class Bar extends Foo{
    static classMethod(){
        return super.classMethod()+', too';
    }
}
```

##7. Class的静态属性和实例属性

//ES7提案

##8. new.target属性

ES6为`new`引入了`target`属性，返回`new`作用于的那个构造函数。如果构造函数不是通过`new`调用的则返回`undefined`这个可以用来确定构造函数是怎么调用的。

>**Note: **子类调用父类时，`new.target`会返回子类

##9. Mixin模式的实现

Mixin模式指的是，将多个类的接口混入另一个类。实现如下

```javascript
function mix(...mixins){
    class Mix{}

    for(let mixin of mixins){
        copyProperties(Mix,mixin);
        copyProperties(Mix.prototype,mixin.prototype);
    }
    return Mix;
}

function copyProperties(target,source){
    for(let key of Reflect.ownKeys(source)){
        if(key!=="constructor"&&key!=="prototype"&&key!=="name"){
            let desc=Object.getOwnPropertyDescriptor(source,key);
            Object.defineProperty(target,key,desc);
        }
    }
}
```

上面代码的`mix`函数，可以将多个对象合成为一个类。使用的时候，只要继承这个类即可。




