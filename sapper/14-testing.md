# Testing

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
* [Debugging](./15-debugging.md)

You can use whatever testing frameworks and libraries you'd like. The defaut in [sapper-template](https://github.com/sveltejs/sapper-template) is [Cypress](https://cypress.io/).

```bash
npm test
```

This will start the server and open Cypress. You can (and should!) add your own tests in `cypress/integration/spec.js` - consult the [docs](https://docs.cypress.io/guides/overview/why-cypress.html) for more information.