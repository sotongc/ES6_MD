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
        console.log("set")
    },
    set:function(){
        console.log("set")
    }
});
```

上面的代码用自己定义的逻辑复写了`get`和`set`方法。

ES6原生提供Proxy构造函数，用来生成Proxy实例。

