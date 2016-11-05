#Iterator

1. Iterator的概念
2. 数据结构的默认Iterator接口
3. 调用Iterator接口的场合
4. 字符串的Iterator接口
5. Iterator接口与Gnerator函数
6. 遍历器对象的return(), throw()
7. for...of循环

----------------------------------------------

##1. Iterator (遍历器) 的概念

ES6为止，JavaScript总共有四种表示集合的数据结构：

1. Array
2. Object
3. Map
4. Set

使用者可以组合这几个数据结构来创建复合数据结构，然而这需要一种统一的借口机制来遍历不同的的数据接口。

Iterator就是这样一种接口，为不同的数据结构提供统一的访问机制。任何数据结构只要部署Iterator结构就可以完成遍历操作。

Iterator 的作用有三个：

1. 是为各种数据结构提供统一的API
2. 使得数据结构的成员能按某种次序排列
3. 供for...of消费

Iterator 的遍历过程是这样的：

1. 创建一个指针对象，指向当前数据结构的起始位置。遍历器对象本质上就是一个指针对象
2. 不断调用指针的`next`方法，直到它指向数据结构的结束位置。

每次调用`next`方法都会返回包含`value`和`done`的对象。

```javascript
function makeIterator(array){
    var nextIndex=0;
    return{
        next:function(){
            return nextIndex<array.length?{value:array[nextIndex++]}:{done:true};
        }
    }
}
```

ES6中，有些数据结构原生具备Iterator接口，不用任何处理就能被`for...of`遍历，有些不行，比如对象

##2. 数据结构的默认Iterator接口

Iterator接口的目的，就是为所有数据结构，提供了一种统一的访问机制，即`for...of`循环。

一种数据结构只要部署了Iterator接口，我们就称这种数据结构时 “可遍历的”。

ES6规定，默认的Iterator接口部署在数据结构的`Symbol.iterator`属性，或者说，一个数据结构只要具有`Symbol.iterator`属性，就可以认为是 “可遍历的”。`Symbol.iterator`属性本身是一个函数，就是当前数据结构默认的遍历器生成函数。执行这个函数，就会返回一个遍历器。

```javascript
const obj={
    [Symbol.iterator]:function(){
        return {
            next:function(){
                return {
                    value:1,
                    done:true
                };
            }
        };
    }
};
```

