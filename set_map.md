#Set和Map数据结构

1. Set
2. WeakSet
3. Map
4. WeakMap

##1. Set

###基本用法

ES6提供了新的数据结构Set。它类似于数组，但是成员的值都是唯一的，没有重复的值。

下面是Set的使用方法

```javascript
var set=new Set([1,2,3]);

[3,4,5].map(x=>s.add(x));
```

利用`Set`唯一性的特点，可以用来代替过去的数组去重的方式。

```javascript
[...new Set(array)]
```

>**Note: **向set添加值的时候不会发生类型转换。Set内部判断连个值是否相等用的是类似于`Object.is()`的 "Same-value-equality"的策略。

###Set 实例的属性和方法

- `Set.prototype.constructor`
- `Set.prototype.size`返回`Set`实例的成员总数
- `add()`添加某个值，返回Set结构本身
- `delete()`删除某个值，返回一个布尔值，表示删除是否成功
- `has()`返回一个布尔值，表示是否是该set的成员
- `clear()`清除所有成员，没有返回值

###遍历操作

Set结构的实例有四个遍历方法，可以用于遍历成员。

- `keys()`: 返回键名的遍历器
- `values()`: 返回键值的遍历器
- `entries()`: 返回键值对的遍历器
- `forEach()`: 使用回调函数遍历每个成员

>**Note: **`Set`的遍历顺序就是插入顺序。这个特性有时非常有用，比如使用Set保存一个回调函数列表，调用时就能保证按照添加顺序调用。

1. `keys()`,`values()`,`entries()`

三个方法返回的都是遍历器对象`Iterator`。由于Set结构没有键名，只有键值,所以`keys()`和`values()`方法的行为完全一致。

```javascript
let set=new Set(['red','green','blue']);

for(let item of set.keys()){
    console.log(item);
}

for(let item of set.values()){
    console.log(item);
}

for(let item of set){
    console.log(item);
}
// red green blue

for(let item of set.entries()){
    console.log(item);
}
//["red","red"]
//["green","green"]
//["blue","blue"]
```

2. `forEach()`
```javascript
let set=new Set([1,2,3]);
set.forEach((value,key)=>console.log(value*2));
```

##2. WeakSet

WeakSet结构与Set类似，也是不重复的值的集合。但是，它与Set有两个区别。

1. WeakSet 的成员只能是对象，而不能是其他类型的值。
2. WeakSet 中的对象都是弱引用，也就是说，如果其他对象都不再引用该对象那无论该对象是否存在于WeakSet中，GC机制都会回收内存。这一特性导致WeakSet的成员是不可引用的，因而WeakSet是不可遍历的。

```javascript
new WeakSet([[1,2],[3,4]]);

new WeakSet([1,2,3]);//报错，数组的成员为WeakSet成员，而不是数组本身，数组的成员只能是对象。
```

WeakSet结构有以下三个方法：

-WeakSet.add()
-WeakSet.delete()
-WeakSet.has()

>WeakSet的成员都是弱引用，但可以用来存储DOM节点而不必担心引发内存泄漏。

```javascript
const foos=new WeakSet()
class Foo{
    constructor(){
        foos.add(this)
    }
    method(){
        if(!foos.has(this)){
            throw new TypeError();
        }
    }
}
```

##3. Map

###目的和基本用法

JavaScript的对象（Object),本质上是键值对的集合（Hash结构）但传统上只能用字符串当做键。这给它带来了很大的限制。

ES6的Map数据结构类似于对象，也是键值对的集合，但键可以是各种类型的值。是更完善的HASH解构的实现。

```javascript
var m=new Map();
var o={p:'Hello World'};

m.set(o,'content');
m.get(o) //"content"

m.has(o)//true
m.delete(o)
m.has(o)//false
```

Map的构造函数也可以接受数组为参数，用数组位数表示键值对

```javascript
var map=new Map([
    ['name','张三'],
    ['title','Author']
]);
```

>Note: 只有对同一个对象的引用，Map结构才将其视为同一个键。

```javascript
var map=new Map();

map.set(['a'],555);
map.get(['a']); // undefined
```

上面的索引实际上是两个内存地址。Map的键实际上是跟内存地址绑定的，只要内存地址不一样就被视为两个键。Map对于相等的定义和`Object.is()`相同。

###实例方法及遍历方法

实例方法：

- `size`
- `set(key,valu)` 
- `get(key)` 
- `has(key)` 
- `delete(key)` 
- `clear()` 

遍历方法：

- `keys()`: 返回键名的遍历器
- `values()`: 返回键值的遍历器
- `entries()`: 返回键值对的遍历器
- `forEach()`: 使用回调函数遍历每个成员


##4. WeakMap

WeakMap和WeakSet 类似

