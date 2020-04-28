# [Polka](https://github.com/lukeed/polka)

**Contents**
* [Usage](#usage)
* [API](#api)
  * [`polka(options)`](#polkaoptions)
  * [`use(base, ...fn)`](#usebase-fn)
  * [`parse(req)`](#parsereq)
  * [`listen()`](#listen)
  * [`handler(req, res, parsed)`](#handlerreq-res-parsed)
* [Routing](#routing)
  * [Basics](#basics)
  * [Patterns](#patterns)
  * [Parameters](#parameters)
  * [Methods](#methods)
  * [Handlers](#handlers)
    * [Async Handlers](#async-handlers)
* [Middleware](#middleware)
  * [Middleware Sequence](#middleware-sequence)
  * [Middleware Errors](#middleware-errors)
* [Comparisons](#comparisons)

Polka is an extremely minimal, highly performant Express.js alternative.

Essentially, Polka is just a [native HTTP server](https://nodejs.org/dist/latest-v9.x/docs/api/http.html#http_class_http_server) with added support for routing, middleware, and sub-applications. That's it!

## Usage
[Back to Top](#polka)

```js
import polka from 'polka';

function one(req, res, next) {
    req.hello = 'world';
    next();
}

function two(req, res, next) {
    req.foo = '...needs better demo';
    next();
}

polka()
    .use(one, two)
    .get('/users/:id', (req, res) => {
        console.log(`~> Hello, ${req.hello}`);
        res.end(`User: ${req.params.id}`);
    })
    .listen(3000, err => {
        if (err) throw err;
        console.log(`> Running on localhost:3000`);
    });
```

## API
[Back to Top](#polka)

Polka extends [Trouter](https://github.com/lukeed/trouter) which means it inherits its API, too.

### `polka(options)`
[Back to Top](#polka)

Returns an instance of Polka

**options.server**

Type: `Server`

A custom, instantiated server that the Polka instance should attach its [`handler`](#handlerreq-res-parsed) to. This is useful if you have initialized a server elsewhere in your application and want Polka to use *it* instead of creating a new `http.Server`.

Polka *only* updates the server when [`polka.listen`](#listen) is called. At this time, Polka will create a [`http.Server`](https://nodejs.org/api/http.html#http_class_http_server) if a server was not already provided via `options.server`.

> **Important**: The `server` key will be `undefined` until `polka.listen` is invoked, unless a server was provided.

**options.onError**

Type: `Function`

A catch-all error handler; executed whenever a middleware throws an error. Change this if you don't like the default behavior.

Its signature is `(err, req, res, next)`, where `err` is the `String` or `Error` thrown by your middleware.

> **Caution**: Use `next()` to bypass certain errors **at your own risk!**  
> You must be certain that the exception will be handled elsewhere or *can* be safely ignored.  
> Otherwise your response will never terminate!

**options.onNoMatch**

Type: `Function`

A handler when no route definitions were matched. Change this if you don't like default behavior, which sends a `404` status & `Not found` response.

Its signature is `(req, res)` and requires that you terminate the response.

### `use(base, ...fn)`
[Back to Top](#polka)

Attache [middleware(s)](#middleware) and/or sub-application(s) to the server. These will execute *before* your routes' [handlers](#handlers).

**Important**: If a `base` pathname is provided, all functions within the same `use()` block will *only* execute when the `req.path` matches the `base` path.

**base**

Type: `String`  
Default: `undefined`

The base path on which the following middleware(s) or sub-application should be mounted.

**fn**

Type: `Function|Array`

You may pass one or more functions at a time. Each function must have the standardized `(req, res, next)` signature.

You may also pass a sub-application, which *must* be accompanied by a `base` pathname.

Please see [Middleware](#middleware) and [Express' middleware examples](http://expressjs.com/en/4x/api.html#middleware-callback-function-examples) for more info.

### `parse(req)`
[Back to Top](#polka)

Returns: `Object` or `undefined`

As of `v0.5.0`, this is an alias of the [`@polka/url`](https://github.com/lukeed/polka/blob/master/packages/url) module. For nearly all cases, you'll notice no changes.

But, for whatever reason, you can quickly swap in [`parseurl`]() again:

```js
const app = polka();
app.parse = require('parseurl');
```

### `listen()`
[Back to Top](#polka)

Returns: `Polka`

Boots (or creates) the underlying [`http.Server`](https://nodejs.org/api/http.html#http_class_http_server) for the first time. All arguments are passed to [`server.listen`](https://nodejs.org/dist/latest-v9.x/docs/api/net.html#net_server_listen) directly with no changes.

As of `v0.5.0`, this method no longer returns a Promise. Instead, the current Polka instance is returned directly, allowing for chained operations:

```js
// Could not do this before 0.5.0
const { server, handler } = polka().listen();

// Or this!
const app = polka().listen(PORT, onAppStart);

app.use('users', require('./users'))
  .get('/', (req, res) => {
    res.end('Pretty cool!');
  });
```

### `handler(req, res, parsed)`
[Back to Top](#polka)

The main Polka [`IncomingMessage`](https://nodejs.org/api/http.html#http_class_http_incomingmessage) handler. It receives all requests and tries to match the incoming URL against known routes.

If the `req.url` is not immediately matched, Polka will look for sub-applications or middleware groups matching the `req.url`'s [`base`](#usebase-fn) path. If any are found, they are appended to the loop, running *after* any global middleware.

> **Note**: Any middleware defined within a sub-application is run *after* the main app's (aka, global) middleware and *before* the sub-application's route handler.

At the end of the loop, if a middleware hasn't yet terminated the response (or thrown an error), the route handler will be executed, if found - otherwise a `(404) Not found` response is returned, configurable via [`options.onNoMatch`](#polkaoptions).

**req**

Type: `IncomingMessage`

**res**

Type: `ServerResponse`

**parsed**

Type: `Object`

Optionally provide a parsed [URL](https://nodejs.org/dist/latest-v9.x/docs/api/url.html#url_class_url) object. Useful if you've already parsed the incoming path. Otherwise, [`app.parse`](#parsereq) (aka [`parseurl`](https://github.com/pillarjs/parseurl)) will run by default.

## Routing
[Back to Top](#polka)

Routes are used to define how an application responds to varying HTTP methods and endpoints.

> If you're coming from express, there's nothing new here.  
> However, do check out [Comparisons](#comparisons) for some pattern changes.

### Basics
[Back to Top](#polka)

Each route is comprised of a [path pattern](#patterns), a [HTTP method](#methods), and a [handler](#handlers) (aka, what do you want to do).

In code, this looks like:

```js
app.METHOD(pattern, handler);
```

wherein:

* `app` is an instance of `polka`
* [`METHOD`](#methods) is any valid HTTP/1.1 method, lowercased
* [`pattern`](#patterns) is a routing pattern (string)
* [`handler`](#handlers) is the function to execute when `pattern` is matched.

The following example demonstrates some simple routes.

```js
const app = polka();

app.get('/', (req, res) => {
  res.end('Hello world!');
});

app.get('/users', (req, res) => {
  res.end('Get all users!');
});

app.post('/users', (req, res) => {
  res.end('Create a new User!');
});

app.put('/users/:id', (req, res) => {
  res.end(`Update User with ID of ${req.params.id}`);
});

app.delete('/users/:id', (req, res) => {
  res.end(`CY@ User ${req.params.id}!`);
});
```

### Patterns
[Back to Top](#polka)

Unlike the very popular [`path-to-regexp`](https://github.com/pillarjs/path-to-regexp), Polka uses string comparison to locate route matches. While [faster](https://github.com/lukeed/matchit#benchmarks) & more memory efficient, this does also prevent complex pattern matching.

All the basic an dmost commonly used patterns are supported. You probably only ever used these patterns in the first place.

> See [Comparisons](#comparisons) for the list of `RegExp`-based patterns that Polka does not support.

The supported pattern types are:

* static (`/users`)
* named parameters (`/users/:id`)
* nested parameters (`/users/:id/books/:title`)
* optional parameters (`/users/:id?/books/:title?`)
* any match / wildcards (`/users/*`)

### Parameters
[Back to Top](#polka)

Any named parameters included within your route [`pattern`](#patterns) will be automatically added to your incoming `req` object. All parameters will be found within `req.params` under the same name they were given.

> **Important**: Your parameter names should be unique, as shared names will ovewrite each other!

```js
app.get('/users/:id/books/:title', (req, res) => {
  let { id, title } = req.params;
  res.end(`User: ${id} && Book: ${title}`);
});
```

```bash
$ curl /users/123/books/Narnia
#=> User: 123 && Book: Narnia
```

### Methods
[Back to Top](#polka)

Any valid HTTP/1.1 method is supported. However, only the most common methods are used throughout this documentation for demo purposes.

> **Note**: For a full list of valid METHODs, please see [this list](https://github.com/lukeed/trouter#method).

### Handlers
[Back to Top](#polka)

Request handlers accept the incoming [`IncomingMessage`](https://nodejs.org/api/http.html#http_class_http_incomingmessage) and the formulating [`ServerResponse`](https://nodejs.org/dist/latest-v9.x/docs/api/http.html#http_class_http_serverresponse).

Every route definition must contain a valid `handler` function, or else an error will be thrown at runtime.

> **Important**: You must *always* terminate a `ServerResponse`!

It's a **very good** practice to *always* terminate your response ([`res.end`](https://nodejs.org/api/http.html#http_request_end_data_encoding_callback)) inside a handler, even if you expect a [middleware](#middleware) to do it for you. In the event a response is / was not terminated, the server will hang and eventually exit with a `TIMEOUT` error.

> **Note**: This is a native `http` behavior.

#### Async Handlers
[Back to Top](#polka)

If using Node 7.4 or later, you may leverage native `async` and `await` syntax.

No special preparation is needed - simply add the appropriate keywords.

```js
const app = polka();

const sleep = ms => new Promise(r => setTimeout(r, ms));

async function authenticate(req, res, next) {
  let token = req.headers['authorization'];
  if (!token) return (res.statusCode=401,res.end('No token!'));
  req.user = await Users.find(token);
  next();
}

app
  .use(authenticate)
  .get('/', async (req, res) => {
    console.log('~> current user', req.user);
    await sleep(500);
    res.end(`Hello, ${req.user.name}`);
  });
```

## Middleware
[Back to Top](#polka)

Middleware are functions that run in between (hence "middle") receiving the request and executing your route's [`handler`](#handlers) response.

> Coming from Express? Use any middleware you already know and love.

The middleware signature receives the request (`req`), the response (`res`), and a callback (`next`).

These can apply mutations to the `req` and `res` objects, and unlike Express, have access to `req.params`, `req.path`, `req.search`, and `req.query`.

Most importantly, a middleware ***must*** either call `next()` or terminate the response (`res.end`). Failure to do this will result in a never-ending response, which will eventually crash the `http.Server`.

```js
function logger(req, res, next) {
  console.log(`~> Received ${req.method} on ${req.url}`);
  next();
}

function authorize(req, res, next) {
  req.token = req.headers['authorization'];
  req.token ? next() : ((res.statusCode=401) && res.end('No token!'));
}

polka()
  .use(logger, authorize)
  .get('*', (req, res) => {
    console.log(`~> user token: ${req.token}`);
    res.end('Hello, valid user');
  });
```

```bash
$ curl /
# ~> Received GET on /
#=> (401) No token!

$curl -H "authorization: secret" /foobar
# ~> Received GET on /foobar
# ~> user token: secret
#=> (200) Hello, valid user
```

### Middleware Sequence
[Back to Top](#polka)

In Polka, middleware functions are organized into tiers.

Unlike Express, Polka middleware are tiered into "global", "filtered", and "route-specific" groups.

* Global middleware are defined via `.use('/', ...fn)` or `.use(...fn)`, which are synonymous. *Because* every request's `pathname` begins with a `/`, this tier is always triggered.

* Sub-group or "filtered" middleware are defined with a base `pathname` that's more specific than `/`. For example, defining `.use('/users', ...fn)` will run on any `/users/**/*` request.
  * These functions will execute *after* "global" middleware but before the route-specific handler.

* Route handlers match specific paths and execute last in the chain. They must also match the `method` action.

Once the chain of middleware handler(s) has been composed, Polka will iterate through them sequentially until all functions have run, until a chain member has terminated the response early, or until a chain member has thrown an error.

Contrast this with Express, which does not tier your middleware and instead iterates through your entire application in the sequence that you composed it.

```js
// Express
express()
  .get('/', get)
  .use(foo)
  .get('/users/123', user)
  .use('/users', users);

// Polka
Polka()
  .get('/', get)
  .use(foo)
  .get('/users/123', user)
  .use('/users', users);
```

```bash
$ curl {APP}/
# Express :: [get]
# Polka   :: [foo, get]

$ curl {APP}/users/123
# Express :: [foo, user]
# Polka   :: [foo, users, user]
```

### Middleware Errors
[Back to Top](#polka)

If an error arises within a middleware, the loop will be exited. This means that no other middleware will execute and neither will the route handler.

Similarly, regardless of `statusCode`, an early response termination will also exit the loop and prevent the route handler from running.

There are three ways to "throw" an error from within a middleware function.

> **Hint**: None of them use `throw`

1. **Pass any string to `next()`**

    This will exit the loop and send a `500` status code, with your erro string as the response body.

    ```js
    polka()
      .use((res, res, next) => next('💩'))
      .get('*', (res, res) => '*', (req, res) => res.end('wont run'));
    ```

    ```bash
    $ curl /
    #=> (500) 💩
    ```

2. **Pass an `Error` to `next()`**

    This is similar to the above option, but gives you a window in changing the `statusCode` to something other than the `500` default.

    ```js
    function oopsies(req, res, next) {
      let err = new Error('try again');
      err.code = 422;
      next(err);
    }
    ```

    ```bash
    $ curl /
    #=> (422) Try again
    ```

3. **Terminate the response early**

    Once the response has been ended, there's no reason to continue the loop!

    This approach is the most versatile as it allows to control every aspect of the outgoing `res`.

    ```js
    function oopsies(req, res, next) {
      if (true) {
        res.writeHead(400, {
          'Content-Type': 'application/json',
          'X-Error-Code': 'Please dont do this IRL'
        });

        let json = JSON.stringify({ error: 'Missing CSRF token' });
        res.end(json);
      } else {
        next(); // never called FYI
      }
    }
    ```

    ```bash
    $ curl /
    #=> (400) {"error":"Missing CSRF token"}
    ```

## Comparisons
[Back to Top](#polka)

Polka's API aims to be *very* similar to Express since most Node.js developers are already familiar with it. If you know Express, you already know Polka.

There are, however, a few main differences. Polka does not support or offer:

1. **Polka uses a tiered middleware system.**

    Express maintains the sequence of your route and middleware declarations during its runtime, which can pose a problem when composing sub-applications. Typically, this forces you to duplicate groups of logic.

    Please see [Middleware Sequence](#middleware-sequence) for an example and additional details.

2. **Any built-in view/rendering engines**.

    Most templating engines can be incorporated into middleware functions or used directly within a route handler.

3. **The ability to `throw` from within middleware.**

    However, all other forms of middleware-errors are supported. (See [Middleware Errors](#middleware-errors).)

    ```js
    function middleware(req, res, next) {
      // pass an error message to next()
      next('uh oh');

      // pass an Error to next()
      next(new Error('🙀'));

      // send an early, customized error response
      res.statusCode = 401;
      res.end('Who are you?');
    }
    ```

4. **Express-like response helpers... yet! [(#14)](https://github.com/lukeed/polka/issues/14)**

    Express has a nice set of [response helpers](http://expressjs.com/en/4x/api.html#res.append). While Polka relies on the [native Node.js response methods](https://nodejs.org/dist/latest-v9.x/docs/api/http.html#http_class_http_serverresponse), it would be very easy / possible to attach a global middleware that contained a similar set of helpers.

5. **`RegExp`-based route patterns.**

    Polka's router uses string comparison to match paths against patterns. It's a lot quicker and more efficient.

    The following routing patterns **are not** supported:

    ```js
    app.get('/ab?cd', _ => {});
    app.get('/ab+cd', _ => {});
    app.get('/ab*cd', _ => {});    
    app.get('/ab(cd)?e', _ => {});
    app.get('/a/', _ => {});
    app.get('/.*fly$/', _ => {});
    ```

    The following routing patterns **are** supported:

    ```js
    app.get('/users', _ => {});
    app.get('/users/:id', _ => {});
    app.get('/users/:id?', _ => {});
    app.get('/users/:id/books/:title', _ => {});
    app.get('/users/*', _ => {});
    ```

6. **Polka instances are not (directly) the request handler.**

    Most packages in the Express ecosystem expect you to pass your `app` directly into the package. This is because `express()` returns only a middleware signature directly.

    In the Polka-sphere, this functionality lives in your application's [`handler`](#handlerreq-res-parsed) instead.

    Here's an example with [`supertest`](https://github.com/visionmedia/supertest), a popular testing utility for Express apps.

    ```js
    const request = require('supertest');
    const send = require('@polka/send-type');

    const express = require('express')();
    const polka = require('polka')();

    express.get('/user', (req, res) => {
      res.status(200).json({ name: 'john' });
    });

    polka.get('/user', (req, res) => {
      send(res, 200, { name: 'john' });
    });

    function isExpected(app) {
      request(app)
        .get('/user')
        .expect('Content-Type', /json/)
        .expect('Content-Length', '15')
        .expect(200);
    }

    // Express: Pass in the entire application directly
    isExpected(express);

    // Polka: Pass in the application `handler` instead
    isExpected(polka.handler);
    ```