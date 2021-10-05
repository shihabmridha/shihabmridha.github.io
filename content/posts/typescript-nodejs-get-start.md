+++
author = "Dijkstra"
categories = ["Node.JS", "TypeScript"]
date = 2019-10-30T05:03:00Z
description = "End of the day everyone wants type safety in their code. Better type safety means less error in production. TypeScript provides type safety to JavaScript and it plays very well with Node.JS"
draft = false
slug = "typescript-nodejs-get-start"
summary = "End of the day everyone wants type safety in their code. Better type safety means less error in production. TypeScript provides type safety to JavaScript and it plays very well with Node.JS"
tags = ["Node.JS", "TypeScript"]
title = "TypeScript with NodeJS. Type safety with better IntelliSense in NodeJS"

+++


TypeScript is awesome. If you have never hard of TS then it is the right time to google for it. In case you do not want to switch the tab right now I will write a very short description about TypeScript.

### What is TypeScript?

TypeScript is a super set of JavaScript. What the hell does it mean? Well, what it means is TypeScript is basically JavaScript with some syntactical sugar. Why would you care? If you love type safety like me and dream about writing type safe JS then TypeScript in here for the rescue. It provides some advanced features that JavaScript still doesn't provide. For example: Decorators, Interface etc. Oh, did I mention that TypeScript is developed by Microsoft? Oh another thing, [Ryan Dahl](https://en.wikipedia.org/wiki/Ryan_Dahl) the creator of [Node.JS](https://nodejs.org/en/) praises TypeScript.

### Why should I care about Type Safety?

There are a lot of reasons for it. If you are from background of Java, C#, C++, Go or other strictly typed language then JavaScript would feel like a pain in the ass. At least I felt this way. You will miss type safety in JavaScript and will start to hate the language. But, this is just a personal preference right? What if you are not from the world of strictly typed language. JavaScript is your first language or you know Python very well. JavaScript/Python are not strictly typed language. They are loosely typed scripting language. They do not care about type that much. You can store string inside integer. The interpreter would do the conversion do the rest for you. This is one of many reasons why these languages **perform lower** than compiled languages. Type safety **reduces a lots of errors** in production by identifying them in compile time. For type safety IDE's are able to provide better **IntelliSense** than dynamically typed languages.

### Some example cases

**Mongoose:**

* If you use mongodb with node.js then you probably have heard of mongoose module. It is an ORM for mongodb. In mongoose you have to design the schema of a collection first then use it. For example: `User` is a schema containing fields like: name, email, password etc. Now, in plain javascript when you try to access the schema or create an instance of the schema the IDE won't be able to suggest you available properties of the schema. You can achieve is by typescript.
* Mongoose has some query functions like `findOne()`, `find()`. `findOne()` returns a single document where `find()` returns array of documents. Now, see the code snippet below. If you use typescript then IDE would give you an error because `findOne()` doesn't returns array, so `.length` property is not valid here.

```javascript
const users = await User.findOne({ name: 'Musa' });
if (users.length) { // IDE would give an error here
	...
}
```

### Well I am convinced. Where do I start?

It is pretty straight forward to learn TypeScript. There are tons of tutorials out there. But, the one that I would highly recommend is an official [TypeScript in 5 minutes](http://bit.ly/2oqXSri) tutorial. This tutorial is more than enough to get you started with TypeScript. All you need is a code editor and TypeScript compiler. My choice of code editor is [Visual Studio Code](https://code.visualstudio.com/).

Prerequisite:

1. Code editor (VS Code).
2. NodeJS and NPM.
3. [TypeScript](https://www.npmjs.com/package/tslint) compiler.
4. [TSLint](https://www.npmjs.com/package/tslint) extension (Built in VS Code).

First thing first, install NodeJS and NPM then install typescript globally. Command to install typescript `npm install -g typescript`.

Now, create a directory and inside that directory create a file named `test.ts`. Then, from that directory run `tsc test.ts`. A new `test.js` file will be created. You can take a look at the js file. Its a bit messy but you would get an idea how TypeScript transpiler/compiler turns typescript to plain JavaScript.

### Type Definition ([Type Declaration File](https://www.typescriptlang.org/docs/handbook/declaration-files/introduction.html))

In strictly typed world you can not just add a property to an object and call it when you want. Its a no do. What you would typically do is create an interface and then implements it to a class. Why thought? Lets assume you just required `express` in your `app.js` and called `express.foo()` which you already know is not a valid method. TypeScript compiler would catch this error in compile time and also if you have TSLint installed then the IDE would catch this as error. What type definition file does is, it provides an interface of all the available methods/properties of an object. For example, after importing `express` when you write `express.`, your IDE would show all the available options for the object. How would the IDE know about all the available properties? Well, its written in the type definition file. Now a days for increasing popularity of TypeScript, almost every NPM module contains type definition in it. Previously, type definition were provided by community. And most active and quality definitions were provided by [DefinitelyTyped](http://definitelytyped.org/). Type definitions are `devDependencies`. To install a type definition you would run the following command.

```bash
npm install --save-dev @types/express
```

### TS Configuration files

`tsconfig.json` contains configurations to be used by typescript compiler. This configuration file defines where the compiled code would be created, what module system to use, if it should support source map, which folders to watch etc. You will find more detail [here](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html).

`tslint.json` is same as `eslint.json` but its for typescript. I believe you are already familiar with ESLint. I am not going to write anything about it since you already know it. To find more about tslint check [here](https://www.npmjs.com/package/tslint).

### TypeScript, NodeJS starter

Microsoft has an official [GitHub repository](https://github.com/microsoft/TypeScript-Node-Starter) to get you started quickly in no time. This is not an standard but just one of many ways to structure your project. The structure should vary project by project. But I like the initial idea of this structure. I typically follow the following structure inspired from their official repository.

```text
test/
view/
src/
  - configs/
  - models/
  - controllers/
  - services/
app.js
server.js
[..more files..]
```

After creating project structure, first thing you should do is install `@types/node` which is type definition for nodejs. Remember to add it as dev dependency.

```
npm install --save-dev @types/node
```

### Tips

1. Use typescript formatter `tsfmt.json` file to format output plain JavaScript.
2. Use live compile in development `tsc --watch`. Config file is required.
3. Use sourcemap support for debugging.

## Conclusion

If you google something like 'why use typescript with nodejs then you will find a good amount of articles saying how good typescript. You will also notice that how people are arguing the fact that _if you need type-safety then why are you using loosely typed language in the first place? Why don't you go for Java or C#?_ You will also see people struggling to get things work in typescript way.

When I was not familiar with TypeScript I was on their side. But, once I started using it I became a fan. I can't convince you just by saying that TypeScript is good. You have to try it by yourself to feel it.

