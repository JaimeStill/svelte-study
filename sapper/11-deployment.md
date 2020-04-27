# Deployment

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
* [Security](./12-security.md)
* [Base URLs](./13-base-urls.md)
* [Testing](./14-testing.md)
* [Debugging](./15-debugging.md)
* [CSS Frameworks](./a1-css-frameworks.md)

**Contents**
* [Deploying to Now](#deploying-to-now)
* [Deploying Service Workers](#deploying-service-workers)

Sapper apps can run anywhere that supports Node 8 or higher.

## Deploying to Now
[Back to Top](#deployment)

We can very easily deploy our apps to [Now](https://zeit.co/now):

```bash
npm install -g now
now
```

This will upload the source code to Now, whereupon it will do `npm run build` and `npm start` and give you a URL for the deployed app.

For other hosting environments, you may need to do `npm run build` yourself.

## Deploying Service Workers
[Back to Top](#deployment)

Sapper makes the Service Worker file (`service-worker.js`) unique by including a timestamp in the source code (caluclated using `Date.now()`).

In environments where the app is deployed to multiple servers (such as [Now](https://zeit.co/now)), it is advisable to use a consistent timestamp for all deployments. Otherwise, users may run into issues where the Service Worker updates unexpectedly because the app hits server 1, then server 2, and they have slightly different timestamps.

To override Sapper's timestamp, you can use an environment variable (e.g. `SAPPER_TIMESTAMP`) and then modify the `service-worker.js`:

```js
const timestamp - process.env.SAPPER_TIMESTAMP;

const ASSETS = `cache${timestamp}`;

export default {
  plugins: [
    replace({
      `process.env.SAPPER_TIMESTAMP`: process.env.SAPPER_TIMESTAMP || Date.now()
    })
  ]
}
```

Then you can set it using the environment varable, e.g.:

```bash
SAPPER_TIMESTAMP=$(date + %s%3N) npm run build
```

When deploying to [Now](https://zeit.co/now), you can pass the environment variable into Now itself:

```bash
now -e SAPPER_TIMESTAMP=$(date + %s%3N)
```