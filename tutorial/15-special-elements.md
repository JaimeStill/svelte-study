# Special Elements

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
* [Context API](./14-context-api.md)

**Contents**
* [\<svelte:self>](#-svelte:self-)
* [\<svelte:component>](#-svelte:component-)
* [\<svelte:window>](#-svelte:window-)
* [\<svelte:window> bindings](#-svelte:window-bindings)
* [\<svelte:body>](#-svelte:body-)
* [\<svelte:head>](#-svelte:head-)
* [\<svelte:options>](#-svelte:options-)

## \<svelte:self>
[Back to Top](#special-elements)

Svelte provides a variety of built-in elements. The first, `<svelte:self>`, allows a component to contain itself recursively.

It's useful for things like this folder tree view, where folders can contain *other* folders. In `Folder.svelte` we want to be able to do this:

```svelte
{#if file.type === 'folder'}
  <Folder {...file}/>
{:else}
  <File {...file}/>
{/if}
```

but that's impossible, because a file can't import itself. Instead, we use `<svelte:self>`:

```svelte
{#if file.type === 'folder'}
  <svelte:self {...file}/>
{:else}
  <File {...file}/>
{/if}
```

**File.svelte**
```svelte
<script>
	export let name;
	$: type = name.slice(name.lastIndexOf('.') + 1);
</script>

<style>
	span {
		padding: 0 0 0 1.5em;
		background: 0 0.1em no-repeat;
		background-size: 1em 1em;
	}
</style>

<span style="background-image: url(tutorial/icons/{type}.svg)">{name}</span>
```

**Folder.svelte**
```svelte
<script>
	import File from './File.svelte';

	export let expanded = false;
	export let name;
	export let files;

	function toggle() {
		expanded = !expanded;
	}
</script>

<style>
	span {
		padding: 0 0 0 1.5em;
		background: url(tutorial/icons/folder.svg) 0 0.1em no-repeat;
		background-size: 1em 1em;
		font-weight: bold;
		cursor: pointer;
	}

	.expanded {
		background-image: url(tutorial/icons/folder-open.svg);
	}

	ul {
		padding: 0.2em 0 0 0.5em;
		margin: 0 0 0 0.5em;
		list-style: none;
		border-left: 1px solid #eee;
	}

	li {
		padding: 0.2em 0;
	}
</style>

<span class:expanded on:click={toggle}>{name}</span>

{#if expanded}
	<ul>
		{#each files as file}
			<li>
				{#if file.type === 'folder'}
					<svelte:self {...file}/>
				{:else}
					<File {...file}/>
				{/if}
			</li>
		{/each}
	</ul>
{/if}
```

**App.svelte**
```svelte
<script>
	import Folder from './Folder.svelte';

	let root = [
		{
			type: 'folder',
			name: 'Important work stuff',
			files: [
				{ type: 'file', name: 'quarterly-results.xlsx' }
			]
		},
		{
			type: 'folder',
			name: 'Animal GIFs',
			files: [
				{
					type: 'folder',
					name: 'Dogs',
					files: [
						{ type: 'file', name: 'treadmill.gif' },
						{ type: 'file', name: 'rope-jumping.gif' }
					]
				},
				{
					type: 'folder',
					name: 'Goats',
					files: [
						{ type: 'file', name: 'parkour.gif' },
						{ type: 'file', name: 'rampage.gif' }
					]
				},
				{ type: 'file', name: 'cat-roomba.gif' },
				{ type: 'file', name: 'duck-shuffle.gif' },
				{ type: 'file', name: 'monkey-on-a-pig.gif' }
			]
		},
		{ type: 'file', name: 'TODO.md' }
	];
</script>

<Folder name="Home" files={root} expanded/>
```

## \<svelte:component>
[Back to Top](#special-elements)

A component can change its category altogether with `<svelte:component>`. Instead of a sequence of `if` blocks:

```svelte
{#if selected.color === 'red'}
  <RedThing />
{:else if selected.color === 'green'}
  <GreenThing />
{:else if selected.color === 'blue'}
  <BlueThing />
{/if}
```

we can have a single dynamic component:

```svelte
<svelte:component this={selected.component}/>
```

The `this` value can be any component constructor, or a falsy value - if it's falsy, no component is rendered.

**RedThing.svelte**
```svelte
<style>
	strong { color: red; }
</style>

<strong>red thing</strong>
```

**GreenThing.svelte**
```svelte
<style>
	strong { color: green; }
</style>

<strong>green thing</strong>
```

**BlueThing.svelte**
```svelte
<style>
	strong { color: blue; }
</style>

<strong>blue thing</strong>
```

**App.svelte**
```svelte
<script>
	import RedThing from './RedThing.svelte';
	import GreenThing from './GreenThing.svelte';
	import BlueThing from './BlueThing.svelte';

	const options = [
		{ color: 'red',   component: RedThing   },
		{ color: 'green', component: GreenThing },
		{ color: 'blue',  component: BlueThing  },
	];

	let selected = options[0];
</script>

<select bind:value={selected}>
	{#each options as option}
		<option value={option}>{option.color}</option>
	{/each}
</select>

<svelte:component this={selected.component}/>
```

## \<svelte:window>
[Back to Top](#special-elements)

Just as you can add event listeners to any DOM element, you can add event listeners to the `window` object with `<svelte:window>`.

> As with DOM elements, you can use [event modifiers](./04-events.md#event-modifiers) like `preventDefault`.

**App.svelte**
```svelte
<script>
	let key;
	let keyCode;

	function handleKeydown(event) {
		key = event.key;
		keyCode = event.keyCode;
	}
</script>

<style>
	div {
		display: flex;
		height: 100%;
		align-items: center;
		justify-content: center;
		flex-direction: column;
	}

	kbd {
		background-color: #eee;
		border-radius: 4px;
		font-size: 6em;
		padding: 0.2em 0.5em;
		border-top: 5px solid rgba(255,255,255,0.5);
		border-left: 5px solid rgba(255,255,255,0.5);
		border-right: 5px solid rgba(0,0,0,0.2);
		border-bottom: 5px solid rgba(0,0,0,0.2);
		color: #555;
	}
</style>

<svelte:window on:keydown={handleKeydown}/>

<div style="text-align: center">
	{#if key}
		<kbd>{key === ' ' ? 'Space' : key}</kbd>
		<p>{keyCode}</p>
	{:else}
		<p>Focus this window and press any key</p>
	{/if}
</div>
```

## \<svelte:window> bindings
[Back to Top](#special-elements)

You can also bind to certain properties of `window`, such as `scrollY`.

The list of properties you can bind to is as follows:
* `innerWidth`
* `innerHeight`
* `outerWidth`
* `outerHeight`
* `scrollX`
* `scrollY`
* `online` - an alias for `window.navigator.onLine`

All except `scrollX` and `scrollY` are readonly.

**App.svelte**
```svelte
<script>
	const layers = [0, 1, 2, 3, 4, 5, 6, 7, 8];

	let y;
</script>

<style>
	.parallax-container {
		position: fixed;
		width: 2400px;
		height: 712px;
		left: 50%;
		transform: translate(-50%,0);
	}

	.parallax-container img {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		will-change: transform;
	}

	.parallax-container img:last-child::after {
		content: '';
		position: absolute;
		width: 100%;
		height: 100%;
		background: rgb(45,10,13);
	}

	.text {
		position: relative;
		width: 100%;
		height: 300vh;
		color: rgb(220,113,43);
		text-align: center;
		padding: 4em 0.5em 0.5em 0.5em;
		box-sizing: border-box;
		pointer-events: none;
	}

	span {
		display: block;
		font-size: 1em;
		text-transform: uppercase;
		will-change: transform, opacity;
	}

	.foreground {
		position: absolute;
		top: 711px;
		left: 0;
		width: 100%;
		height: calc(100% - 712px);
		background-color: rgb(32,0,1);
		color: white;
		padding: 50vh 0 0 0;
	}

	:global(body) {
		margin: 0;
		padding: 0;
		background-color: rgb(253, 174, 51);
	}
</style>

<svelte:window bind:scrollY={y}/>

<a class="parallax-container" href="https://www.firewatchgame.com">
	{#each layers as layer}
		<img
			style="transform: translate(0,{-y * layer / (layers.length - 1)}px)"
			src="https://www.firewatchgame.com/images/parallax/parallax{layer}.png"
			alt="parallax layer {layer}"
		>
	{/each}
</a>

<div class="text">
	<span style="opacity: {1 - Math.max(0, y / 40)}">
		scroll down
	</span>

	<div class="foreground">
		You have scrolled {y} pixels
	</div>
</div>
```

## \<svelte:body>
[Back to Top](#special-elements)

Similar to `<svelte:window>`, the `<svelte:body>` element allows you to listen for events that fire on `document.body`. This is useful with the `mouseenter` and `mouseleave` events, which don't fire on `window`.

**App.svelte**
```svelte
<script>
	let hereKitty = false;

	const handleMouseenter = () => hereKitty = true;
	const handleMouseleave = () => hereKitty = false;
</script>

<style>
	img {
		position: absolute;
		left: 0;
		bottom: -60px;
		transform: translate(-80%, 0) rotate(-30deg);
		transform-origin: 100% 100%;
		transition: transform 0.4s;
	}

	.curious {
		transform: translate(-15%, 0) rotate(0deg);
	}

	:global(body) {
		overflow: hidden;
	}
</style>

<svelte:body
	on:mouseenter={handleMouseenter}
	on:mouseleave={handleMouseleave}
/>

<!-- creative commons BY-NC http://www.pngall.com/kitten-png/download/7247 -->
<img
	class:curious={hereKitty}
	alt="Kitten wants to know what's going on"
	src="tutorial/kitten.png"
>
```

## \<svelte:head>
[Back to Top](#special-elements)

The `<svelte:head>` element allows you to insert elements inside the `<head>` of your document.

> In server-side rendering (SSR) mode, contents of `<svelte:head>` are returned separately from the rest of your HTML.

**App.svelte**
```svelte
<svelte:head>
	<link rel="stylesheet" href="tutorial/dark-theme.css">
</svelte:head>

<h1>Hello world!</h1>
```

## \<svelte:options>
[Back to Top](#special-elements)

Lastly, `<svelte:options>` allows you to specify compiler options.

We'll use the `immutable` option as an example. In this app, the `<Todo>` component flashes whenever it receives new data. Clicking on one of the items toggles its `done` state by creating an updated `todos` array. This causes the *other* `<Todo>` items to flash, even though they don't end up making any changes to the DOM.

We can optimise this by telling the `<Todo>` component to expect *immutable* data. This means that we're promising never to *mutate* the `todo` prop, but will instead create new todo objects whenever things change.

Add this to the top of the `Todo.svelte` file:

```svelte
<svelte:options immutable={true}/>
```

> You can shorten this to `<svelte:options immutable/>` if you prefer.

Now, when you toggle todos by clicking on them, only the updated component flashes.

The options that can be set here are:
* `immutable={true}` - you never use mutable data, so the compiler can do simple refential equality checks to determine if values have changed.
* `immutable={false}` - the default. Svelte will be more conservative about whether or not mutable objects have changed.
* `accessors={true}` - adds getters and setters for the component's props.
* `accessors={false}` - the default.
* `namespace="..."` - the namespace where this component will be used, most commonly `"svg"`.
* `tag="..."` - the name to use when compiling this component as a custom element.

Consult the [API reference](https://svelte.dev/docs#svelte_options) for more information on these options.

**flash.js**
```js
export default function flash(element) {
	requestAnimationFrame(() => {
		element.style.transition = 'none';
		element.style.color = 'rgba(255,62,0,1)';
		element.style.backgroundColor = 'rgba(255,62,0,0.2)';

		setTimeout(() => {
			element.style.transition = 'color 1s, background 1s';
			element.style.color = '';
			element.style.backgroundColor = '';
		});
	});
}
```

**Todo.svelte**
```svelte
<svelte:options immutable={true}/>
<script>
	import { afterUpdate } from 'svelte';
	import flash from './flash.js';

	export let todo;

	let div;

	afterUpdate(() => {
		flash(div);
	});
</script>

<style>
	div {
		cursor: pointer;
		line-height: 1.5;
	}
</style>

<!-- the text will flash red whenever
     the `todo` object changes -->
<div bind:this={div} on:click>
	{todo.done ? '👍': ''} {todo.text}
</div>
```

**App.svelte**
```svelte
<script>
	import Todo from './Todo.svelte';

	let todos = [
		{ id: 1, done: true, text: 'wash the car' },
		{ id: 2, done: false, text: 'take the dog for a walk' },
		{ id: 3, done: false, text: 'mow the lawn' }
	];

	function toggle(toggled) {
		todos = todos.map(todo => {
			if (todo === toggled) {
				// return a new object
				return {
					id: todo.id,
					text: todo.text,
					done: !todo.done
				};
			}

			// return the same object
			return todo;
		});
	}
</script>

<h2>Todos</h2>
{#each todos as todo}
	<Todo {todo} on:click={() => toggle(todo)}/>
{/each}
```