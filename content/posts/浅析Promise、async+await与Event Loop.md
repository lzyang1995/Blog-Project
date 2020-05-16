---
title: "浅析Promise、async/await与Event Loop"
date: 2020-05-14T11:33:21+08:00
draft: false
tags: ["JavaScript", "Promise", "Event Loop"]
categories: ["JavaScript"]
lightgallery: true
---

JavaScript引擎是基于Event Loop来运行的，不断处理队列中的各项任务。Event Loop包含一个调用栈（Stack），若干个任务队列（Queue）和一个堆（Heap）。它的工作模式就是不断从任务队列中取出任务放入栈中执行，一次只执行一个任务，当前任务执行完之后再从任务队列中取出下一个任务开始执行。当我们把Promise以及async/await也考虑进Event Loop时，各项任务的执行顺序就变得稍显复杂。

<!--more-->

这里谈谈我在查阅一些资料之后对于它们的理解。

## 宏任务（Macrotask）

在Event Loop中，其中一个任务队列被称为宏任务队列（Macrotask Queue），用于存放各种宏任务。属于宏任务的主要有以下几种：

+ JS脚本
+ setTimeout、setInterval中的callback函数
+ 用于处理各种事件（如click、mousemove）的handler函数

一个比较经典的关于Event Loop和宏任务的问题就是setTimeout中的callback的执行顺序问题。例如下面的代码：

```js
console.log(0);
setTimeout(() => console.log(1), 0);
console.log(2);
```

该代码的输出顺序应该是0, 2, 1。这里虽然setTimeout的第二个参数被设置为0，但其中的callback仍然在`console.log(2)`之后执行，因为当前栈中的任务是上面的脚本，setTimeout中的callback是被放入了宏任务队列中，只有在上面的脚本执行结束之后，这个callback才会被放入栈中执行。具体的执行步骤如下：

1. 上述脚本被放入栈中
2. 执行`console.log(0)`，输出0
3. 执行setTimeout，开启一个线程用于计时。由于延时被设置为0，因此计时立刻结束，callback被放入宏任务队列中
4. 执行`console.log(2)`，输出2
5. 脚本执行完毕，callback被放入栈中开始执行，输出1

## Promise与Event Loop

对于Promise，首先其构造函数中的参数（`function (resolve, reject) {}`）是同步执行的，即在构造Promise时会立即执行，执行完毕后才会返回构造好的Promise对象。另外，在Promise settle之后，then/catch/finally中的handler函数会被放入另一个任务队列，称为微任务队列（Microtask Queue）。JS引擎在执行完当前的宏任务之后，会先把微任务队列中的所有任务都执行完毕（包括执行过程中新加入的微任务），再去执行宏任务队列中的下一个宏任务。除了Promise的handler函数之外，我们还可以通过`queueMicrotask`将一个函数放入微任务队列中。

对于下面的例子：

```js
console.log(0);
Promise.resolve().then(() => console.log(1)).then(() => console.log(2));
setTimeout(() => console.log(3), 0);
console.log(4);
```

上述代码的输出为0, 4, 1, 2, 3。具体的步骤如下：

1. 上述脚本被放入栈中
2. 执行`console.log(0)`，输出0
3. `Promise.resolve()`返回一个fulfilled promise，因此函数`() => console.log(1)`被放入微任务队列中
4. 执行setTimeout，函数`() => console.log(3)`被放入宏任务队列中
5. 执行`console.log(4)`，输出4，上述脚本执行完毕
6. 从微任务队列中取出函数`() => console.log(1)`并执行，输出1。该函数相当于返回一个fulfilled promise，因此`() => console.log(2)`又被
放入微任务队列中
7. 从微任务队列中取出函数`() => console.log(2)`并执行，输出2
8. 微任务队列已经清空，从宏任务队列中取出函数`() => console.log(3)`并执行，输出3

我们再来看一个例子：

```js
console.log(0);
new Promise(function(resolve, reject) {
    console.log(1);
    setTimeout(() => console.log(2), 0);
    resolve();
}).then(() => console.log(3));
console.log(4);
```

这个代码的输出将会是0, 1, 4, 3, 2。具体步骤如下：

1. 上述脚本被放入栈中
2. 执行`console.log(0)`，输出0
3. 执行Promise构造函数中的内容。首先执行`console.log(1)`，输出1；然后执行setTimeout，将`() => console.log(2)`放入宏任务队列中；
最后执行resolve，使得该promise变成一个fulfilled promise。
4. 由于promise已经fulfilled，函数`() => console.log(3)`被放入微任务队列中
5. 执行`console.log(4)`，输出4，上述脚本执行完毕
6. 从微任务队列中取出函数`() => console.log(3)`并执行，输出3
7. 微任务队列已经清空，从宏任务队列中取出函数`() => console.log(2)`并执行，输出2

## async/await与Event Loop

async/await被称作是Promise的语法糖，所以要分析包含async/await的代码的执行顺序，我们需要理解async/await内部是如何通过Promise来实现的。下面谈谈我在查阅一些资料后的个人理解。

对于下面的async函数的伪代码：

```js
async function func() {
    doSomething();
    let result = await A;
    processResult(result);
}
```

实际上`await`后面的内容相当于Promise A的handler函数。也就是说，上面的代码等价于：

```js
async function func() {
    doSomething();
    A.then((result) => {
        processResult(result);
    });
}
```

因此，当我们的调用栈进入async函数内部，遇到await之后，会把await右边的Promise中的同步内容执行完，然后退出该async函数去执行该函数后面的同步内容。如果await右侧的Promise resolve了，那么该async函数剩下的部分会被放入微任务队列中。

例如，对于下面的代码：

```js
async function func1() {
    console.log(1);
    await func2();
    console.log(2);
}

async function func2() {
    console.log(3);
}

console.log(0);
func1().then(() => console.log(4));
setTimeout(() => console.log(5), 0)
console.log(6);
```

其输出顺序将会是0, 1, 3, 6, 2, 4, 5。具体分析如下：

1. 上述脚本被放入栈中
2. 执行`console.log(0)`，输出0
3. 进入func1函数，执行`console.log(1)`，输出1
4. 进入func2函数，执行`console.log(3)`，输出3，并返回一个fulfilled promise（值为undefined）。func1函数await后面的部分被放入微任务队列中。
5. 执行setTimeout，将函数`() => console.log(5)`放入宏任务队列中
6. 执行`console.log(6)`，输出6。脚本结束，调用栈清空
7. 从微任务队列中取出func1函数await后面的部分开始执行，输出2。由于func1返回了一个fulfilled promise（值为undefined），因此函数`() => console.log(4)`被放入微任务队列中
8. 从微任务队列中取出函数`() => console.log(4)`执行，输出4
9. 微任务队列为空，从宏任务队列中取出`() => console.log(5)`并执行，输出5

## 参考资料

1. <https://segmentfault.com/a/1190000017554062>
2. <https://juejin.im/post/5c148ec8e51d4576e83fd836>
3. <https://www.cnblogs.com/beyondcheng/p/3440154.html>
4. <https://juejin.im/post/5c9a43175188252d876e5903>
5. <https://javascript.info/microtask-queue>
6. <https://javascript.info/event-loop>
