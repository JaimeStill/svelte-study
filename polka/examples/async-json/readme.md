# Example: Async

> **WARNING** This will only work with Node 7.4 or later!

This example makes use of Node's built-in `async`/`await` - no compiling required!

It forwards all requests to [HNPWA's JSON API](), utilizing [`node-fetch`]() for server-side requests.

## Setup

```bash
$ yarn
$ yarn start
```

## Usage

There are only a few valid paths: `/`, `/news`, and `/newest`.

Anything else will return `(404) Not Found`.

```bash
$ curl localhost:3000
#=> (200) JSON
$ curl localhost:3000/news
#=> (200) JSON
$ curl localhost:3000/foobar
#=> (404) Not Found
```