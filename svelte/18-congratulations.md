# Congratulations

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
* [Special Elements](./15-special-elements.md)
* [Module Context](./16-module-context.md)
* [Debugging](./17-debugging.md)

You've now finished the Svelte tutorial and are ready to start building apps. You can refer back to individual chapters at any time, or continue your learning via the [API reference](https://svelte.dev/docs), [Examples](https://svelte.dev/examples), and [Blog](https://svelte.dev/blog).

To get setup in your local development environment, check out [the quickstart guide](https://svelte.dev/blog/the-easiest-way-to-get-started).

If you're looking for a more expansive framework that includes routing, server-side rendering and everything else, take a look at [Sapper](https://sapper.svelte.dev/).

**App.svelte**
```svelte
<script>
	import { onMount } from 'svelte';

	let characters = ['🥳', '🎉', '✨'];

	let confetti = new Array(100).fill()
		.map((_, i) => {
			return {
				character: characters[i % characters.length],
				x: Math.random() * 100,
				y: -20 - Math.random() * 100,
				r: 0.1 + Math.random() * 1
			};
		})
		.sort((a, b) => a.r - b.r);

	onMount(() => {
		let frame;

		function loop() {
			frame = requestAnimationFrame(loop);

			confetti = confetti.map(emoji => {
				emoji.y += 0.7 * emoji.r;
				if (emoji.y > 120) emoji.y = -20;
				return emoji;
			});
		}

		loop();

		return () => cancelAnimationFrame(frame);
	});
</script>

<style>
	:global(body) {
		overflow: hidden;
	}

	span {
		position: absolute;
		font-size: 5vw;
	}
</style>

{#each confetti as c}
	<span style="left: {c.x}%; top: {c.y}%; transform: scale({c.r})">{c.character}</span>
{/each}
```