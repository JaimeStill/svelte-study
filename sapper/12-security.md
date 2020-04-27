# Security

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
* [Base URLs](./13-base-urls.md)
* [Testing](./14-testing.md)
* [Debugging](./15-debugging.md)
* [CSS Frameworks](./a1-css-frameworks.md)

**Contents**
* [Content Security Policy (CSP)](#content-security-policy-csp)

By default, Sapper does not add security headers to your app, but you may add them yourself using middleware such as [Helmet](https://helmetjs.github.io/).

## Content Security Policy (CSP)
[Back to Top](#security)

Sapper generates inline `<script>` sections, which can fail to execute if [Content Security Policy (CSP)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) headers disallow arbitrary script execution (`unsafe-inline`).

To work around this, Sapper can inject a [nonce](https://www.troyhunt.com/locking-down-your-website-scripts-with-csp-hashes-nonces-and-report-uri/) which can be configured with middleware to emit the proper CSP headers. Here is an example using [Express](https://expressjs.com/) and [Helmet](https://helmetjs.github.io/):

**server.js**
```js
import uuidv4 from 'uuid/v4';
import helmet from 'helmet';

app.use((req, res, next) => {
  res.locals.nonce = uuidv4();
  next();
});

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      scriptSrc: [
        "'self'",
        (req, res) => `'nonce-${res.locals.nonce}'`
      ]
    }
  }
}));

app.use(sapper.middleware());
```

using `res.locals.nonce` in this way follows the convention set by [Helmet's CSP docs](https://helmetjs.github.io/docs/csp/#generating-nonces).