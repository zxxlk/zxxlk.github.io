---
title: Promise
date: 2021-03-19 16:42:18
tags:
---


## *Promise*

 ***1. Promise	的含义***

Promise是异步编程的一种解决方案，为了解决“回调地狱”问题，提出Promise对象，并且后来加入了ES6标准。Promise对象简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。从语法上说，Promise 是一个对象，从它可以获取异步操作的消息。
`Promise`对象有以下两个特点。

（1）对象的状态不受外界影响。`Promise`对象代表一个异步操作，有三种状态：`pending`（进行中）、`fulfilled`（已成功）和`rejected`（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。这也是`Promise`这个名字的由来，它的英语意思就是“承诺”，表示其他手段无法改变。

（2）一旦状态改变，就不会再变，任何时候都可以得到这个结果。`Promise`对象的状态改变，只有两种可能：从`pending`变为`fulfilled`和从`pending`变为`rejected`。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果，这时就称为 resolved（已定型）。如果改变已经发生了，你再对`Promise`对象添加回调函数，也会立即得到这个结果。这与事件（Event）完全不同，事件的特点是，如果你错过了它，再去监听，是得不到结果的。

 ***2. 基本用法***

Promise 对象的构造器（constructor）语法如下：

  

```javascript
  let promise =  new  Promise(function(resolve, reject)  { 
	     // executor（生产者代码） 
    });
```

传递给 `new Promise` 的函数被称为 **executor**。当 `new Promise` 被创建，executor 会自动运行。它包含最终应产出结果的生产者代码。

它的参数  `resolve`  和  `reject`  是由 JavaScript 自身提供的回调。我们的代码仅在 executor 的内部。

当 executor 获得了结果，无论是早还是晚都没关系，它应该调用以下回调之一：

-   `resolve(value)`  — 如果任务成功完成并带有结果  `value`。
-   `reject(error)`  — 如果出现了 error，`error`  即为 error 对象。

所以总结一下就是：executor 会自动运行并尝试执行一项工作。尝试结束后，如果成功则调用  `resolve`，如果出现 error 则调用  `reject`。这两个函数都是可选的，不一定要提供。它们都接受`Promise`对象传出的值作为参数。并且，`resolve/reject` 只需要一个参数（或不包含任何参数），并且将忽略额外的参数。

由  `new Promise`  构造器返回的  `promise`  对象具有以下内部属性：


-   `state`  — 最初是  `"pending"`，然后在  `resolve`  被调用时变为  `"fulfilled"`，或者在  `reject`  被调用时变为  `"rejected"`。
-   `result`  — 最初是  `undefined`，然后在  `resolve(value)`  被调用时变为  `value`，或者在  `reject(error)`  被调用时变为  `error`。

下面是一个`Promise`对象的简单例子。

```javascript
function asyncFunction() {
    return new Promise(function (resolve, reject) {
        setTimeout(function () {
            resolve('Async Hello world');
        }, 16);
    });
}
    
asyncFunction().then(function (value) {
	 	console.log(value);    	// => 'Async Hello world'
	}).catch(function (error) {
	 	console.log(error);
});
```

`asyncFunction` 这个函数会返回promise对象，该promise对象会在setTimeout之后的16ms时被resolve, 这时 `then` 的回调函数会被调用，并输出 `'Async Hello world'` 。在这种情况下 `catch` 的回调函数并不会被执行（因为promise返回了resolve）， 不过如果运行环境没有提供 `setTimeout` 函数的话，那么上面代码在执行中就会产生异常，在 `catch` 中设置的回调函数就会被执行。

 ***3. Promise 链***
 
 连续执行两个或者多个异步操作是一个常见的需求，在上一个操作执行成功之后，开始下一个的操作，并带着上一步操作所返回的结果。我们可以通过创造一个 **Promise 链**来实现这种需求。

```javascript
  new  Promise(function(resolve, reject)  {  
	    setTimeout(()  =>  resolve(1),  1000);  // (*)  
    }).then(function(result)  {  // (**) 
		 alert(result);  // 1 
	     return result *  2;  
     }).then(function(result)  {  
	     alert(result);  // 2  
	     return result *  2;  
     }).then(function(result)  {  
	     alert(result);  // 4  
	     return result *  2;  
     });
```

  它的理念是将 result 通过  `.then`  处理程序（handler）链进行传递。

运行流程如下：
1.  初始 promise 在 1 秒后进行 resolve  `(*)`，
2.  然后  `.then`  处理程序（handler）被调用  `(**)`。
3.  它返回的值被传入下一个  `.then`  处理程序（handler）`(***)`
4.  ……依此类推。

随着 result 在处理程序（handler）链中传递，我们可以看到一系列的 `alert` 调用：`1` → `2` → `4`。
之所以这么运行，是因为对 `promise.then` 的调用会返回了一个 promise，所以我们可以在其之上调用下一个 `.then`。

**新手常犯的一个经典错误：从技术上讲，我们也可以将多个  `.then`  添加到一个 promise 上。但这并不是 promise 链（chaining）。**

例如：

   

```javascript
 let promise =  new  Promise(function(resolve, reject)  { 
	    setTimeout(()  =>  resolve(1),  1000);  
    }); 
     promise.then(function(result)  {  
	     alert(result);  // 1  
	     return result *  2; 
     }); 
     promise.then(function(result)  {  
	     alert(result);  // 1  
	     return result *  2;  
	 }); 
     promise.then(function(result)  {  
	     alert(result);  // 1  
	     return result *  2;  
	 });
```

在这里所做的只是一个 promise 的几个处理程序（handler）。它们不会相互传递 result；相反，它们之间彼此独立运行处理任务。

在同一个 promise 上的所有 `.then` 获得的结果都相同 — 该 promise 的结果。所以，在上面的代码中，所有 `alert` 都显示相同的内容：`1`。

 ***4. 错误处理***
 
 Promise 链在错误（error）处理中十分强大。当一个 promise 被 reject 时，控制权将移交至最近的 rejection 处理程序（handler）。`.catch` 不必是立即的。它可能在一个或多个 `.then` 之后出现。或者，如下示例：可能该网站一切正常，但响应不是有效的 JSON。捕获所有 error 的最简单的方法是，将 `.catch` 附加到链的末尾：

```javascript
    fetch('/article/promise-chaining/user.json')  
	    .then(response  => response.json())  
	    .then(user  =>  fetch(`https://api.github.com/users/${user.name}`))  
	    .then(response  => response.json())  
	    .then(githubUser  =>  new  Promise((resolve, reject)  =>  {  
		    let img = document.createElement('img'); 
		    img.src = githubUser.avatar_url; 
		    img.className =  "promise-avatar-example"; 
		    document.body.append(img);  
		    setTimeout(()  =>  { 
			    img.remove(); 
			    resolve(githubUser);  
		    },  3000);  
	    }))  
    .catch(error => alert(error.message));
```

通常情况下，这样的 `.catch` 根本不会被触发。但是如果上述任意一个 promise 被 reject（网络问题或者无效的 json 或其他），`.catch` 就会捕获它。
 -   `.catch`  处理 promise 中的各种 error：在  `reject()`  调用中的，或者在处理程序（handler）中抛出的（thrown）error。
 -   我们应该将  `.catch`  准确地放到我们想要处理 error，并知道如何处理这些 error 的地方。处理程序应该分析 error（可以自定义 error 类来帮助分析）并再次抛出未知的 error（可能它们是编程错误）。
 -   如果没有办法从 error 中恢复的话，不使用  `.catch`  也可以。
 -   在任何情况下我们都应该有  `unhandledrejection`  事件处理程序（用于浏览器，以及其他环境的模拟），以跟踪未处理的 error 并告知用户（可能还有我们的服务器）有关信息，以使我们的应用程序永远不会“死掉”。
 
 ***5. Promise API***
 
在 `Promise` 类中，有 5 种静态方法。

 **- Promise.all**
 
 假设我们希望并行执行多个 promise，并等待所有 promise 都准备就绪。例如，并行下载几个 URL，并等到所有内容都下载完毕后再对它们进行处理。这就是  `Promise.all`  的用途。

语法： 


```javascript
 let promise = Promise.all([...promises...]);
```

Promise.all 接受一个 promise 数组作为参数（从技术上讲，它可以是任何可迭代的，但通常是一个数组）并返回一个新的 promise。

当所有给定的 promise 的状态都变为 fulfilled 时，新的 promise 才会 resolve，并且其结果数组将成为新的 promise 的结果。

例如，下面的 Promise.all 在 3 秒之后被 settled，然后它的结果就是一个 [1, 2, 3] 数组：

```javascript
Promise.all([
  new Promise(resolve => setTimeout(() => resolve(1), 3000)), // 1
  new Promise(resolve => setTimeout(() => resolve(2), 2000)), // 2
  new Promise(resolve => setTimeout(() => resolve(3), 1000))  // 3
]).then(alert); // 1,2,3 当上面这些 promise 准备好时：每个 promise 都贡献了数组中的一个元素
```
请注意，结果数组中元素的顺序与其在源 promise 中的顺序相同。即使第一个 promise 花费了最长的时间才 resolve，但它仍是结果数组中的第一个。

一个常见的技巧是，将一个任务数据数组映射（map）到一个 promise 数组，然后将其包装到 Promise.all。

如果任意一个 promise 被 reject，由 Promise.all 返回的 promise 就会立即 reject，并且带有的就是这个 error。

例如：

```javascript
Promise.all([
  new Promise((resolve, reject) => setTimeout(() => resolve(1), 1000)),
  new Promise((resolve, reject) => setTimeout(() => reject(new Error("Whoops!")), 2000)),
  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000))
]).catch(alert); // Error: Whoops!
```

这里的第二个 promise 在两秒后 reject。这立即导致了 Promise.all 的 reject，因此 .catch 执行了：被 reject 的 error 成为了整个 Promise.all 的结果。

> 如果出现 error，其他 promise 将被忽略。 如果其中一个 promise 被 reject，Promise.all 就会立即被 reject，完全忽略列表中其他的 promise。它们的结果也被忽略。
> 
> 例如，像上面那个例子，如果有多个同时进行的 fetch 调用，其中一个失败，其他的 fetch 操作仍然会继续执行，但是
> Promise.all 将不会再关心（watch）它们。它们可能会 settle，但是它们的结果将被忽略。
> 
> Promise.all 没有采取任何措施来取消它们，因为 promise 中没有“取消”的概念。

 **- Promise.allSettled**
 
Promise.allSettled 等待所有的 promise 都被 settle，无论结果如何。结果数组具有：

 - {status:"fulfilled", value:result} 对于成功的响应，
 - {status:"rejected",   reason:error} 对于 error。

例如，我们想要获取（fetch）多个用户的信息。即使其中一个请求失败，我们仍然对其他的感兴趣。

让我们使用 Promise.allSettled：

```javascript
let urls = [
  'https://api.github.com/users/iliakan',
  'https://api.github.com/users/remy',
  'https://no-such-url'
];

Promise.allSettled(urls.map(url => fetch(url)))
  .then(results => { // (*)
    results.forEach((result, num) => {
      if (result.status == "fulfilled") {
        alert(`${urls[num]}: ${result.value.status}`);
      }
      if (result.status == "rejected") {
        alert(`${urls[num]}: ${result.reason}`);
      }
    });
  });
```
上面的 (*) 行中的 results 将会是：

```javascript
[
  {status: 'fulfilled', value: ...response...},
  {status: 'fulfilled', value: ...response...},
  {status: 'rejected', reason: ...error object...}
]
```
所以，对于每个 promise，我们都得到了其状态（status）和 value/reason。

 **- Promise.race**

与 Promise.all 类似，但只等待第一个 fulfilled 的 promise 并获取其结果（或 error）。

```javascript
Promise.race([
  new Promise((resolve, reject) => setTimeout(() => resolve(1), 1000)),
  new Promise((resolve, reject) => setTimeout(() => reject(new Error("Whoops!")), 2000)),
  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000))
]).then(alert); // 1
```
这里第一个 promise 最快，所以它变成了结果。第一个 fulfilled 的 promise “赢得了比赛”之后，所有进一步的 result/error 都会被忽略。

 **- Promise.resolve**

Promise.resolve(value) 用结果 value 创建一个 resolved 的 promise。有时需要将现有对象转为 Promise 对象，Promise.resolve()方法就起到这个作用。

```javascript
const jsPromise = Promise.resolve($.ajax('/whatever.json'));
```
上面代码将 jQuery 生成的deferred对象，转为一个新的 Promise 对象。

 **- Promise.reject**

Promise.reject(reason)方法也会返回一个新的 Promise 实例，该实例的状态为rejected。

```javascript
const p = Promise.reject('出错了');
// 等同于
const p = new Promise((resolve, reject) => reject('出错了'))

p.then(null, function (s) {
  console.log(s)
});
// 出错了
```
上面代码生成一个 Promise 对象的实例p，状态为rejected，回调函数会立即执行。