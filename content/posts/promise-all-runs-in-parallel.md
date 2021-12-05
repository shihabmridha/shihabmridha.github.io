+++
author = "Dijkstra"
categories = ["JavaScript", "Concurrency", "Parallelism"]
date = 2019-10-30T07:37:34Z
description = "Things will run concurrently in single-core CPU and will run in parallel in multi-core CPU. It's main task is to wait until all the promises that are passed to it are resolved."
draft = false
slug = "promise-all-runs-in-parallel"
summary = "Things will run concurrently in single-core CPU and will run in parallel in multi-core CPU. It's main task is to wait until all the promises that are passed to it are resolved."
tags = ["JavaScript", "Concurrency", "Parallelism"]
title = "Does Promise.all() run tasks in parallel?"

+++


**_TL; DR_**

**No!** More accurately, it **depends on various factors**.

---

The promise API is a blessing for JavaScript developers. It makes writing asynchronous JavaScript a lot easier. Go to hell **callback**! Like some other JavaScript concepts promise confuses a lot of people. I am not going to talk about promise in this article. There are tons of articles related to promise.

There is a misconception about **[concurrency](https://en.wikipedia.org/wiki/Concurrency_(computer_science))** and **parallelism** in the JavaScript world. When the promise got introduced people started to think that now everything is now going to run in parallel. I have found that this is not completely true.

### Let's talk about some **important fundamentals** (briefly)
A single-core CPU can not perform multiple tasks at the same time. So, it means it is not possible to get parallelism in a single-core CPU. Then how does my single-core CPU perform multiple tasks at the same time? Well, there are a lot of things going on in the operating system but there are two main concepts. One is [multi-threading](https://en.wikipedia.org/wiki/Multithreading_(computer_architecture)) and another one is [context switching](https://en.wikipedia.org/wiki/Context_switch). Multiple tasks are handled by multiple threads. What happens is the CPU performs one task (using a thread) for a certain time and then pause the current task and move on to the next task (using another thread) and start processing it. This switching happens so fast that we think everything is running in parallel even though we are using a single-core CPU.
But, the modern computer has dedicated hardware for I/O (network, file read/write, etc). There is a concept called DMA (direct memory access) that allows the computer to perform I/O without blocking the CPU. It means, when we are dealing with network or reading/writing in the SSD/HDD, our CPU is mostly sitting idle. It is the fundamental concept behind the creation of Node.JS. Now, if we think about it we can achieve parallelism where CPU would do one thing at the same time I/O would do something else. 

> A single-core CPU can not perform multiple tasks at the same time.

### Writing `async` infront of a function does not make it asynchronous
It is very important to understand that writing [`async` infront of a function or returning a promise from a function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) does not make the function nonblocking/asynchronous. See the example below:

```javascript
async function isPrime(num) {
  for(var i = 2; i < num; i++)
    if(num % i === 0) return false;
  return num > 1;
}

var isItPrime = await isPrime(89098977); // Blocking operation
```
The function above returns a promise but it is a pure cpu intensive operation. So, whenever we will call the function the CPU won't be able to process anything else. Which means it blocks the event loop.

### What's up with Promise.all
`Promise.all` doesn't guarantee us to run things in parallel. In fact, `Promise.all` is only reliable for waiting until all the promises given to it are resolved. It's job is to ensure that no promises get passed until they are done with their job. Is it confusing? Let's try with some examples.

```javascript
// getResultsFromDatabase() returns promise and does not contains blocking operation

const promise1 = getResultsFromDatabase(user1);
const promise2 = getResultsFromDatabase(user2);

const [res1, res2] = await Promise.all(promise1, promise2);
processData(res1, res2);
```

Take a look at the code above carefully. Both `getResultsFromDatabase` functions started executing before executing the 3rd line. The third line ensures that `processData` doesn't get executed until both the promises are done.

You can achieve the same result without using `Promise.all`. Example:

```javascript
const promise1 = getResultsFromDatabase(user1);
const promise2 = getResultsFromDatabase(user2);

const res1 = await promise1;
const res2 = await promise2;

processData(res1, res2);
```

The same thing happens here. `getResultsFromDatabase` gets executed immediately and then waits for them to resolve. `Promise.all` will return a reject promise if any of the given promises gets rejected. But, the reject will contain the reason for the first reject event if multiple reject happen.

### How would Promise.all behave?
Depending on the "Task/CPU" it might run in parallel or concurrently or sequentially. In single-core CPU the promises would run concurrently and in multi-core CPU they can be executed (!) in parallel for CPU intensive tasks. As explained earlier that most of the modern computers can do I/O in parallel. So, it is kind of safe to assume that network call, file read/write, etc tasks can run in parallel with cpu.

### Conclusion
JavaScript runtime is single-threaded. We do not have access to thread in JavaScript (recent version of NodeJS allows us to create thread using worker_threads module). Even if we have multi-core CPU we still can't run cpu intensive tasks in parallel directly using JavaScript. But, the browser/NodeJS uses C/C++ (!) where they have access to thread. So, they can achieve parallelism (ex: NodeJS's crypto module).



