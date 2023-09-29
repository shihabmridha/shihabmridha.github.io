+++
author = "Dijkstra"
date = 2019-11-29T12:47:05Z
description = "Use nodejs worker thread to handle CPU intensive task without blocking the main thread. JSON parsing using nodejs worker thread."
draft = false
slug = "nodejs-worker-thread-json-parse"
summary = "Use nodejs worker thread to handle CPU intensive task without blocking the main thread. JSON parsing using nodejs worker thread."
tags = ["nodejs"]
title = "NodeJS worker threads. Handle CPU intensive task without blocking the event loop"

+++




> Parsing JSON using worker_thread would [not technically be benificial](https://news.ycombinator.com/item?id=21007207). The below article is just to demonstrate how you can utilize worker_threads in Node.JS

In my recent project, I had a task where a user would be able to import data. Data will be in JSON format. The file size could be as large as 200MB. So, the challenge was to parse a large JSON file. Importing a file is a simple and typical operation but, when we are talking about Node.JS we have to be very careful about it. Because uploading the large file in the system is not the issue. The issue is parsing the large file. JSON parsing is a **CPU intensive task**. This means, while parsing the JSON, the system will not be able to perform any other task (ex: response to other requests). Why is that? Because JSON.parse() function will block the main thread from performing other operations. In other words, the **event loop** will not respond until the JSON.parse() is done with it. This is a very important concept to understand. Here is an official article about [why you should not block the event loop](https://nodejs.org/ru/docs/guides/dont-block-the-event-loop/). You will find some recommendations about parsing JSON in that official article as well.

<h3><a style="color: #c40233" href="#gotcha">! SEE THE GOTCHA BELOW</a></h3>

## How to parse JSON without blocking the event loop in Node.JS?

So, the main goal was to parse large JSON files without affecting the main thread aka event loop.

### Possible solutions

* Asynchronous JSON parsing using Big-friendly JSON (BFJ) (recommended)
* Worker threads **_(for the sake of this article)_**

I already had a solution in my mind which was stream parsing the JSON using a very popular module named [BFJ aka Big-friendly JSON](https://www.npmjs.com/package/bfj). BFJ is **not as fast as**  `JSON.parse()` function because it parses some data then let the event loop handle some other tasks. I have implemented it anyway and let the client test it. In my development environment (Macbook pro 2015 with 16GB memory) BFJ took 7-8 seconds to parse 20MB of data. I thought I might use **_worker_threads_** to offload the task to some other thread. The Node.JS [**_worker threads_**](https://nodejs.org/api/worker_threads.html) module is already stable.

## Worker threads

From documentation:

> The `worker_threads` module enables the use of threads that execute JavaScript in parallel.

This is what we need, right? My main program will run normally and I will offload the JSON parsing task to some other threads. So, lets do this. First thing first, import necessary stuff and create a new thread.

```javascript
const { Worker, isMainThread } = require('worker_threads');

let worker;
if (isMainThread) {
    worker = new Worker(__dirname + '/worker.js');
    
    worker.on('message', (data) => {
      // 'data' contains the parsed JSON sent by worker thread
      // Do something with data
    });
    
    worker.on('error', (error) => {
      // Logging error caused by worker thread
      console.log(error.message);
    });
    
    worker.on('exit', (code) => {
        if (code !== 0)
            throw new Error(`Worker stopped with exit code ${code}`);
        else
            logger.info('Worker stopped ' + code);
    });
}
```

In the code above we are checking that if we are currently in main thread or not. If we are in main thread then create another thread. Also, listening to a few ( `exit`, `message`, `error`) events.  _The `exit` event callback would get executed when the worker thread exited by an error or manually being exited (Example: `process.exit(1)`). The `message` event callback will get executed when the worker thread sends some data/message to main thread. And finally, the `error` callback will get executed when the worker thread encounters an error._

> The above code will run in main thread.

Notice the **line number 6**. It is one of the most important part. We have created a new thread but what would the thread do? _Line number 6 defines which task to perform._ We have addressed a file named `worker.js`. Let's take a look at that file.

```javascript
const { isMainThread, parentPort } = require('worker_threads');

if (!isMainThread) {
	parentPort.on('message', (data) => {
    	// 'data' contains the payload sent by main thread
        const parsedJSON = JSON.parse(data);
        
        // Send data back to main thread
        parentPort.postMessage(parsedJSON);
    }
}
```

Unlike the main thread, we are now checking if we are in worker threads. If yes then listen to the `message` event. Notice the usage of `parentPort`. It is the bridge between main thread and worker thread.  _The `message` event callback will get executed when the worker thread receives any message/data from main thread._

If you notice the above **app.js** and **worker.js** file carefully then you will see we are not sending any data to worker thread yet. To send a message or some data to worker thread, from main thread we have to use `postMessage` function like this `worker.postMessage(payload)`. So finally, the main.js file would look like this.

```javascript
const { Worker, isMainThread } = require('worker_threads');

let worker;
if (isMainThread) {
    worker = new Worker(__dirname + '/worker.js');
    
    worker.on('message', (data) => {
      // 'data' contains the parsed JSON sent by worker thread
      // Do something with data
    });
    
    worker.on('error', (error) => {
      // Logging error caused by worker thread
      console.log(error.message);
    });
    
    worker.on('exit', (code) => {
        if (code !== 0)
            throw new Error(`Worker stopped with exit code ${code}`);
        else
            logger.info('Worker stopped ' + code);
    });
}

// Assume we have an endpoint that receive a JSON formated data as file
app.post('/upload', multer({...}).singe('data'), (req, res) => {
    // Read file sent by user
	fs.readFile(req.file, (data) => {
        // Send the data to worker thread
    	worker.postMessage(data);

        res.send('We are processing your file.');
    });
});
```

## Full data flow

Let's assume our app has a REST endpoint that receives a file. The file contains texts in valid JSON format that we will parse and do some processing later. In our final **main.js** file, first we are reading the file and then sending the data to our worker thread (**main.js,**  **line 30**). This data is received by the worker thread (**worker.js, line 4**) and `message` event gets executed. Worker thread runs the `JSON.parse()` and sends the data back to main thread (**worker.js, line 9**). When main thread receives the parsed JSON data the `message` event gets emitted and it's callback gets executed (**main.js, line 7**). There you have your parsed JSON data. Do the necessary processing now.

<h2 id="gotcha">Gotcha</h2>

From Node's official documentation:

> In actual practice, use a pool of Workers instead for these kinds of tasks. Otherwise, the overhead of creating Workers would likely exceed their benefit.

When you transfer data from a worker thread to the main thread, the data gets copied. To prevent copy of the data you have to use either `ArrayBuffer` or `SharedArrayBuffer`. So in our case, when we transfer parsed JSON from worker thread to main thread it gets copied (NodeJS implicitly stringify the data from worker and send it to main thread and then call parse again). **Which means, we do not get any benefit!**

## Conclusion

Since I was not getting any benefit from worker_threads, I had to stick with BFJ. The main use of worker threads is to enable the use of threads in NodeJS so that user can perform CPU-intensive JavaScript operations. Worker threads will not help that much with I/O intensive work. NodeJS's default non-blocking I/O is more efficient in that case.

