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


