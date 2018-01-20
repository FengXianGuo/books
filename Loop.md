---
# 图解搞懂JavaScript引擎Event Loop
---

![](https://user-gold-cdn.xitu.io/2018/1/20/16112dee30db2997?w=2044&h=1538&f=jpeg&s=1121386)

# 一，js单线程存在的问题

js是单线程的，处理任务是一件接着一件处理，所以如果一个任务需要处理很久的话，后面的任务就会被阻塞
a
所以js通过Event Loop事件循环的方式解决了这个问题,在了解事件循环前，我们需要了解一些关键词

# 二,什么是stack，queue，heap，event loop

- stack（栈）：吃多了吐
- queue（队列）：吃多了...释放
- heap(堆)：存储obj对象

## 执行栈

![](https://user-gold-cdn.xitu.io/2018/1/20/16112dee3088cd86?w=238&h=460&f=jpeg&s=43149)

js引擎运行时，当代码开始运行的时候，会将代码，压入执行栈进行执行

例：
![](https://user-gold-cdn.xitu.io/2018/1/20/16112dee2e9050a2?w=352&h=428&f=jpeg&s=42111)

当代码被解析后，函数会依次被压入到栈中

![](https://user-gold-cdn.xitu.io/2018/1/20/16112dee32bc6cfe?w=1954&h=1064&f=jpeg&s=393086)

有入栈，就要有出栈，当函数c执行完，开始出栈

![](https://user-gold-cdn.xitu.io/2018/1/20/16112dee309510b6?w=1926&h=1200&f=jpeg&s=497342)

## 当执行栈遇到异步

前面执行栈，先入后出，但其实也是同步的，同步就意味着会阻塞，所以需要异步，那当执行栈中出现异步代码会怎么样

![](https://user-gold-cdn.xitu.io/2018/1/20/16112dee354ad411?w=582&h=300&f=jpeg&s=65746)

此时在代码中，添加点击事件和setTimeout，现在观察一下执行顺序

![](https://user-gold-cdn.xitu.io/2018/1/20/16112dee5d26cc3e?w=1650&h=828&f=jpeg&s=268117)

观察此时的执行栈效果，和上面的函数嵌套有显著区别

1，console.log("sync")的语句，不会被压入到执行栈底部，因为console已经执行结束了

2，click和settimeout都入栈了，但它们内部的console没有入栈的，这说明他们没有执行完

3，如果click没有执行完，那为什么setTimeout会入栈，不应该被阻塞吗？

答案是：当浏览器在执行栈执行的时候，发现有异步任务之后，会交给webapi去维护，而执行栈则继续执行后面的任务

![](https://user-gold-cdn.xitu.io/2018/1/20/16112dee60328fce?w=1894&h=786&f=jpeg&s=213852)

同样，setTimeout同样会被添加到webapi中

![](https://user-gold-cdn.xitu.io/2018/1/20/16112dee5e3f8985?w=1854&h=728&f=jpeg&s=185357)

webapi是浏览器自己实现的功能，这里专门维护事件。

上面setTimeout旁边有个进度条，这个进度就是设置的等待时间

## 回调队列callback queue

上面的例子，当setTimeout执行结束的时候，是不是就应该回到执行栈，进行执行输出呢？

答案：并不是！

![](https://user-gold-cdn.xitu.io/2018/1/20/16112dee84e4ae06?w=1930&h=1166&f=jpeg&s=253360)

此时，倒计时结束后的setTimeout的可执行函数，被放如了回调队列

最后，setTimeout的可执行函数，被从回调队列中取出，再此放入了执行栈

![](https://user-gold-cdn.xitu.io/2018/1/20/16112dee629756d7?w=1770&h=1046&f=jpeg&s=258631)

这样的执行过程就叫 event loop事件循环

# Event Loop的具体流程

## 执行栈任务清空后，才会从回调队列头部取出一个任务

![](https://user-gold-cdn.xitu.io/2018/1/20/16112dee6092ba02?w=430&h=216&f=jpeg&s=34590)

上面是一个最简单的例子，输出结果是1，3，2

这是为什么？

![](https://user-gold-cdn.xitu.io/2018/1/20/16112dee87d916c3?w=2038&h=898&f=jpeg&s=251043)

上图展示了具体的执行顺序：

1，console.log（1）被压入执行栈

2，setTimeout在执行栈被识别为异步任务，放入webapis中

3，console.log（3）被压入执行栈，此时setTimeout的可执行代码还在回调队列里等待

4，console.log（3)执行完成后，从回调队列头部取出console.log(2)，放入执行栈

5，console.log(2)执行

## 回调队列先进先出

需要格外注意，回调队列是先进先出的，例：
![](https://user-gold-cdn.xitu.io/2018/1/20/16112dee8a003842?w=392&h=336&f=jpeg&s=57799)

![](https://user-gold-cdn.xitu.io/2018/1/20/16112dee9363401d?w=1946&h=1142&f=jpeg&s=329869)

当console.log(4)执行完成后，从回调队列里取出了console.log(2);

**注意：只有console.log(2)执行完成，执行栈再次清空时，才会从回调队列取出console.log(3)**

## 测试概念是否正确

![](https://user-gold-cdn.xitu.io/2018/1/20/16112dee91ffd725?w=448&h=478&f=jpeg&s=83848)

上面的代码最后输出1，5，2，4，3，执行过程：

1，输出1，将2push进回调队列

2，将4push进回调队列

3，输出5

4，清空了执行栈，读取输出2，发现有3，将3push进回调队列

5，清空了执行栈，读取输出4

6，清空了执行栈，读取输出3

**至此，看起来好像没问题了，但是！！！！！！，事情还没有结束**

# Macrotask(宏任务)、Microtask（微任务） 

通过上面的例子，想必已经对event loop有了一定的了解，现在继续看一个例子

```
console.log(1);
setTimeout(()=>{
	console.log(2)
})
var p = new Promise((resolve,reject)=>{
	console.log(3)
	resolve("成功")
})
p.then(()=>{
	console.log(4)
})
console.log(5)
```

按照event loop的概念，应该是1，3，5，2，4，因为setTimeout和then会被放到回调队列里，然后又是先进先出，所以应该是2先输出，4后输出

但事实输出的顺序是1，3，5，4，2！

![](https://user-gold-cdn.xitu.io/2018/1/20/16112deeb4972f6a?w=628&h=572&f=jpeg&s=66180)


这是因为promise的then方法，被认为是在Microtask微任务队列当中

## 什么是Macrotask(宏任务)

Macrotask(宏任务)很好理解，就是咱们前面介绍过的回调队列callback queue

## 什么是Microtask（微任务）

Microtask(微任务)同样是一个任务队列，**这个队列的执行顺序是在清空执行栈之后**

用图展示就是

![](https://user-gold-cdn.xitu.io/2018/1/20/16112dee9df2a709?w=1792&h=710&f=jpeg&s=465709)

可以看到Macrotask(宏任务)也就是回调队列上面还有一个Microtask（微任务）

Microtask（微任务）虽然是队列，**但并不是一个一个放入执行栈**，而是当执行栈请空，会执行全部Microtask（微任务）队列中的任务，最后才是取回调队列的第一个Macrotask(宏任务)

例：

![](https://user-gold-cdn.xitu.io/2018/1/20/16112deeb63c271f?w=682&h=542&f=jpeg&s=64607)

上面的执行过程是：

1，将setTimeout给push进宏任务

2，将then(2)push进微任务

3，将then(4)push进微任务

4，任务队列为空，取出微任务第一个then(2)压入执行栈

5，输出2，将then(3)push进微任务

6，任务队列为空，取出微任务第一个then(4)压入执行栈

7，输出4

8，任务队列为空，取出微任务第一个then(3)压入执行栈

9，输出3

10，任务队列为空，微任务也为空，取出宏任务中的setTimeout（1）

11，输出1

## 为什么then是微任务

这和每个浏览器有关，每个浏览器实现的promise不同，有的then是宏任务，有的是微任务，chrome是微任务，普遍都默认为微任务

除了then以外，还有几个事件也被记为微任务：

- process.nextTick
- promises
- Object.observe
- MutationObserver

```
console.log("start");
setImmediate(()=>{
    console.log(1)
})
Promise.resolve().then(()=>{
    console.log(4);
})
Promise.resolve().then(()=>{
    console.log(5);
})
process.nextTick(function foo() {
    console.log(2);
});
process.nextTick(function foo() {
    console.log(3);
});
console.log("end")
```

上面代码输出start,end,2,3,4,5,1

process.nextTick的概念和then不太一样，process.nextTick是加入到执行栈底部，所以和其他的表现并不一致



## 最后的测试

```
console.log("1");
setTimeout(()=>{
    console.log(2)
    Promise.resolve().then(()=>{
        console.log(3);
        process.nextTick(function foo() {
            console.log(4);
        });
    })
})
Promise.resolve().then(()=>{
    console.log(5);    
    setTimeout(()=>{
        console.log(6)
    })
    Promise.resolve().then(()=>{
        console.log(7);
    })
})

process.nextTick(function foo() {
    console.log(8);
    process.nextTick(function foo() {
        console.log(9);
    });
});
console.log("10")
```
执行顺序：

1，输出1

2，将setTimeout（2）push进宏任务

3，将then（5）push进微任务

4，在执行栈底部添加nextTick（8）

5，输出10

6，执行nextTick（8）

7，输出8

8，在执行栈底部添加nextTick（9）

9，输出9

10，执行微任务then（5）

11，输出5

12，将setTimeout（6）push进宏任务

13，将then（7）push进微任务

14，执行微任务then（7）

15，输出7

16，取出setTimeout（2）

17，输出2

18，将then（3）push进微任务

19，执行微任务then（3）

20，输出3

21，在执行栈底部添加nextTick（4）

22，输出4

23，取出setTimeout（6）

24，输出6

最后结果是：1，10，8，9，5，7，2，3，4，6



# 参考
- [并发模型与事件循环](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop)
- [youtube视频](https://2014.jsconf.eu/speakers/philip-roberts-what-the-heck-is-the-event-loop-anyway.html)

