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

As of `v0.5.0`, this method no longer returns a Promise. Instead, teh current Polka instance is returned directly, allowing for chained operations:

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

Optionally provide a parsed [URL](https://nodejs.org/dist/latest-v9.x/docs/api/url.html#url_class_url) object. Useful if you've already parsed teh incoming path. Otherwise, [`app.parse`](#parsereq) (aka [`parseurl`](https://github.com/pillarjs/parseurl)) will run by default.

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

