#Promise对象

1. Promise的含义
2. 基本用法
3. Promise.prototype.then()
4. Promise.prototype.catch()
5. Promise.all()
6. Promise.race()
7. Promise.resolve()
8. Promise.reject()
9. 两个有用的附加方法
10. 应用

##1. Promise的含义

Promise是异步编程的一种解决方案。最早由社区提出和实现，ES6将其写进了语言标准，原生提供了Promise对象。

Promise对象有以下两个特点：

1. 对象的状态不受外界影响。
2. 一旦状态改变，就不会再变，任何时候都可以得到这个结果。

有了`Promise`对象，就可以将异步操作以同步操作的流程表达出来，避免了层层嵌套的回调函数。

`Promise`的缺点：

1. 无法取消，一旦新建它会立即执行，中途无法取消。
2. 如果不设置回调函数，`Promise`内部抛出的错误，不会反应到外部。
3. 当处于`Pending`状态时，无法得知目前进展到哪一个阶段。

##2. 基本用法

ES6规定，Promise对象是一个构造函数，用来生成Promise实例。下面代码创造了一个Promise实例

```javascript
var promise=new Promise(function(resolve,reject){
    if(/*异步操作成功*/){
        resolve(value);
    }else{
        reject(error);
    }
});
```

`Promise`构造函数接收一个函数作为参数，该函数的两个参数分别是`resolve`和`reject`。它们是两个函数，由JavaScript引擎提供，不用自己部署。

- `resolve`函数的作用是，将Promise对象的状态从“未完成”变为“成功”，在异步操作成功时调用，并将异步操作的解构作为参数传递出去。
- `reject`函数的作用是，将Promise对象的状态从“未完成”变为“失败”，在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。

`Promise`实例生成以后，可以用`then`方法分别制定`Resolved`状态和`Reject`状态的回调函数。

```javascript
promise.then(function(value){
    // success
},function(error){
    // failure
});
```

下面是一个Promise对象的简单例子。

```javascript
function timeout(ms){
    return new Promise((resolve,reject)=>{
        setTimeout(resolve,ms,'done');
    });
}

timeout(100).then((value)=>{
    console.log(value);
});
```

Promise新建后就会立即执行。

```javascript

let promise=new Promise(function(resolve,reject){
    console.log('Promise');
    resolve();
});

promise.then(function(){
    console.log('Resolved');
});

console.log("Hi");

// 'Promised'
// 'Hi'
// 'Resolved'
```

下面是一个用Promise对象实现的Ajax操作的例子。

```javascript
var getJSON=function(url){
    var promise=new Promise(function(){
        var xhr=new XMLHttpRequest();
        xhr.open("GET",url);
        xhr.onreadystatechange=handler;
        xhr.responseType="json";
        xhr.setRequestHeader("Accept","application/json");
        xhr.send();

        function handler(){
            if(this.readyState!==4){
                return;
            }
            if(this.status===200){
                resolve(this.response);
            }else{
                reject(new Error(this.statusText));
            }
        }
    });
    return promise;
};

getJSON("/posts.json").then(function(json){
    console.log(json)
},function(error){
    console.log(error)
});
```

##3. Promise.prototype.then()

Promise 实例具有`then`方法，也就是说，`then`方法返回的是一个新的Promise实例（注意，不是原来那个Promise实例）。因此可以采用链式写法。

```javascript
getJSON("/posts.json").then(function(){
    return json.post;
}).then(function(post){
    //...
});
```

第一个回调函数完成后，会将返回结果作为参数，传入第二个回调函数。采用链式的`then`，可以指定一组按照次序调用的回调函数。这是，前一个回调函数，有可能返回的还是一个Promise对象，这时后一个回调函数，就会等待该Promise对象的状态发生变化，才会被调用。

```javascript
getJson("/post/1.json").then(function(post){
    return getJson(post.commentURL);
}).then(function funcA(comments){
    console.log("Resolved: ",comments);
},function funcB(err){
    console.log("Rejected",err);
});
```

##4. Promise.prototype.catch()

`Promise.prototype.catch`方法是`.then(null,rejection)`的别名，用于指定发生错误时的回调函数。另外，`then`方法指定的回调函数，如果运行中抛出错误，也会被`catch`方法捕获。

```javascript
getJSON("/posts.json").then(function(posts){
    //...
}).catch(function(error){
    console.log("发生错误",error)
});
```

Promise对象的错误具有 "冒泡"性质，会一直向后传递，直到被捕获为止。也就是说，错误总是会被下一个`catch`语句捕获。

```javascript
getJSON("/post/1.json").then(function(post){
    return getJSON(post.commentURL);
}).then(function(comments){
    // some code
}).catch(function(error){
    // 处理前面三个Promise产生的错误
});
```

前面三个Promise对象，任意一个抛出错误，都会被最后一个`catch`捕获。

##5. Promise.all()

`Promise.all`方法用于将多个Promise实例，包装成一个新的Promise实例。

```javascript
var p=Promise.all([p1,p2,p3]);
```

Promise.all 方法的参数可以不是数组，但必须具有Iterator接口，且返回的每个成员都是Promise实例。

p 的状态由`p1`,`p2`,`p3`决定，分成两种情况。

1. 只有p1,p2,p3都变成`fulfilled`，p的状态才会变成`fulfilled`。此时p1,p2,p3的返回值组成一个数组，传递给p的回调函数。
2. 只要p1,p2,p3之中有一个被`reject`的实例的返回值，会传递给`p`的回调函数。

##6. Promise.race()

同样是将多个Promise实例，包装成一个新的Promise实例。

```javascript
var p=Promise.race([p1,p2,p3]);
```

上面的栗子中，只要p1,p2,p3中有一个改变状态，p的状态就跟着改变，率先改变的值会传给p的回调函数。

##7. Promise.resolve()


