+++
author = "Dijkstra"
categories = ["FeathersJS", "Node.JS"]
date = 2019-11-19T19:59:24Z
description = "Create a custom route in FeathersJS without creating a new service. A short video of the whole process is given as well."
draft = false
slug = "feathersjs-custom-route-without-service"
summary = "Create a custom route in FeathersJS without creating a new service. A short video of the whole process is given as well."
tags = ["FeathersJS", "Node.JS"]
title = "Create a custom route in FeathersJS without creating new service"

+++


<img src="https://feathersjs.com/img/feathers-logo-wide.png"/>

If you do not know what [FeathersJS](https://feathersjs.com/) is then this line is for you. It is taken from FeathersJS's official website.

> A framework for real-time applications and REST APIs

### Index

* [Why use FeathersJS?](#why-feathers)
* [Install FeathersJS](#install-feathers)
* [Create new app](#create-app)
* [Create new service](#new-service)
* [Create custom route](#custom-route)

<h2 id="why-feathers">Why use FeathersJS?</h2>

* Create an app from scratch within a minute.
* Create REST API with zero configuration.
* Create Real-Time API with zero configuration.
* Service oriented application.
* Database agnostic.
* Supports TypeScript. [Why is this important](__GHOST_URL__/typescript-nodejs-get-start/)?
* CLI to create CRUD using single command.

There are more features that I  about Feathers for example: hooks, auto configured authentication, auto configured test using mocha and more.

<h2 id="install-feathers">Install FeathersJS</h2>

I am assuming you already know how to install, create app and create service using feathers. I am writing an small instruction for those who does not know yet. I will use the latest version of Feathers which is **4.2.3**.

Run the following command to install feathers CLI `npm install -g @feathersjs/cli` .

<h2 id="create-app">Create Feathers app</h2>

First of all create a folder then navigate to it and run the following command `feathers generate app`. You will be asked a few questions like app name, source folder name, which package manager would the CLI use, is it typescript app or javascript, type of API (REST/Real-time/Both) etc.

<h2 id="new-service">Create a new service</h2>

For this topic I am going to create a service named **organizations**. **** To do that run the following command `feathers generate service`. You will be asked a few queries related to your service. CLI will perform a few actions to create a new service. One of the action is create a new folder inside **_[source_directory] > services_**. For my case it looks like this: **_src > services > organizations_**_._ Our organizations service is ready to CRUD.

Organizations directory has three files in it. They are **organizations.class.js, organizations.hooks.js, organizations.service.js**. We will be focusing on **.class** and **.service** file in this article.

Initially the **organizations.class.js** and **organizations.service.js** file would look like this.

```javascript
const { Service } = require('feathers-mongoose');

exports.Organizations = class Organizations extends Service {
  
};

```

```javascript
// Initializes the `organizations` service on path `/organizations`
const { Organizations } = require('./organizations.class');
const createModel = require('../../models/organizations.model');
const hooks = require('./organizations.hooks');

module.exports = function (app) {
  const options = {
    Model: createModel(app),
    paginate: app.get('paginate')
  };

  // Initialize our service with any options it requires
  app.use('/organizations', new Organizations(options, app));

  // Get our initialized service so that we can register hooks
  const service = app.service('organizations');

  service.hooks(hooks);
};

```

<h2 id="custom-route">Create custom route</h2>

## The easy way

FeathersJS uses ExpressJS under the hood. So, we will be able to take advantage of it. To make a custom route the easy way follow the code block below and notice from line 15 to 18.

```
// Initializes the `organizations` service on path `/organizations`
const { Organizations } = require('./organizations.class');
const createModel = require('../../models/organizations.model');
const hooks = require('./organizations.hooks');

module.exports = function (app) {
  const options = {
    Model: createModel(app),
    paginate: app.get('paginate')
  };

  // Initialize our service with any options it requires
  app.use('/organizations', new Organizations(options, app));

  // Initialize our custom route
  app.get('/custom-organizations', (req, res) => {
  	res.send('Hello from custom route');
  });

  // Get our initialized service so that we can register hooks
  const service = app.service('organizations');

  service.hooks(hooks);
};

```

## The feathers way

In the **service > organizations** directory create a new file named **custom-organizations.class.js**. Copy everything from **organizations.class.js** and rename Organizations to CustomOrganizations. Now the file should look like this.

```javascript
const { Service } = require('feathers-mongoose');

exports.CustomOrganizations = class CustomOrganizations extends Service {
};

```

Now we have to connect this custom class with our service somehow. Here is how you can achieve it. This is update version of **organizations.service.js**.

```javascript
// Initializes the `organizations` service on path `/organizations`
const { Organizations } = require('./organizations.class');
const { CustomOrganizations } = require('./custom-organizations.class');
const createModel = require('../../models/organizations.model');
const hooks = require('./organizations.hooks');

module.exports = function (app) {
  const options = {
    Model: createModel(app),
    paginate: app.get('paginate')
  };

  // Initialize our service with any options it requires
  app.use('/organizations', new Organizations(options, app));

  // Initialize our custom route
  app.use('/custom-organizations', new CustomOrganizations(options, app));

  // Get our initialized service so that we can register hooks
  const service = app.service('organizations');

  service.hooks(hooks);
};

```

Notice line number 3 and 17. First imported the class in service, then register the class on the applicaton.

> _**Technically, we are creating a new feathers service.**_

Now run the app and hit `/custom-organizations`. You should get expected response.

## Conclusion

Technically speaking we had to create a new service but, also for simple task this way of creating a custom endpoint is convenient in my opinion.

## Video

<iframe width="480" height="270" src="https://www.youtube.com/embed/o5eUzfNd2xc?feature=oembed" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>



### Reference:

* The image is from feathers's official website.

