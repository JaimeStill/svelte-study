# Context API

**Topics**  
* [Introduction](./readme.md)
* [Reactivity](./01-reactivity.md)
* [Props](./02-props.md)
* [Logic](./03-logic.md)
* [Events](./04-events.md)
* [Bindings](./05-bindings.md)
* [Lifecycle](./06-lifecycle.md)
* [Stores](./07-stores.md)
* [Motion](./08-motion.md)
* [Transitions](./09-transitions.md)
* [Animations](./10-animations.md)
* [Actions](./11-actions.md)
* [Classes](./12-classes.md)
* [Component Composition](./13-component-composition.md)
* [Special Elements](./15-special-elements.md)

**Contents**
* [Context Keys](#context-keys)
* [Contexts vs. Stores](#contexts-vs.-stores)

The context API provides a mechanism for components to 'talk' to each other wihtout passing around data nad functions as props, or dispatching lots of events. It's an advanced feature, but a useful one.

Take this example app using a [Mapbox GL](https://docs.mapbox.com/mapbox-gl-js/overview/) map. We'd like to display the markers, using the `<MapMarker>` component, but we don't want to have to pass around a reference to the underlying Mapbox instance as a prop of each component.

There are two halves to the context API - `setContext` and `getContext`. If a component calls `setContext(key, context)`, then any *child* component can retrieve the context with `const context = getContext(key)`.

Let's set the context first. In `Map.svelte`, import `setContext` from `svelte` and `key` from `mapbox.js` and call `setContext`:

```svelte
import { onMount, setContext } from 'svelte';
import { mapbox, key } from './mapbox.js';

setContext(key, {
  getMap: () => map
});
```

The context object can be anything you like. Like [lifecycle functions](./06-lifecycle.md), `setContext` and `getContext` must be called during component initialization; since `map` isn't created until the component has mounted, our context object contains a `getMap` function rather than `map` itself.

On the other side of the equation, in `MapMarker.svelte`, we can now get a reference to the Mapbox instance:

```svelte
import { getContext } from 'svelte';
import { mapbox, key } from './mapbox.js';

const { getMap } = getContext(key);
const map = getMap();
```

The markers can now add themselves to the map.

> A more finished version of `<MapMarker>` would also handle removal and prop changes, but we're only demonstrating context here.

## Context Keys
[Back to Top](#context-api)

In `mapbox.js` you'll see this line:

```js
const key = {};
```

We can use anything as a key - we could do `setContext('mapbox', ...)` for example. The downside of using a string is that different component libraries might accidentally use the same one; using an object literal means the keys are guaranteed not to conflict in any circumstance (since an object only has referential equality to itself, i.e. `{} !== {}` whereas `"x" === "x"`), even when you have multiple different contexts operating across many component layers.

## Contexts vs. Stores
[Back to Top](#context-api)

Contexts and stores seem similar. They differ in that stores are available to *any* part of an app, while a context is only available to *a component and its descendants*. This can be helpful if you want to use several instances of a component without the state of one interfering with the state of the others.

In fact, you might use the two together. Since context is not reactive, values that change over time should be represented as stores:

```js
const { these, are, stores } = getContext(...);
```

**mapbox.js**
```js
import mapbox from 'mapbox-gl';

// https://docs.mapbox.com/help/glossary/access-token/
mapbox.accessToken = MAPBOX_ACCESS_TOKEN;

const key = {};

export { mapbox, key };
```

**Map.svelte**
```svelte
<script>
	import { onMount, setContext } from 'svelte';
	import { mapbox, key } from './mapbox.js';

	setContext(key, {
		getMap: () => map
	});

	export let lat;
	export let lon;
	export let zoom;

	let container;
	let map;

	onMount(() => {
		const link = document.createElement('link');
		link.rel = 'stylesheet';
		link.href = 'https://unpkg.com/mapbox-gl/dist/mapbox-gl.css';

		link.onload = () => {
			map = new mapbox.Map({
				container,
				style: 'mapbox://styles/mapbox/streets-v9',
				center: [lon, lat],
				zoom
			});
		};

		document.head.appendChild(link);

		return () => {
			map.remove();
			link.parentNode.removeChild(link);
		};
	});
</script>

<style>
	div {
		width: 100%;
		height: 100%;
	}
</style>

<div bind:this={container}>
	{#if map}
		<slot></slot>
	{/if}
</div>
```

**MapMarker.svelte**
```svelte
<script>
	import { getContext } from 'svelte';
	import { mapbox, key } from './mapbox.js';

	const { getMap } = getContext(key);
	const map = getMap();

	export let lat;
	export let lon;
	export let label;

	const popup = new mapbox.Popup({ offset: 25 })
		.setText(label);

	const marker = new mapbox.Marker()
		.setLngLat([lon, lat])
		.setPopup(popup)
		.addTo(map);
</script>
```

**App.svelte**
```svelte
<script>
	import Map from './Map.svelte';
	import MapMarker from './MapMarker.svelte';
</script>

<Map lat={35} lon={-84} zoom={3.5}>
	<MapMarker lat={37.8225} lon={-122.0024} label="Svelte Body Shaping"/>
	<MapMarker lat={33.8981} lon={-118.4169} label="Svelte Barbershop & Essentials"/>
	<MapMarker lat={29.7230} lon={-95.4189} label="Svelte Waxing Studio"/>
	<MapMarker lat={28.3378} lon={-81.3966} label="Svelte 30 Nutritional Consultants"/>
	<MapMarker lat={40.6483} lon={-74.0237} label="Svelte Brands LLC"/>
	<MapMarker lat={40.6986} lon={-74.4100} label="Svelte Medical Systems"/>
</Map>
```