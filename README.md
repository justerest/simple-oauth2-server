[![npm version](https://badge.fury.io/js/simple-oauth2-server.svg)](http://badge.fury.io/js/simple-oauth2-server)

# simple-oauth2-server
## Introdution
Simple module for deploying oAuth2 server with several levels of protection.
Perfect work with <a href="https://github.com/simplabs/ember-simple-auth">`ember-simple-auth`</a>

It use <a href="https://github.com/typicode/lowdb">`lowdb`</a> for saving tokens from the box.


## Basic usage
```
npm i --save simple-oauth2-server
```
```javascript
const express = require('express');
const app = express();
const simpleOAuth2Server = require('simple-oauth2-server');

simpleOAuth2Server
    .init(app, {
        // your function for authentication (must return `true` or `false`)
        checkPassword: function(request) {
            const {
                username,
                password
            } = request.body;
            if (username === 'user' && password === 'pass') {
                return true;
            }
            return false;
        }
    })
    .extend({
        routes: ['/secret'], // routes which you want to protect
        methods: ['get', 'post', 'delete', 'put'] // methods for protective routes
    });
```
Your protection is enabled! And server send tokens on requests on `tokenGetPath`.


## Default options
```javascript
{
  /**
    @function Your function for issuing tokens
    @default undefined
    @param request
  **/
  checkPassword: /* required declare */,
  /**
    @property Protected routes
    @default []
    @type array
  **/
  routes: [],
  /**
    @property Methods for protected routes ['get', 'post', 'delete', 'put'] (except 'any')
    @default []
    @type array
  **/
  methods: [],
  /**
    @property Token lifetime
    @default one day
    @type integer
  **/
  tokenExpired: 24 * 60 * 60,
  /**
  @function Your function for configuring token format
  @default undefined
  @param request
  **/  
  tokenExtend: function(request) {
    return {
      username: request.body.username
    };
  }
  /**
    @property Route where server issues tokens
    @default '/token'
    @type string
  **/
  tokenGetPath: '/token',
  /**
    @property Route where server revokes tokens
    @default '/tokenRevocation'
    @type string
  **/
  tokenRevocationPath: '/tokenRevocation',
  /**
    @function Function for extraction access token from headers (must return value of access token)
    @default сonfigured for Bearer tokens
    @param request
  **/
  authorizationHeader: function(request) {
      return request.get('Authorization') ? request.get('Authorization').replace('Bearer ', '') : false;
  }
}
```

## Token info
On protect routes you can get token info from `req.token`
```javascript
app.get('/secret-data', (req, res) => {
    console.log(req.token);
    res.send(/* secret data */);
});
```

Default information in token (not re-written)
```javascript
{
    access_token: uuid(),
    refresh_token: uuid(),
    expires_in: this.tokenExpired,
    expires_at: moment()
}

```

## Add new layer of protection
If you need to several levels protection you can add new protect function and extend protection for other routes:
```javascript
simpleOAuth2Server
    .addProtect(checkUserRights)
    .extend({
        routes: ['/secret*'],
        methods: ['get']
    });
```

You can combine many layers of protection for your application. Make joint layers and layers with unique function of protection. And you can add layer of protection as middleware in route instead extending:
```javascript
const superProtect = simpleOAuth2Server
    .addProtect(isSuperAdmin)
    .addProtect(checkUserRights)
    .protect;

app.get('/only/super/users/can/read', superProtect, (req, res) => {
    res.send('You are super!');
});
```

## Full usage
```javascript
simpleOAuth2Server
    // Let's start issuing tokens
    .init(app, {
        checkPassword: authenticationCheck, // Your function for issuing tokens (required)
        tokenExpired: 24 * 60 * 60, // one day by default
        tokenGetPath: '/token',
        tokenRevocationPath: '/tokenRevocation',
        // Your function for configuring token format if it's needed
        tokenExtend: function(request) {
            return {
                username: request.body.username
            };
        },
        // Function for extraction access token from headers (must return value of access token)
        // Configured for Bearer tokens by default
        authorizationHeader: function(request) {
            return request.get('Authorization') ? request.get('Authorization').replace('Bearer ', '') : false;
        }
    })
    // Enable protection on routes (access only for authenticated users)
    .extend({
        // routes which you want to protect, example: ['/secret/documents', '/secret-images/**']
        routes: ['/comments/'],
        // methods for protect routes, example (except 'any'): ['get', 'post', 'delete', 'put']
        methods: ['post']
    })
    // Add new protective layer for some routes
    .addProtect(checkAccess)
    // Access only for authorized users
    .extend({
        routes: ['/posts/'],
        methods: ['post']
    })
    .extend({
        routes: ['/posts/:post_id'],
        methods: ['put', 'get', 'delete']
    })
    // Remove all previous levels of protection (function checkAccess in this example)
    .clearProtects()
    // Add new protective layer for some routes
    .addProtect(isAdmin)
    // Access only for administator
    .extend({
        routes: ['/users/'],
        methods: ['post']
    })
    .extend({
        routes: ['/users/:post_id'],
        methods: ['delete']
    });
```

## Example
You can watch an usage example on https://github.com/justerest/simple-oauth2-server/blob/master/example/app.js

Just run:
```
git clone https://github.com/justerest/simple-oauth2-server.git
cd /simple-oauth2-server
npm i && npm start
```

## Have questions or problems?
You can send me message on justerest@yandex.ru or create an issue.
I'm glad to listen any questions, criticism and suggestions.

