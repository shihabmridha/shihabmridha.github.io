+++
author = "Dijkstra"
date = 2019-12-09T07:18:06Z
description = "Upload a file to multiple destination using multer module in nodejs."
draft = false
slug = "upload-file-multiple-destinations-multer"
summary = "Upload a file to multiple destination using multer module in nodejs."
tags = ["nodejs"]
title = "Upload file to multiple destinations using multer in node.js"

+++


This is a very straight forward article. If you work with Node.JS a lot then there is a high possibility that you are familiar with [multer](https://www.npmjs.com/package/multer). If you do not know about it then this line for you (taken from official repository).

> Multer is a node.js middleware for handling `multipart/form-data`, which is primarily used for uploading files. It is written on top of [busboy](https://github.com/mscdex/busboy) for maximum efficiency.

**Important:** _Multer will not process any form which is not multipart (`multipart/form-data`)._

## The problem

A few days back I was roaming around stackoverflow and found a question where the author wanted to _upload same file in different destination using multer_. The problem is very straight forward and so does the solution is.

## The solution

I have created a simple  **`express` ** to demonstrate the usage of it.

```javascript
const app = require("express")();
const multer  = require('multer')

const storageA = multer.diskStorage({
  destination: function (req, file, cb) {');
    cb(null, './xuploads/')
  },
})

const storageB = multer.diskStorage({
  destination: function (req, file, cb) {
    cb(null, './yuploads/')
  },
})

const destA = multer({ storage: storageA })
const destB = multer({ storage: storageB })

function fileUpload(req, res, next) {
  destA.single('file')(req, res, next);
  destB.single('file')(req, res, next);
  next();
}

app.post("/", fileUpload, (req, res) => {
  res.send("Received file");
});


app.listen(3000, () => {
  console.log("Server started");
});

```

If you run the above code and send a **multipart** request to `localhost:3000` then you will see that two directory has been created named `xuploads` and `yuploads`. Inside those folders you will see the file that you have uploaded.

## Usecase

I never had such requrements where I had to upload the same file to multiple destinations. The question's author wanted to upload the file to both `database` and `file system`.

