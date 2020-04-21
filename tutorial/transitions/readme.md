# Transitions

**Topics**  
* [Introduction](../readme.md)
* [Reactivity](../reactivity/readme.md)
* [Props](../props/readme.md)
* [Logic](../logic/readme.md)
* [Events](../events/readme.md)
* [Bindings](../bindings/readme.md)
* [Lifecycle](../lifecycle/readme.md)
* [Stores](../stores/readme.md)
* [Motion](../motion/readme.md)
* [Animations](../animations/readme.md)

**Contents**  
* [The transition Directive](#the-transition-directive)
* [Adding Parameters](#adding-parameters)
* [In and Out](#in-and-out)
* [Custom CSS Transitions](#custom-css-transitions)
* [Custom JS Transitions](#custom-js-transitions)
* [Transition Events](#transition-events)
* [Local Transitions](#local-transitions)
* [Deferred Transitions](#deferred-transitions)

## The transition Directive
[Back to Top](#transitions)

We can make more appealing user interfaces by gracefully transitioning elements into and out of the DOM. Svelte makes this very easy with the `transition` directive.

```svelte
<script>
  import { fade } from 'svelte/transition';

  let visible = true;
</script>

<label>
  <input type="checkbox" bind:checked={visible}>
  visible
</label>

{#if visible}
  <p transition:fade>
    Fades in and out
  </p>
{/if}
```

## Adding Parameters
[Back to Top](#transitions)

```svelte
<script>
  import { fly } from 'svelte/transition';
  let visible = true;
</script>

<label>
  <input type="checkbox" bind:checked={visible}>
  visible
</label>

{#if visible}
  <p transition:fly={{ y: 200, duration: 2000 }}>
    Flies in and out
  </p>
{/if}
```

Note that the transition is *reversible* - if you toggle the checkbox while the transition is ongoing, it transitions from teh current point, rather than the beginning or the end.

## In and Out
[Back to Top](#transitions)

Instead of the `transition` directive, an element can have an `in` or an `out` directive, or both together.

```svelte
<script>
  import { fade, fly } from 'svelte/transition';
  let visible = true;
</script>

<label>
  <input type="checkbox" bind:checked={visible}>
  visible
</label>

{#if visible}
  <p in:fly={{ y: 200, duration: 2000 }} out:fade>
    Flies in, fades out
  </p>
{/if}
```

In this case, the transitions are *not* reversed.

## Custom CSS Transitions
[Back to Top](#transitions)

The `svelte/transition` module has a handful of built-in transitions, but it's very easy to create your own. By way of example, this is the source of the `fade` transition:

```js
function fade(node, {
  delay = 0,
  duration = 400
}) {
  const o = +getComputedStyle(node).opacity;

  return {
    delay,
    duration,
    css: t => `opacity: ${t * o}`
  };
}
```

The function takes two arguments - the node to which the transition is applied, and any paramters that were passed in - and returns a transition object which can have the following properties:

* `delay` - milliseconds before the transition begins
* `duration` - length of the transition in milliseconds
* `easing` - a `p => t` easing function
* `css` - a `(t, u) => css` function, where `u === 1 - t`.
* `tick` - a `(t, u) => { ... }` function that has some effect on the node

The `t` value is `0` at the beginning of an intro or the end of an outro, and `1` at the end of an intro or beginning of an outro.

Most of the time you should return the `css` property and *not* the `tick` property, as CSS animations run off the main thread to prevent jank where possible. Svelte 'simulates' the transition and constructs a CSS animation, then lets it run.

For example, teh `fade` transition generates a CSS animation somewhat like this:

```css
0% { opacity: 0 }
10% { opacity: 0.1 }
20% { opacity: 0.2 }
/* ... */
100% { opacity: 1 }
```

```svelte
<script>
  import { fade } from 'svelte/transition';
  import { elasticOut } from 'svelte/easing';

  let visible = true;

  function spin(node, { duration }) {
    return {
      duration,
      css: t => {
        const eased = elasticOut(t);

        return `
          transform: scale(${eased}) rotate(${eased * 1080}deg);
          color: hsl(
            ${~~(t * 360)},
            ${Math.min(100, 1000 - 1000 * t)}%,
            ${Math.min(50, 500 - 500 * t)}%
          );`
      }
    };
  }
</script>

<style>
  .centered {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
  }

  span {
    position: absolute;
    transform: translate(-50%, -50%);
    font-size: 4em;
  }
</style>

<label>
  <input type="checkbox" bind:checked={visible}>
  visible
</label>

{#if visible}
  <div class="centered" in:spin={{duration: 8000}} out:fade>
    <span>transitions!</span>
  </div>
{/if}
```

> Remember: with great power comes great responsibility.

## Custom JS Transitions
[Back to Top](#transitions)

While you should generally use CSS for transitions as much as possible, there are some effects that can't be achieved without JavaScript, such as a typewriter effect.

```svelte
<script>
  import { fade } from 'svelte/transition';

  let visible = false;

  function typewriter(node, { speed = 50 }) {
    const valid = (
      node.childNodes.length === 1 &&
      node.childNodes[0].nodeType === Node.TEXT_NODE
    );

    if (!valid) {
      throw new Error(`This trnasition only works on elements with a single text node child`);
    }

    const text = node.textContent;
    const duration = text.length * speed;

    return {
      duration,
      tick: t => {
        const i = ~~(text.length * t);
        node.textContent = text.slice(0, i);
      }
    };
  }
</script>

<label>
  <input type="checkbox" bind:checked={visible}>
  visible
</label>

{#if visible}
  <p in:typewriter out:fade>
    The quick brown fox jumps over the lazy dog
  </p>
{/if}
```

## Transition Events
[Back to Top](#transitions)

It can be useful to know when transitions are beginning and ending. Svelte dispatches events that you can listen to like any other DOM event.

```svelte
<script>
  import { fly } from 'svelte/transition';

  let visible = true;
  let status = 'waiting...';
</script>

<p>status: {status}</p>

<label>
  <input type="checkbox" bind:checked={visible}>
  visible
</label>

{#if visible}
  <p transition:fly={{ y: 200, duration: 2000 }}
     on:introstart={() => status = 'intro started'}
     on:outrostart={() => status = 'outro started'}
     on:introend={() => status = 'intro ended'}
     on:outroend={() => status = 'outro ended'}>
    Flies in and out
  </p>
{/if}
```

## Local Transitions
[Back to Top](#transitions)

Ordinarily, transitions will play on elements when any container block is added or destroyed. In the example here, toggling the visibility of the entire list also applies transitions to individual list elements.

Instead, we'd like transitions to play only when individual items are added and remove - in other words, when the user drags the slider.

We can achieve this with a *local* transition, which only plays when the immediate parent block is added or removed.

```svelte
<script>
  import { slide } from 'svelte/transition';

  let showItems = true;
  let i = 5;

  let items = [
    'one', 'two', 'three', 'four', 'five',
    'six', 'seven', 'eight', 'nine', 'ten'
  ];
</script>

<style>
  div {
    padding: 0.5em 0;
    border-top: 1px solid #eee;
  }
</style>

<label>
  <input type="checkbox" bind:checked={showItems}>
  show list
</label>

<label>
  <input type="range" bind:value={i} max=10>
</label>

{#if showItems}
  {#each items.slice(0, i) as item}
    <div transition:slide|local>
      {item}
    </div>
  {/each}
{/if}
```

## Deferred Transitions
[Back to Top](#transitions)

A particularly powerful feature of Svelte's transition engine is the ability to *defer* transitions, so that they can be coordinated between multiple elements.

Take this pair of todo lists, in which toggling a todo sends it to the opposite list. In the real world, objects don't behave like that - instead of disappearing and reappearing in another place, they move through a series of intermediate positions. Using motion can go a long way towards helping users understand what's happening in your app.

We can achieve this effect using the `crossfade` function, which creates a pair of transitions called `send` and `receive`. When an element is 'sent', it looks for a corresponding element being 'received', and generates a transition that transforms the element to its counterpart's position and fades it out. When an element is 'received', the reverse happens. If there is no counterpart, the `fallback` transition is used.

```svelte
<script>
  import { quintOut } from 'svelte/easing';
  import { crossfade } from 'svelte/transition';

  const [send, receive] = crossfade({
    duration: d => Math.sqrt(d * 200),
    fallback(node, params) {
      const style = getComputedStyle(node);
      const transform = style.transform === 'none'
        ? ''
        : style.transform;

      return {
        duration: 600,
        easing: quintOut,
        css: t => `
          transform: ${transform} scale(${t});
          opacity: ${t}
        `
      };
    }
  });

  let uid = 1;

  let todos = [
		{ id: uid++, done: false, description: 'write some docs' },
		{ id: uid++, done: false, description: 'start writing blog post' },
		{ id: uid++, done: true,  description: 'buy some milk' },
		{ id: uid++, done: false, description: 'mow the lawn' },
		{ id: uid++, done: false, description: 'feed the turtle' },
		{ id: uid++, done: false, description: 'fix some bugs' },
	];

	function add(input) {
		const todo = {
			id: uid++,
			done: false,
			description: input.value
		};

		todos = [todo, ...todos];
		input.value = '';
	}

	function remove(todo) {
		todos = todos.filter(t => t !== todo);
	}

	function mark(todo, done) {
		todo.done = done;
		remove(todo);
		todos = todos.concat(todo);
	}
</script>

<style>
  .board {
		display: grid;
		grid-template-columns: 1fr 1fr;
		grid-gap: 1em;
		max-width: 36em;
		margin: 0 auto;
	}

	.board > input {
		font-size: 1.4em;
		grid-column: 1/3;
	}

	h2 {
		font-size: 2em;
		font-weight: 200;
		user-select: none;
		margin: 0 0 0.5em 0;
	}

	label {
		position: relative;
		line-height: 1.2;
		padding: 0.5em 2.5em 0.5em 2em;
		margin: 0 0 0.5em 0;
		border-radius: 2px;
		user-select: none;
		border: 1px solid hsl(240, 8%, 70%);
		background-color:hsl(240, 8%, 93%);
		color: #333;
	}

	input[type="checkbox"] {
		position: absolute;
		left: 0.5em;
		top: 0.6em;
		margin: 0;
	}

	.done {
		border: 1px solid hsl(240, 8%, 90%);
		background-color:hsl(240, 8%, 98%);
	}

	button {
		position: absolute;
		top: 0;
		right: 0.2em;
		width: 2em;
		height: 100%;
		background: no-repeat 50% 50% url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cpath fill='%23676778' d='M12,2C17.53,2 22,6.47 22,12C22,17.53 17.53,22 12,22C6.47,22 2,17.53 2,12C2,6.47 6.47,2 12,2M17,7H14.5L13.5,6H10.5L9.5,7H7V9H17V7M9,18H15A1,1 0 0,0 16,17V10H8V17A1,1 0 0,0 9,18Z'%3E%3C/path%3E%3C/svg%3E");
		background-size: 1.4em 1.4em;
		border: none;
		opacity: 0;
		transition: opacity 0.2s;
		text-indent: -9999px;
		cursor: pointer;
	}

	label:hover button {
		opacity: 1;
	}
</style>

<div class='board'>
	<input
		placeholder="what needs to be done?"
		on:keydown={e => e.which === 13 && add(e.target)}>

	<div class='left'>
		<h2>todo</h2>
		{#each todos.filter(t => !t.done) as todo (todo.id)}
			<label
				in:receive="{{key: todo.id}}"
				out:send="{{key: todo.id}}">
				<input type=checkbox on:change={() => mark(todo, true)}>
				{todo.description}
				<button on:click="{() => remove(todo)}">remove</button>
			</label>
		{/each}
	</div>

	<div class='right'>
		<h2>done</h2>
		{#each todos.filter(t => t.done) as todo (todo.id)}
			<label
				class="done"
				in:receive="{{key: todo.id}}"
				out:send="{{key: todo.id}}">
				<input type=checkbox checked on:change={() => mark(todo, false)}>
				{todo.description}
				<button on:click="{() => remove(todo)}">remove</button>
			</label>
		{/each}
	</div>
</div>
```