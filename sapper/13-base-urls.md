# Base URLs

**Topics**
* [Introduction](./readme.md)
* [Sapper App Structure](./01-sapper-app-structure.md)
* [Routing](./02-routing.md)
* [Client API](./03-client-api.md)
* [Preloading](./04-preloading.md)
* [Layouts](./05-layouts.md)
* [Server-Side Rendering](./06-server-side-rendering.md)
* [Stores](./07-stores.md)
* [Prefetching](./08-prefetching.md)
* [Building](./09-building.md)
* [Exporting](./10-exporting.md)
* [Deployment](./11-deployment.md)
* [Security](./12-security.md)
* [Testing](./14-testing.md)
* [Debugging](./15-debugging.md)

Ordinarily, the root of your Sapper app is served at `/`. But in some cases, your app may need to be served from a different base path - for example, if Sapper only controls part of your domain, or if you have multiple Sapper apps living side-by-side.

This can be done like so:

**server.js**
```js
express() // or Polka, or a similar framework
  .use(
    '/my-base-path', // <!-- add this line
    compression({ threshold: 0 }),
    serve('static'),
    sapper.middleware()
  )
  .listen(process.env.PORT);
```

Sapper will detect the base path and configure both the server-side and client-side routers accordingly.

If you're [exporting](./10-exporting.md) your app, you will need to tell the exporter where to begin crawling:

```bash
sapper export --basepath my-base-path
```