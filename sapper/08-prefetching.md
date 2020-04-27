# Prefetching

**Topics**
* [Introduction](./readme.md)
* [Sapper App Structure](./01-sapper-app-structure.md)
* [Routing](./02-routing.md)
* [Client API](./03-client-api.md)
* [Preloading](./04-preloading.md)
* [Layouts](./05-layouts.md)
* [Server-Side Rendering](./06-server-side-rendering.md)
* [Stores](./07-stores.md)
* [Building](./09-building.md)
* [Exporting](./10-exporting.md)
* [Deployment](./11-deployment.md)
* [Security](./12-security.md)
* [Base URLs](./13-base-urls.md)
* [Testing](./14-testing.md)
* [Debugging](./15-debugging.md)
* [CSS Frameworks](./a1-css-frameworks.md)

**Contents**
* [rel=prefetch](#relprefetch)

Sapper uses code splitting to break your app into smaller chunks (one per route), ensuring fast startup times.

For *dynamic* routes, such as our `src/routes/blog/[slug].svelte` example, that's not enough. In order to render the blog post, we need to fetch the data for it, and we can't do that until we know what `slug` is. In the worst case, that could cause lag as the browser waits for the data to come back from the server.

## rel=prefetch
[Back to Top](#prefetching)

We can mitigate that by *prefetching* the data. Adding a `rel=prefetch` attribute to a link...

```svelte
<a rel=prefetch href="blog/what-is-sapper">What is Sapper?</a>
```

...will cause Sapper to run the page's `preload` function as soon as the user hovers over the link (on a desktop) or touches it (on mobile), rather than waiting for the `click` event to trigger navigation. Typically, this buys us an extra couple of hundred milliseconds, which is the difference between a user interface that feels laggy, and one that feels snappy.

> `rel=prefetch` is a Sapper idiom, not a standard attribute for `<a>` elements