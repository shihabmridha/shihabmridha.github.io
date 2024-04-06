+++
author = "Dijkstra"
date = 2020-02-02T18:00:03Z
description = "This script will download course for you from Educative.io. It uses Bun, TypeScript and your login credentials."
draft = false
slug = "educative-io-course-downloader"
summary = "This script will download course for you from Educative.io. It uses Bun, TypeScript and your login credentials."
tags = ["bun", "typescript", "javascript"]
title = "Educative.io course downloader"

+++
Are you familiar with [educative.io](https://www.educative.io)? They provide interactive courses for software developers. One of their famous course is [Grokking the system design interview](https://www.educative.io/courses/grokking-the-system-design-interview). Course authors are software engineer of large companies like Google, Amazon, Microsoft, Databricks etc. So you know, this course is not authored by some average software engineers. In this course you will learn how Twitter, Uber, Dropbox etc systems are designed.

Recently I have got a subscription of educative.io and started to explore their courses. Then, one thing came to my mind. What if my subscription ends? These are not videos that I can download and watch later. So, I have to download it somehow to read it later. That is when I started to write a simple script which would do the downloading for me.

### How it works?

* Opens a browser to perform login.
* Login using your given credentials. (If not already logged in)
* Go to the course page and fetch all the lesson's link.
* Go to lesson link and save the page as user's preferred setting.

### Prerequisites

* Bun
* Educative.io subscription (for paid courses)

### How to use?

Instructions are given in to the repository but for the sake of this article here we go again. Here is the [GitHub repository](https://github.com/shihabmridha/educative.io-downloader) link.

* Clone the project first. Then navigate to the project directory.
* Run `bun install` to install required dependencies.
* Open `config/default.json` file in a test editor and put your email, password and course URLs that you want to download.
* Finally run, `bun start`.

Now the script should start downloading the course and save it to `downloads/{COURSE_TITLE}` directory. Each lessons are number by their order.

### Conclusion

This script is written in a couple of hours and there are a lot of things to improve. Feel free to create PR, issue etc.

Thank you for reading.
