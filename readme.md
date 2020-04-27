# Svelte Study

[Svelte](https://svelte.dev) is a radical new approach to building user interfaces. Whereas traditional frameworks like React and Vue do the bulk of their work in the *browser*, Svelte shifts that work into a *compile step* that happens when you build your app.

Instead of using techniques like virtual DOM diffing, Svelte writes code that surgically updates the DOM when the state of your app changes.

[Read the introductory blog post](https://svelte.dev/blog/svelte-3-rethinking-reactivity) to learn more.

[Sapper](https://sapper.svelte.dev) is a framework for building web applications of all sizes, with a beautiful development experience and flexible filesystem-based routing.

Unlike single-page apps, Sapper doesn't compromise on SEO, progressive enhancement or the initial load experience - but unlike traditional server-rendered apps, navigation is instantaneous for that app-like feel.

[Read the introductory blog post](https://svelte.dev/blog/sapper-towards-the-ideal-web-app-framework) to learn more.

[Polka](https://github.com/lukeed/polka) is an extremely minimal, highly performant Express.js alternative. Essentially, Polka is just a [native HTTP server](https://nodejs.org/dist/latest-v9.x/docs/api/http.html#http_class_http_server) with added support for routing, middleware, and sub-applications.

## Repository

* [Svelte](./svelte)
* [Sapper](./sapper)
* [Polka](./polka)
* [Rollup](./rollup)