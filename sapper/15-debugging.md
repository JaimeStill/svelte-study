# Debugging

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
* [Base URLs](./13-base-urls.md)
* [Testing](./14-testing.md)
* [CSS Frameworks](./a1-css-frameworks.md)

Debugging your server code is particularly easy with [ndb](https://github.com/GoogleChromeLabs/ndb). Install it globally...

```bash
npm install -g ndb
```

...then run Sapper:

```bash
ndb npm run dev
```

> This assumes that `npm run dev` runs `sapper dev`. You can also run Sapper via [npx](https://blog.npmjs.org/post/162869356040/introducing-npx-an-npm-package-runner), as in `ndb npx sapper dev`.

Note that you may not see any terminal output for a few seconds while ndb starts up.