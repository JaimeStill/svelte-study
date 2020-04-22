# Stores

**Topics**
* [Introduction](./readme.md)
* [Sapper App Structure](./01-sapper-app-structure.md)
* [Routing](./02-routing.md)
* [Client API](./03-client-api.md)
* [Preloading](./04-preloading.md)
* [Layouts](./05-layouts.md)
* [Server-Side Rendering](./06-server-side-rendering.md)
* [Prefetching](./08-prefetching.md)
* [Building](./09-building.md)
* [Exporting](./10-exporting.md)
* [Deployment](./11-deployment.md)
* [Security](./12-security.md)
* [Base URLs](./13-base-urls.md)
* [Testing](./14-testing.md)
* [Debugging](./15-debugging.md)

**Contents**
* [Seeding Session Data](#seeding-session-data)

The `page` and `session` values passed to `preload` functions are available to components as [stores](../tutorial/07-stores.md#writable-stores), along with `preloading`.

Inside a component, get references to the stores like so:

```svelte
<script>
  import { stores } from '@sapper/app';
  const { preloading, page, session } = stores();
</script>
```

* `preloading` contains a readonly boolean value, indicating whether or not a navigation is pending.
* `page` contains a readonly `{ host, path, params, query }` object, identical to that passed to `preload` functions.
* `session` contains whatever data was seeded on the server. It is a [writable-store](../tutorial/07-stores.md#writable-stores), meaning you can update it with new data (for example, after the user logs in) and your app will be refreshed.

## Seeding Session Data
[Back to Top](#stores)

On the server, you can populate `session` by passing an option to `sapper.middleware`:

**src/server.js**
```js
express() // or Polka, or a similar framework
  .use(
    serve('static'),
    authenticationMiddleware(),
    sapper.middleware({
      session: (req, res) => ({
        user: req.user
      })
    })
  )
  .listen(process.env.PORT);
```

> Session data must be serializable (using [devalue](https://github.com/Rich-Harris/devalue)) - no functions or custom classes, just built-in JavaScript data types.