# Proxy和Reflect

1. Proxy概述
2. Proxy实例的方法
3. Proxy.revocable()
4. Reflect概述
5. Reflect对象的方法

##1. Proxy概述

Proxy用于修改某些操作的默认行为，等同于在语言层面做出修改，所以属于一种 “元编程”

Proxy可以理解成，一个钩子函数，外界对该对象的访问，都必须先通过这层，然后可以对外界的访问进行过滤和改写。

```javascript
var obj=new Proxy({},{
    get:function(target,key,receiver){
        return 35;
    },
    set:function(){
        return 35;
    }
});
```

上面的代码用自己定义的逻辑复写了`get`和`set`方法。

ES6原生提供Proxy构造函数，用来生成Proxy实例。

```javascript
var proxy=new Proxy(target,handler);
```

`target`为所要拦截的目标对象，`handler`参数也是一个对象，用来定制拦截行为

>**Note: **要使得Proxy起作用，必须针对Proxy实例进行操作，而不是针对目标对象进行操作；如果`handler`没有设置任何拦截行为，就等同于直接访问原对象。

下面试`Proxy`支持的拦截操作一览

1. `get(target,propKey,receiver)` 拦截对象的属性读取
2. `set(target,propKey,value,receiver)` 拦截对象属性的设置
3. ...

## 4. Reflect 概述

`Reflect`对象与`Proxy`对象一样，夜是ES6为了操作对象而提供的新API。`Reflect`对象的设计目的有这样几个。

1. 将`Object`对象的一些明显属于语言内部的方法放到`Reflect`对象上。现阶段，某些方法同时在`Object`和`Reflect`对象上部署，未来将只会使用`Reflect` 
2. 修改某些`Object`方法返回的结果，让其变得更合理
3. 让`Object`操作都变成函数行为。
4. `Reflect`对象的方法与`Proxy`对象的方法一一对应，只要是`Proxy`对象的方法，就能在`Reflect`对象上找到对应的方法。不管怎么修改默认行为，你总可以在`Reflect`上获取默认行为

```javascript
var loggedObj=new Proxy(obj,{
    get(target,name){
        return Reflect.get(target,name);
    },
    has(target,name){
        return Reflect.has(target,name)
    }
});
```


##5. Reflect 对象的方法

- Reflect.apply(target,thisArg,args)
- Reflect.construct(target,args)
- Reflect.get(target,name,receiver)
- Reflect.set(target,name,value,receiver)
- Reflect.defineProperty(target,name,desc)
- Reflect.deleteProperty(target,name)
- Reflect.has(target,name)
- Reflect.ownKeys(target)
- Reflect.isExtensible(target)
- Reflect.preventExtensions(target)
- Reflect.getOwnPropertyDescriptor(target, name)
- Reflect.getPrototypeOf(target)
- Reflect.setPrototypeOf(target, prototype)