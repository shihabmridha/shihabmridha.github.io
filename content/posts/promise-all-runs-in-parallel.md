+++
author = "Dijkstra"
categories = ["JavaScript", "Concurrency", "Parallelism"]
date = 2019-10-30T07:37:34Z
description = "Things will run concurrently in single core CPU and will run in parallel in multi core CPU. It's main task is to wait until all the promises that are passed to it are resolved."
draft = false
slug = "promise-all-runs-in-parallel"
summary = "Things will run concurrently in single core CPU and will run in parallel in multi core CPU. It's main task is to wait until all the promises that are passed to it are resolved."
tags = ["JavaScript", "Concurrency", "Parallelism"]
title = "Does Promise.all() run tasks in parallel?"

+++


**_TL;DR_**

A big **No!** More accurately, it **depends on the CPU**.

---

The promise API is a blessing for JavaScript developers. It makes writing asynchronous JavaScript a lot easier. Go to hell **callback**! The problem is, like all other JavaScript concepts promise confuses a lot of people. I am not going to talk about promise in this article. There are tons of articles related to promise.

There is a misconception about **[concurrency](https://en.wikipedia.org/wiki/Concurrency_(computer_science))** and **parallelism** in the JavaScript world. When the promise was introduced people started to think that now everything is now going to run in parallel. I have no idea where they got this idea of parallelism.

### Let's talk about some **important**  **fundamentals**

A CPU can not perform multiple tasks at the same time. So, it means it is not possible to get parallelism in a single-core CPU. Then how does my single-core CPU perform multiple tasks at the same time? Well, there are a lot of things going on in the operating system but there are two main concepts. One is [multi-threading](https://en.wikipedia.org/wiki/Multithreading_(computer_architecture)) and another one is [context switching](https://en.wikipedia.org/wiki/Context_switch). Multiple tasks are handled by multiple threads. What happens is the CPU performs one task (using a thread) for a certain time and then pause the current task and move on to the next task (using another thread) and start processing it. This switching happens so fast that you think everything is running in parallel even though you are using a single-core CPU.

> A CPU can not perform multiple tasks at the same time.

### What's up with Promise.all

So, now you understand why `Promise.all` doesn't guarantee you to run things in parallel. In fact, `Promise.all` is only reliable for waiting until all the promises passed to it are done. Its job is to ensure that no promises get passed until they are done with their job. Is it confusing? Let's try with some examples.

```javascript
const promise1 = getResultsFromDatabase(user1);
const promise2 = getResultsFromDatabase(user2);

const [res1, res2] = await Promise.all(promise1, promise2);
processData(res1, res2);
```

Take a look at the code above carefully. Both `getResultsFromDatabase` function started executing before executing the 3rd line. The third line ensures that `processData` doesn't get executed before both of the promises are done.

You can achieve the same result without using `Promise.all`. Example:

```javascript
const promise1 = getResultsFromDatabase(user1);
const promise2 = getResultsFromDatabase(user2);

const res1 = await promise1;
const res2 = await promise2;

processData(res1, res2);
```

The same thing happens here. `getResultsFromDatabase` gets executed immediately and then waits for them to resolve.

### How would Promise.all behave?

Depending on the CPU it might run in parallel or concurrently or sequentially. We are gonna ignore the hyper-threaded CPU this time. In single-core CPU the promises would run concurrently and in multi-core CPU they can be executed (!) in parallel.

`Promise.all` will return a reject promise if any of the given promises rejects. But, the reject will contain the reason for the first reject event if multiple reject happens.

**But when is it actually gonna run in parallel?**

Let's assume, browser API has a function named **doStuff**. When you call the function the browser picks a thread from its pool and executes the function in that thread.

```javascript
await Promise.all([
  doStuff(),
  doStuff()
])
```

In the above snippet both the function would run in parallel if the machine has multiple CPUs. This is true for Node.JS as well. Instead of browser API, NodeJS has its own API.

### Conclusion

JavaScript runtime is single-threaded. We do not have access to thread in JavaScript. Even if you have multi-core CPU you still can't run tasks in parallel using JavaScript. But, the browser/NodeJS uses C/C++ (!) where they have access to thread. So, they can achieve parallelism.



