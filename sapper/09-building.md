# Building

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
* [Exporting](./10-exporting.md)
* [Deployment](./11-deployment.md)
* [Security](./12-security.md)
* [Base URLs](./13-base-urls.md)
* [Testing](./14-testing.md)
* [Debugging](./15-debugging.md)
* [CSS Frameworks](./a1-css-frameworks.md)

**Contents**
* [sapper build](#sapper-build)

Up until now we've been using `sapper dev` to build our application and run a development server. But when it comes to production, we want to create a self-contained optimized build.

## sapper build
[Back to Top](#building)

This command packages up your application into the `__sapper__/build` directory. (You can change this to a custom directory, as well as controlling various other options - do `sapper build --help` for more information.).

The output is a Node app that you can run from the project root:

```bash
node __sapper__/build
```