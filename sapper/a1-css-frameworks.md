# CSS Framework Setup

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
* [Debugging](./15-debugging.md)

**Contents**
* [Bulma](#bulma)
  [Font Awesome](#font-awesome)
* [Spectre](#spectre)
* [Tailwinds CSS](#tailwinds-css)

## Bulma
[Back to Top](#css-framework-setup)

```bash
yarn add -D bulma sass svelte-preprocess
```

**rollup.config.js**
```js
import { scss } from 'svelte-preprocess';
// export default...
svelte({
  preprocess: [
    scss({
      omitSourceMapUrl: true,
      includePaths: [
        'node_modules',
        'src'
      ]
    })
  ]
})
```

**src/scss/global.scss**
```scss
@import 'node_modules/bulma/bulma.sass';
```

### Font Awesome
[Back to Top](#css-framework-setup)

```bash
yarn add -D @fortawesome/fontawesome-free
```

**client.js**
```js
import '@fortawesome/fontawesome-free/js/all';
```

## Spectre
[Back to Top](#css-framework-setup)

```bash
yarn add -D spectre.css rollup-plugin-css-only
```

**rollup.config.js**
```js
import css from 'rollup-plugin-css-only';
// export default...
plugins: [
  css({ output: 'static/index.css' }),
  // svelte({})...
]
```

**src/client.js**
```js
import 'spectre.css/dist/spectre.min.css';
import 'spectre.css/dist/spectre-exp.min.css';
import 'spectre.css/dist/spectre-icons.min.css';
```

**src/template.html**
```html
<link rel="stylesheet" href="index.css"/>
```

## Tailwinds CSS
[Back to Top](#css-framework-setup)

```bash
yarn add -D tailwindcss postcss-cli @fullhuman/postcss-purgecss
./node_modules/.bin/tailwind init tailwind.js
```

Create a `postcss.config.js` file:

```js
const tailwindcss = require('tailwindcss');

const purgecss = require('@fullhuman/postcss-purgecss')({
  content: ['./src/**/*.svelte', './src/**/*.html'],
  defaultExtractor: content => content.match(/[A-Za-z0-9-_:/]+/g) || []
});

module.exports = {
  plugins: [
    tailwindcss('./tailwind.js'),
    ...(process.env.NODE_ENV === "production" ? [purgecss] : [])
  ]
};
```

Create `static/tailwind.css`:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Update `package.json`:

```js
"scripts": {
  "watch:tailwind": "postcss static/tailwind.css -o static/index.css -w",
  "build:tailwind": "SET NODE_ENV=production postcss static/tailwind.css -o static/index.css",
  "build": "yarn build:tailwind && sapper build --legacy"
}
```

Add stylesheet link to `src/template.html`:

```html
<link rel="stylesheet" href="index.css"/>
```

**Compiles and hot-reloads for development**
```bash
yarn watch:tailwind
```

```bash
yarn dev
```

**compiles and minifies for production**
```bash
yarn build
```