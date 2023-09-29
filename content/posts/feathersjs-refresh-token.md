+++
author = "Dijkstra"
date = 2020-07-17T09:29:18Z
description = "FeathersJS provides authentication mechanism out of the box. But, it does not provide support for a refresh token. In this article we will see how we can implement refresh token feature to our system."
draft = false
slug = "feathersjs-refresh-token"
summary = "FeathersJS provides authentication mechanism out of the box. But, it does not provide support for a refresh token. In this article we will see how we can implement refresh token feature to our system."
tags = ["nodejs", "feathersjs", "javascript"]
title = "FeathersJS Refresh Token Implementation"

+++


FeathersJS provides authentication mechanism out of the box. But, it does not provide support for a refresh_token. In this article we will see how we can implement refresh_token feature to our system.

> In the article I will be using Mongoose but the main part should be identical for any ORM you use.

### What FeathersJS gives you

After creating a FeathersJS application you will get a file in **src** directory named `authentication.js`. You can say that this file is the **service** (I believe you know what a service is in FeathersJS) for authentication. In this authentication service the endpoint is defined and also other configurations like which class to use for _local-strategy_ or which class to use for _jwt-strategy_ is defined. By default the file looks like this.

```javascript
const { AuthenticationService, JWTStrategy } = require('@feathersjs/authentication');
const { LocalStrategy } = require('@feathersjs/authentication-local');
const { expressOauth } = require('@feathersjs/authentication-oauth');

module.exports = app => {
  const authentication = new AuthenticationService(app);

  authentication.register('jwt', new JWTStrategy());
  authentication.register('local', new LocalStrategy());

  app.use('/authentication', authentication);
  app.configure(expressOauth());
};
```

As you can see from the code above, Feathers uses its built-in `AuthenticationService` class to instantiate authentication service. For _local-strategy_ it uses `LocalStrategy` class and for _jwt-strategy_ it uses `JWTStrategy` class. All the default authentication features/behaviours you get from feathers are defined in those classes. But, the mechanism for refresh token is not built in to Feathers. I will not go deep into how refresh token works because there are tons of articles about it. One thing that I need to mention is, everyone uses a separate endpoint to refresh a token ex: `/refresh-token` but we are going to use our existing `/authentication` endpoint to refresh a token. It would make the whole process a lot easier.

### Goal to achieve

* When user logs in we send a **_refresh_token_** along with _**access_token**_.
* Allow user to get a new **_refresh_token_** and _**access_token**_ using previously issued valid **_refresh_token_**. No login needed this time.

### Plan to implement

* Create a new directory named `authentication` in `src` directory.
* Create 2 files named _CustomAuthService.js_ and _CustomJwtStrategy.js_.
* Create `CustomAuthService` class that inherits `AuthenticationService`.
* Override necessary method in our child class to facilitate refresh token feature.
* Create `CustomJwtStrategy` class that inherits `JWTStrategy`.
* Override necessary method in our child class to facilitate refresh token feature.

We don't need to override anything from `LocalStrategy` class because that class is responsible to handle login using password. Refresh token has noting to do with how we login to the system. One thing to keep in mind that `LocalStrategy` uses `AuthenticationService` as well. The **access_token** we get after login is generated using `AuthenticationService`. That is why we are creating our `CustomAuthService` to override the function that generates **access_token**. So that instead of returning only **access_token** we can generate a **refresh_token** as well.

### Custom Authentication Service

The code snippet below is the full `CustomAuthenticationService` class. Notice that we are overriding the `create` method. If you open the source code of `AuthenticationService` you will see that the `create` method creates _access_token_ and send response to the client. If you are already familiar with FeathersJS then you should know the role of `create` method of a service. The `create` method is responsible for handling **HTTP POST** request to a service. So, we are gonna override the create function and generate refresh_token as well and then append the refresh_token with the response. The snippet below is 90% similar to the original implementation of the `create` method. The only important part in this code is the `data.action` and `data.refresh_token` property. When you want to refresh a token you have to send those two additional parameter. Where `data.action` is 'refresh' and `data.refresh_token` is the refresh token you received when logged in. Example payload:

```json
{
	"strategy": "local",
	"action": "refresh",
	"refresh_token": "eyJhbGciOizcyJ9.eTFiYiJ9.DWe_BPD29z6LT2cttK7"
}
```

In `default.json` file create a new property named `refreshExpiresIn` and give it a value of `1d` because the validity of _refresh_token_ is much greater that _access_token_.

```javascript
const { AuthenticationService } = require('@feathersjs/authentication');
const { NotAuthenticated } = require('@feathersjs/errors');
const logger = require('../logger');

class CustomAuthenticationService extends AuthenticationService {
  /**
   * Create and return a new JWT for a given authentication request.
   * Will trigger the `login` event.
   * @param data The authentication request (should include `strategy` key)
   * @param params Service call parameters
   */
  async create(data, params) {
    const { entity } = this.configuration;
    const authStrategies = params.authStrategies || this.configuration.authStrategies;

    if (!authStrategies.length) {
      throw new NotAuthenticated('No authentication strategies allowed for creating a JWT (`authStrategies`)');
    }

    let refreshTokenPayload;
    let authResult;

    if (data.action === 'refresh' && !data.refresh_token) {
      throw new NotAuthenticated('No refresh token');
    } else if (data.action === 'refresh') {
      refreshTokenPayload = await this.verifyAccessToken(data.refresh_token, params.jwt);
      if (refreshTokenPayload.tokenType !== 'refresh') {
        throw new NotAuthenticated('Invalid token');
      }

      authResult = {
        [entity]: refreshTokenPayload[entity],
        authentication: { strategy: data.strategy },
      };
    } else {
      authResult = await this.authenticate(data, params, ...authStrategies);
    }

    logger.debug('Got authentication result ' + JSON.stringify(authResult));

    if (authResult && authResult.accessToken) {
      return authResult;
    }

    const [payload, jwtOptions] = await Promise.all([
      this.getPayload(authResult, params),
      this.getTokenOptions(authResult, params)
    ]);

    logger.debug(`Creating JWT Access Token with ${JSON.stringify(payload)}, ${JSON.stringify(jwtOptions)}`);

    const accessToken = await this.createAccessToken(payload, jwtOptions, params.secret);

    /**
     * Generate refresh token
     */
    const refreshTokenJwtOptions = {
      ...jwtOptions,
      expiresIn: this.configuration.refreshExpiresIn
    };

    refreshTokenPayload = {
      ...payload,
      tokenType: 'refresh',
      [entity]: authResult[entity]
    };

    logger.debug(`Creating JWT Refresh Token with ${JSON.stringify(refreshTokenPayload)}, ${JSON.stringify(refreshTokenJwtOptions)}`);

    const refreshToken = await this.createAccessToken(refreshTokenPayload, refreshTokenJwtOptions, params.secret);

    return Object.assign({}, { accessToken, refreshToken: refreshToken }, authResult);
  }
}

module.exports = CustomAuthenticationService;

```

### Custom Jwt Strategy

Now, our system has 2 type of JWT tokens and they have different roles. Refresh token is uses only to issue new tokens and access token is used to access all protected routes. Below code snippet is the implementation of `CustomJwtStrategy` class that extends `JWTStrategy`. Notice that we are overriding the `authenticate` method because only that method is responsible to verify a given token. Like our previous custom `create` method this method is also more or less 90% similar to the original implementation of `authenticate` method. The code is very straight forward and I believe I don't have to explain it.

```javascript
const { JWTStrategy } = require('@feathersjs/authentication');
const { NotAuthenticated } = require('@feathersjs/errors');
const logger = require('../logger');

class CustomJwtStrategy extends JWTStrategy {

  async authenticate(authentication, params) {
    const { accessToken } = authentication;
    const { entity } = this.configuration;

    if (!accessToken) {
      throw new NotAuthenticated('No access token');
    }

    const payload = await this.authentication.verifyAccessToken(accessToken, params.jwt);

    // If token type is refresh token then throw error
    if (payload.tokenType === 'refresh') {
      logger.info(`User (${payload.sub}) tried to access using refresh token`);
      throw new NotAuthenticated('Invalid access token');
    }

    const result = {
      accessToken,
      authentication: {
        strategy: 'jwt',
        accessToken,
        payload
      }
    };

    if (entity === null) {
      return result;
    }

    const entityId = await this.getEntityId(result, params);
    const value = await this.getEntity(entityId, params);

    return {
      ...result,
      [entity]: value
    };
  }
}

module.exports = CustomJwtStrategy;
```

### Use custom classes

We have created custom auth service and custom JWT strategy to support refresh_token. Now, open the `authentication.js` file and instead of importing the default import our newly created `CustomAuthService` and `CustomJwtStrategy`. See the code below.

```javascript
const { expressOauth } = require('@feathersjs/authentication-oauth');
const CustomAuthService = require('./authentication/CustomAuthService');
const CustomJwtStrategy = require('./authentication/CustomJwtStrategy');
const { LocalStrategy } = require('@feathersjs/authentication-local');

module.exports = app => {
  const authentication = new CustomAuthService(app);

  authentication.register('jwt', new CustomJwtStrategy());
  authentication.register('local', new LocalStrategy());

  app.use('/authentication', authentication);
  app.configure(expressOauth());
};
```

### Important

One important part is left that is to invalidate an used JWT refresh_token. It can easily be implemented in the `create` method of `CustomAuthenticationService` class. Later sometime I will update this article mentioning how we can implement it.

### Conclusion

FeathersJS is a very effective tool for rapid development. There is no native support for refresh_token yet. That is why I had to come up with something. Hope it helps you.
