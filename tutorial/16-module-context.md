# Module Context

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
* [Debugging](./17-debugging.md)
* [Congratulations](./18-congratulations)

**Contents**  
* [Sharing Code](#sharing-code)
* [Exports](#exports)

## Sharing Code
[Back to Top](#module-context)

In all the examples we've seen so far, the `<script>` block contains code that runs when each component instance is initialized. For the vast majority of components, that's all you'll ever need.

Very occasionally, you'll need to run some code outside of an individual component instance. For example, you can play all five of these audio players simultaneously; it would be better if playing one stopped all the others.

We can do that by declaring a `<script context="module">` block. Code contained inside it will run once, when the module first evaluates, rather than when a component is instantiated.

```svelte
<script context="module">
  let current;
</script>
```

It's now possible for the components to 'talk' to each other without any state management.

```js
function stopOthers() {
  if (current && current !== audio) current.pause();
  current = audio;
}
```

**AudioPlayer.svelte**
```svelte
<script context="module">
	let current;
</script>

<script>
	export let src;
	export let title;
	export let composer;
	export let performer;

	let audio;
	let paused = true;

	function stopOthers() {
		if (current && current !== audio) current.pause();
		current = audio;
	}
</script>

<style>
	article { margin: 0 0 1em 0; max-width: 800px }
	h2, p { margin: 0 0 0.3em 0; }
	audio { width: 100%; margin: 0.5em 0 1em 0; }
	.playing { color: #ff3e00; }
</style>

<article class:playing={!paused}>
	<h2>{title}</h2>
	<p><strong>{composer}</strong> / performed by {performer}</p>

	<audio
		bind:this={audio}
		bind:paused
		on:play={stopOthers}
		controls
		{src}
	></audio>
</article>
```

**App.svelte**
```svelte
<script>
	import AudioPlayer from './AudioPlayer.svelte';
</script>

<!-- https://musopen.org/music/9862-the-blue-danube-op-314/ -->
<AudioPlayer
	src="https://sveltejs.github.io/assets/music/strauss.mp3"
	title="The Blue Danube Waltz"
	composer="Johann Strauss"
	performer="European Archive"
/>

<!-- https://musopen.org/music/43775-the-planets-op-32/ -->
<AudioPlayer
	src="https://sveltejs.github.io/assets/music/holst.mp3"
	title="Mars, the Bringer of War"
	composer="Gustav Holst"
	performer="USAF Heritage of America Band"
/>

<!-- https://musopen.org/music/8010-3-gymnopedies/ -->
<AudioPlayer
	src="https://sveltejs.github.io/assets/music/satie.mp3"
	title="Gymnopédie no. 1"
	composer="Erik Satie"
	performer="Prodigal Procrastinator"
/>

<!-- https://musopen.org/music/2567-symphony-no-5-in-c-minor-op-67/ -->
<AudioPlayer
	src="https://sveltejs.github.io/assets/music/beethoven.mp3"
	title="Symphony no. 5 in Cm, Op. 67 - I. Allegro con brio"
	composer="Ludwig van Beethoven"
	performer="European Archive"
/>

<!-- https://musopen.org/music/43683-requiem-in-d-minor-k-626/ -->
<AudioPlayer
	src="https://sveltejs.github.io/assets/music/mozart.mp3"
	title="Requiem in D minor, K. 626 - III. Sequence - Lacrymosa"
	composer="Wolfgang Amadeus Mozart"
	performer="Markus Staab"
/>
```

## Exports
[Back to Top](#module-context)

Anything exported from a `context="module"` script block becomes an export from the module itself. If we export a `stopAll` function from `AudioPlayer.svelte`:

```svelte
<script context="module">
  const elements = new Set();

  export function stopAll() {
    elements.forEach(element => element.pause());
  }
</script>
```

We can then import it in `App.svelte`:

```svelte
<script>
  import AudioPlayer, { stopAll } from './AudioPlayer.svelte';
</script>
```

and use it in an event handler:

```svelte
<button on:click={stopAll}>
  stop all audio
</button>
```

> You can't have a default export, because the component *is* the default export.

**AudioPlayer.svelte**
```svelte
<script context="module">
	const elements = new Set();
	
	export function stopAll() {
		elements.forEach(element => element.pause());
	}
</script>

<script>
	import { onMount } from 'svelte';

	export let src;
	export let title;
	export let composer;
	export let performer;

	let audio;
	let paused = true;

	onMount(() => {
		elements.add(audio);
		return () => elements.delete(audio);
	});

	function stopOthers() {
		elements.forEach(element => {
			if (element !== audio) element.pause();
		});
	}
</script>

<style>
	article { margin: 0 0 1em 0; max-width: 800px }
	h2, p { margin: 0 0 0.3em 0; }
	audio { width: 100%; margin: 0.5em 0 1em 0; }
	.playing { color: #ff3e00; }
</style>

<article class:playing={!paused}>
	<h2>{title}</h2>
	<p><strong>{composer}</strong> / performed by {performer}</p>

	<audio
		bind:this={audio}
		bind:paused
		on:play={stopOthers}
		controls
		{src}
	></audio>
</article>
```

**App.svelte**
```svelte
<script>
	import AudioPlayer, { stopAll } from './AudioPlayer.svelte';
</script>

<button on:click={stopAll}>
	stop all audio
</button>

<!-- https://musopen.org/music/9862-the-blue-danube-op-314/ -->
<AudioPlayer
	src="https://sveltejs.github.io/assets/music/strauss.mp3"
	title="The Blue Danube Waltz"
	composer="Johann Strauss"
	performer="European Archive"
/>

<!-- https://musopen.org/music/43775-the-planets-op-32/ -->
<AudioPlayer
	src="https://sveltejs.github.io/assets/music/holst.mp3"
	title="Mars, the Bringer of War"
	composer="Gustav Holst"
	performer="USAF Heritage of America Band"
/>

<!-- https://musopen.org/music/8010-3-gymnopedies/ -->
<AudioPlayer
	src="https://sveltejs.github.io/assets/music/satie.mp3"
	title="Gymnopédie no. 1"
	composer="Erik Satie"
	performer="Prodigal Procrastinator"
/>

<!-- https://musopen.org/music/2567-symphony-no-5-in-c-minor-op-67/ -->
<AudioPlayer
	src="https://sveltejs.github.io/assets/music/beethoven.mp3"
	title="Symphony no. 5 in Cm, Op. 67 - I. Allegro con brio"
	composer="Ludwig van Beethoven"
	performer="European Archive"
/>

<!-- https://musopen.org/music/43683-requiem-in-d-minor-k-626/ -->
<AudioPlayer
	src="https://sveltejs.github.io/assets/music/mozart.mp3"
	title="Requiem in D minor, K. 626 - III. Sequence - Lacrymosa"
	composer="Wolfgang Amadeus Mozart"
	performer="Markus Staab"
/>
```