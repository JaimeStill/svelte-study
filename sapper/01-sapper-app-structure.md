# Sapper App Structure

**Topics**  
* [Introduction](./readme.md)
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
* [Debugging](./15-debugging.md)

**Contents**
* [package.json](#packagejson)

If you take a look inside the [sapper-template](https://github.com/sveltejs/sapper-template) repo, you'll see some files that Sapper expects to find:

* `package.json`
* **src**
    * **routes**
        * *your routes here*
        * `_error.svelte`
        * `index.svelte`
    * `client.js`
    * `server.js`
    * `service-worker.js`
    * `template.html`
* **static**
    * *your files here*
* `rollup.config.js` / `webpack.config.js`

When you first run Sapper, it will create an additional `__sapper__` directory containing generated files.

You'll notice a few extra files and a `cypress` directory which relates to [testing](https://sapper.svelte.dev/docs#Testing).

## package.json
[Back to Top](#sapper-app-structure)

Your `package.json` contains your app's dependencies and defines a number of scripts:

* `npm run dev` - start the app in development mode, and watch source files for changes.
* `npm run build` - build the app in production mode.
* `npm run export` - bake out a static version, if applicable (see [exporting](https://sapper.svelte.dev/docs#Exporting)).
* `npm start` - start the app in production mode after you've built it.
* `npm test` - run the tests (see [testing](https://sapper.svelte.dev/docs#Testing)).