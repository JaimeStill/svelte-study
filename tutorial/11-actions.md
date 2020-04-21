# Actions

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
* [Classes](./12-classes.md)
* [Component Composition](./13-component-composition.md)
* [Context API](./14-context-api.md)
* [Special Elements](./15-special-elements.md)

**Contents**  
* [The use Directive](#the-use-directive)
* [Adding Parameters](#adding-parameters)

Actions are essential element-level lifecycle functions. They're useful for things like:
* interfacing with third-party libraries
* lazy-loaded images
* tooltips
* adding custom event handlers

## The use Directive
[Back to Top](#actions)

In this app, we want to make the orange box 'pannable'. It has event handlers for the `panstart`, `panmove`, and `panend` events, but these aren't native DOM events. We have to dispatch them ourselves.

First import the `pannable` function:

```js
import { pannable } from './pannable.js';
```

then use it with the element:

```svelte
<div class="box"
     use:pannable
     on:panstart={handlePanStart}
     on:panmove={handlePanMove}
     on:panend={handlePanEnd}
     style="transform:
       translate({$coords.x}px, {$coords.y}px)
       rotate({$coords.x * 0.2}deg)"></div>
```

Open the `pannable.js` file. Like transition functions, an action function receives a `node` and some optional paramters, and returns an action object. That object can have a `destroy` function, which is called when the element is unmounted.

We want to fire the `panstart` event when the user mouses down on the element, `panmove` events (with `dx` and `dy` properties showing how far the mouse moved) when they drag it, and `panend` events when they mouse up.

**pannable.js**  
```js
export function pannable(node) {
	let x;
	let y;

	function handleMousedown(event) {
		x = event.clientX;
		y = event.clientY;

		node.dispatchEvent(new CustomEvent('panstart', {
			detail: { x, y }
		}));

		window.addEventListener('mousemove', handleMousemove);
		window.addEventListener('mouseup', handleMouseup);
	}

	function handleMousemove(event) {
		const dx = event.clientX - x;
		const dy = event.clientY - y;
		x = event.clientX;
		y = event.clientY;

		node.dispatchEvent(new CustomEvent('panmove', {
			detail: { x, y, dx, dy }
		}));
	}

	function handleMouseup(event) {
		x = event.clientX;
		y = event.clientY;

		node.dispatchEvent(new CustomEvent('panend', {
			detail: { x, y }
		}));

		window.removeEventListener('mousemove', handleMousemove);
		window.removeEventListener('mouseup', handleMouseup);
	}

	node.addEventListener('mousedown', handleMousedown);

	return {
		destroy() {
			node.removeEventListener('mousedown', handleMousedown);
		}
	};
}
```

**App.svelte**  
```svelte
<script>
	import { spring } from 'svelte/motion';
	import { pannable } from './pannable.js';

	const coords = spring({ x: 0, y: 0 }, {
		stiffness: 0.2,
		damping: 0.4
	});

	function handlePanStart() {
		coords.stiffness = coords.damping = 1;
	}

	function handlePanMove(event) {
		coords.update($coords => ({
			x: $coords.x + event.detail.dx,
			y: $coords.y + event.detail.dy
		}));
	}

	function handlePanEnd(event) {
		coords.stiffness = 0.2;
		coords.damping = 0.4;
		coords.set({ x: 0, y: 0 });
	}
</script>

<style>
	.box {
		--width: 100px;
		--height: 100px;
		position: absolute;
		width: var(--width);
		height: var(--height);
		left: calc(50% - var(--width) / 2);
		top: calc(50% - var(--height) / 2);
		border-radius: 4px;
		background-color: #ff3e00;
		cursor: move;
	}
</style>

<div class="box"
		 use:pannable
		 on:panstart={handlePanStart}
		 on:panmove={handlePanMove}
		 on:panend={handlePanEnd}
		 style="transform:
		   translate({$coords.x}px,{$coords.y}px)
			 rotate({$coords.x * 0.2}deg)"></div>
```

## Adding Parameters
[Back to Top](#actions)

Like transitions and animations, an action can take an argument, which the action function will be called with alongside the element it belongs to.

Here, we're using a `longpress` action that fires an event with the same name whenever the user presses and holds the button for a given duration. Right now, if you switch over to the `longpress.js` file, you'll see it's hardcoded to 500ms.

We can change the action function to accept a `duration` as a second argument, and pass that `duration` to the `setTimeout` call.

We can then pass the `duration` value to the action, and add an `update` method in `longpress.js`. This will be called whenever the argument changes.

**longpress.js**  
```js
export function longpress(node, duration) {
	let timer;
	
	const handleMousedown = () => {
		timer = setTimeout(() => {
			node.dispatchEvent(
				new CustomEvent('longpress')
			);
		}, duration);
	};
	
	const handleMouseup = () => {
		clearTimeout(timer)
	};

	node.addEventListener('mousedown', handleMousedown);
	node.addEventListener('mouseup', handleMouseup);

	return {
		update(newDuration) {
			duration = newDuration;
		},
		destroy() {
			node.removeEventListener('mousedown', handleMousedown);
			node.removeEventListener('mouseup', handleMouseup);
		}
	};
}
```

**App.svelte**  
```svelte
<script>
	import { longpress } from './longpress.js';

	let pressed = false;
	let duration = 2000;
</script>

<label>
	<input type=range bind:value={duration} max={2000} step={100}>
	{duration}ms
</label>

<button use:longpress={duration}
	on:longpress="{() => pressed = true}"
	on:mouseenter="{() => pressed = false}"
>press and hold</button>

{#if pressed}
	<p>congratulations, you pressed and held for {duration}ms</p>
{/if}
```

> If you need to pass multiplle arguments to an action, combine them into a single object, as in `use:longpress={{duration, spiciness}}`