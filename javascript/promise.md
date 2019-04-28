## Promise

------

Promise  对象用于表示一个异步操作的最终状态（完成或失败），以及其返回的值。

## 语法：

```javascript
new Promise(function(resolve, reject){...});
```
## Promise 对象的三种状态：

- pending   初始状态，既不是成功，也不是失败状态。
- fulfilled(resolved)  意味着操作成功完成;   
- rejected    意味着操作失败;   调用 reject 或 throw error

状态间转化： pending ->  fulfilled;  pending -> rejected;

下面代码来描述当前状态：

```javascript
{
    let num = Math.floor(Math.random() * 10) % 2; 
    let promise = new Promise(function(resolve, reject){
      	// 当前状态为 ： pending
        if (num) {
            reject(); // 执行这句时，状态为 pending->rejected
        } else {
            resolve(); // 执行这句时，状态为 pending->fulfilled
        }
    })
}
```

![](F:\kkb-notes\javascript\promises.png)

## 类方法

Promise.all(values)

> @params values 是 Promise对象的集合
> @return  返回是一个新的 Promise 对象
> Promise.all(values)	

all 表示所有，也就当values中所有promise对象都成功（状态为：fulfilled）的时候才会触发成功，一旦有任何一个values里面的 promise 对象失败则立刻触发该 promise 对象的失败。

其次返回新 promise 状态由 values 中 promise对象状态决定；

如果触发**成功状态**（fulfilled），会把一个包含values里所有promise的返回值的数组作为成功回调的返回值，顺序跟values的顺序保持一致

如果触发**失败状态**（rejected），会把 values 里第一个触发失败的 promise 对象的错误信息作为它的失败错误信息

```javascript
// 都成功 
var promise1 = new Promise(function(resolve, reject){
    console.log('promise1');
    setTimeout(function(){
        resolve(1);
    }, 200)
});
var promise2 = new Promise(function(resolve, reject){
    console.log('promise2');
    setTimeout(function(){
        resolve(2);
    }, 200)
});
var promise3 = new Promise(function(resolve, reject){
    console.log('promise3');
    setTimeout(function(){
        resolve(3);
    }, 200)
});


Promise.all([
 	promise1,
 	promise2,
 	promise3
]).then(function (values) {
 	console.log(values); 
}, function (error) {
 	console.log(error);
})；
// promise1 
// promise2
// promise3 
// [1, 2, 3]

// 更改一个
promise2 = new Promise(function(resolve, reject){
    console.log('promise2');
    setTimeout(function(){
        reject('error');
    }, 200)
});

// 任何一个失败
Promise.all([
 	promise1,
 	promise2,
 	promise3
]).then(function (values) {
 	console.log(values);
}, function (error) {
 	console.log(error);
})；

// promise1 
// promise2
// promise3 
// error
```

Promise.race(values)

> Creates a Promise that is resolved or rejected when any of the provided Promises are resolved
> or rejected.
> @param values An array of Promises.
> @returns A new Promise.

这方法与 all 恰恰相反；当 values 中任何一个 promise 对象状态为成功或失败就返回； 其中该方法返回一个新的 promise 对象， 该新对象的状态可能 resolved  或 rejected ； 

```javascript
var promise1 = new Promise(function(resolve, reject){
    console.log('promise1');
    setTimeout(function(){
        resolve(1);
    }, 200)
});
var promise2 = new Promise(function(resolve, reject){
    console.log('promise2');
    setTimeout(function(){
        resolve(2);
    }, 200)
});
var promise3 = new Promise(function(resolve, reject){
    console.log('promise3');
    setTimeout(function(){
        resolve(3);
    }, 200)
});
Promise.race([
      promise1,
      promise2,
      promise3
  ]).then(function (value) {
  	console.log(value);
  }, function (error) {
  	console.log(error);
  });
// promise1
// promise2
// promise3
// 1
```

Promise.reject(reason)

>根据提供原因创建一个失败Promise对象
>@param   reason 表示失败的原因
>@returns 返回状态(rejected)为失败的 Promise 对象
>Promise.reject('请求异常');

该方法返回状态为失败的 promise 对象；

```javascript
// 一般常见应用场景


```

Promise.resolve(value | PromiseLike)

>根据提供值返回一个 promise 对象;
>@param value | PromiseLike
>@returns 返回一个状态由给定value决定的 Promise 对象

根据 value 几种情况：

1.  value 是一个 Promise 对象,则直接返回该对象;
2.  如果该值是thenable(即，带有then方法的对象)，返回的Promise对象的最终状态由then方法执行决定;
3. 如果该value为空，基本类型或者不带then方法的对象,返回的Promise对象状态为fulfilled，并且将该value传递给对应的then方法。

```javascript
// 验证第一种情况：
{
    let value = new Promise(function (resolve, reject) {
        setTimeout(function () {
            resolve('成功');
        })
    });
    let nowPromise = Promise.resolve(value);
    nowPromise.then((message) => {}, (err) => {});
    console.log("如果传入Promise对象,直接返回改对象;" + (value === nowPromise));   //=> true
}
```

```javascript
// 验证第二种情况：
let value2 = new Promise(function (resolve, reject) {
   setTimeout(function () {
      resolve('成功'); // reject('失败');
   }, 200)
}).then(function(){
    return Promise.resolve(true);  // 决定nowPromise状态为fulfilled
}, function(){
    return Promise.reject(false); //  决定nowPromise状态为rejected
});
var nowPromise = Promise.resolve(value2);
console.log(nowPromise); // true
```

```javascript
// 验证第三种情况
let value3 = '验证第三种方式';  // 基本数据类型
let promise3 = Promise.resolve(value3);
console.log('promise3', promise3); 
// promise3  状态为： fulfilled
```

------



## Promise 原型  

------

**属性**

`Promise.prototype.constructor`

返回被创建的实例函数.  默认为 Promise 函数.

**方法**

`Promise.prototype.catch(onRejected)`

捕获错误处理，并返回一个新的 promise。 当这个回调函数被调用，新 promise 将以它的返回值（onRejected 函数中的 return 值， 默认函数返回值为 undefined）来 resolve

```javascript
// 描述第一句话
var promise = new Promise(function (resolve, reject){
	reject(new Error('自定义错误')); 
    // 或 throw new Error('成功时返回值')
});
var nowPromise = promise.catch(function(err){
	return err;
});
nowPromise.then(function({message}){
    console.log(message); // 自定义错误
}, function(error){
    console.log(error);
});
```

​	进入catch两种方式：手动调用 reject 或 程序出现异常

Promise.prototype.then(onFulfilled, onRejected)

方法返回一个新的 promise 。最多接收两个参数：Promise 的成功和失败情况的回调函数。 新的promise对象根据回调函数的返回值来 resolve;

Promise.prototype.finally(onFinally)

 方法返回一个Promise，在执行then() 和 catch() 后，都会执行**finally**指定的回调函数。这样可以有效避免同样的语句需要在then() 和 catch() 中各写一次的情况。

```javascript
var nowPromise = new Promise(function(resolve, reject){
   resolve(); // reject(); 
}).then(function(){
	console.log('then');
}).catch(function(){
	console.log('catch');
}).finally(function(){
	console.log('finally');
});
// 可能打印的输出

// catch
// finally

//或

// then
// finally
```