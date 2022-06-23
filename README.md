<div align="center">
  <img src="logo/horizontal.png" alt="sherpa logo" width="450px">
</div>

Sherpa is a fork of [connect](https://github.com/senchalabs/connect), an extensible HTTP server framework for [node](http://nodejs.org) using "plugins" known as _middleware_.

```js
import sherpa from 'cangit/sherpa';
import http2 from 'http2';
import fs from 'fs';

const app = sherpa();

// respond to all requests
app.use(function (req, res) {
  res.end('Hello from Sherpa!\n');
});

// create node.js http2 server and listen on port
const port = 3000;
http2
  .createSecureServer(
    {
      key: fs.readFileSync('./privkey.pem'),
      cert: fs.readFileSync('./cert.pem'),
    },
    app
  )
  .listen(port, () => {
    console.log('🍱 Server running and listening on: ' + port);
  });
```

## Getting Started

Sherpa is a simple framework to glue together various "middleware" to handle requests.

### Install Sherpa

```sh
$ pnpm add github:cangit/sherpa
```

### Use middleware

The core of Sherpa is "using" middleware. Middleware are added as a "stack"
where incoming requests will execute each middleware one-by-one until a middleware
does not call `next()` within it.

```js
app.use(function middleware1(req, res, next) {
  // middleware 1
  next();
});
app.use(function middleware2(req, res, next) {
  // middleware 2
  next();
});
```

### Mount middleware

The `.use()` method also takes an optional path string that is matched against
the beginning of the incoming request URL. This allows for basic routing.

```js
app.use('/foo', function fooMiddleware(req, res, next) {
  // req.url starts with "/foo"
  next();
});
app.use('/bar', function barMiddleware(req, res, next) {
  // req.url starts with "/bar"
  next();
});
```

### Error middleware

There are special cases of "error-handling" middleware. There are middleware
where the function takes exactly 4 arguments. When a middleware passes an error
to `next`, the app will proceed to look for the error middleware that was declared
after that middleware and invoke it, skipping any error middleware above that
middleware and any non-error middleware below.

```js
// regular middleware
app.use(function (req, res, next) {
  // i had an error
  next(new Error('boom!'));
});

// error middleware for errors that occurred in middleware
// declared before this
app.use(function onerror(err, req, res, next) {
  // an error occurred!
});
```

## API

The Sherpa API is very minimalist, enough to create an app and add a chain
of middleware.

When the `sherpa` module is imported, a function is returned that will construct
a new app when called.

```js
// import module
import sherpa from 'cangit/sherpa';

// create app
const app = sherpa();
```

### app(req, res[, next])

The `app` itself is a function. This is just an alias to `app.handle`.

### app.handle(req, res[, out])

Calling the function will run the middleware stack against the given Node.js
http request (`req`) and response (`res`) objects. An optional function `out`
can be provided that will be called if the request (or error) was not handled
by the middleware stack.

### app.use(fn)

Use a function on the app, where the function represents a middleware. The function
will be invoked for every request in the order that `app.use` is called. The function
is called with three arguments:

```js
app.use(function (req, res, next) {
  // req is the Node.js http request object
  // res is the Node.js http response object
  // next is a function to call to invoke the next middleware
});
```

In addition to a plan function, the `fn` argument can also be a Node.js HTTP server
instance or another Sherpa app instance.

### app.use(route, fn)

Use a function on the app, where the function represents a middleware. The function
will be invoked for every request in which the URL (`req.url` property) starts with
the given `route` string in the order that `app.use` is called. The function is
called with three arguments:

```js
app.use('/foo', function (req, res, next) {
  // req is the Node.js http request object
  // res is the Node.js http response object
  // next is a function to call to invoke the next middleware
});
```

In addition to a plan function, the `fn` argument can also be a Node.js HTTP server
instance or another sherpa app instance.

The `route` is always terminated at a path separator (`/`) or a dot (`.`) character.
This means the given routes `/foo/` and `/foo` are the same and both will match requests
with the URLs `/foo`, `/foo/`, `/foo/bar`, and `/foo.bar`, but not match a request with
the URL `/foobar`.

The `route` is matched in a case-insensitive manner.

In order to make middleware easier to write to be agnostic of the `route`, when the
`fn` is invoked, the `req.url` will be altered to remove the `route` part (and the
original will be available as `req.originalUrl`). For example, if `fn` is used at the
route `/foo`, the request for `/foo/bar` will invoke `fn` with `req.url === '/bar'`
and `req.originalUrl === '/foo/bar'`.

## Running Tests

```bash
pnpm install
pnpm test
```

## License

[MIT](LICENSE)
