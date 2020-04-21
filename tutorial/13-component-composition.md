# Component Composition

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
* [Context API](./14-context-api.md)

**Contents**  
* [Slots](#slots)
* [Slot Fallbacks](#slot-fallbacks)
* [Named Slots](#named-slots)
* [Slot Props](#slot-props)

## Slots
[Back to Top](#component-composition)

Just like elements can have children:

```html
<div>
  <p>I'm a child of the div</p>
</div>
```

So can components. Before a component can accept children, though, it needs to know where to put them. We do this with the `<slot>` element.

**Box.svelte**  
```svelte
<style>
	.box {
		width: 300px;
		border: 1px solid #aaa;
		border-radius: 2px;
		box-shadow: 2px 2px 8px rgba(0,0,0,0.1);
		padding: 1em;
		margin: 0 0 1em 0;
	}
</style>

<div class="box">
	<slot></slot>
</div>
```

**App.svelte**  
```svelte
<script>
	import Box from './Box.svelte';
</script>

<Box>
	<h2>Hello!</h2>
	<p>This is a box. It can contain anything.</p>
</Box>
```

## Slot Fallbacks
[Back to Top](#component-composition)

A component can specify *fallbacks* for any slots that are left empty by putting content inside the `<slot>` element.

**Box.svelte**  
```svelte
<style>
	.box {
		width: 300px;
		border: 1px solid #aaa;
		border-radius: 2px;
		box-shadow: 2px 2px 8px rgba(0,0,0,0.1);
		padding: 1em;
		margin: 0 0 1em 0;
	}
</style>

<div class="box">
	<slot>
		<em>no content was provided</em>
	</slot>
</div>
```

**App.svelte**
```svelte
<script>
	import Box from './Box.svelte';
</script>

<Box>
	<h2>Hello!</h2>
	<p>This is a box. It can contain anything.</p>
</Box>

<Box />
```

## Named Slots
[Back to Top](#component-composition)

The previous example contained a *default slot*, which renders the direct children of a component. Sometimes you need more control over placement. In these cases, we can use *named slots*.

**ContactCard.svelte**
```svelte
<style>
	.contact-card {
		width: 300px;
		border: 1px solid #aaa;
		border-radius: 2px;
		box-shadow: 2px 2px 8px rgba(0,0,0,0.1);
		padding: 1em;
	}

	h2 {
		padding: 0 0 0.2em 0;
		margin: 0 0 1em 0;
		border-bottom: 1px solid #ff3e00
	}

	.address, .email {
		padding: 0 0 0 1.5em;
		background:  0 0 no-repeat;
		background-size: 20px 20px;
		margin: 0 0 0.5em 0;
		line-height: 1.2;
	}

	.address { background-image: url(tutorial/icons/map-marker.svg) }
	.email   { background-image: url(tutorial/icons/email.svg) }
	.missing { color: #999 }
</style>

<article class="contact-card">
	<h2>
		<slot name="name">
			<span class="missing">Unknown name</span>
		</slot>
	</h2>

	<div class="address">
		<slot name="address">
			<span class="missing">Unknown address</span>
		</slot>
	</div>

	<div class="email">
		<slot name="email">
			<span class="missing">Unknown email</span>
		</slot>
	</div>
</article>
```

**App.svelte**
```svelte
<script>
	import ContactCard from './ContactCard.svelte';
</script>

<ContactCard>
	<span slot="name">
		P. Sherman
	</span>

	<span slot="address">
		42 Wallaby Way<br>
		Sydney
	</span>
</ContactCard>
```

## Slot Props
[Back to Top](#component-composition)

In this app, we have a `<Hoverable>` component that tracks whether the mouse is currently over it. It needs to pass that data *back* to the parent component, so that we can update the slotted contents.

For this, we use *slot props*. In `Hoverable.svelte`, pass the `hovering` value into the slot:

```svelte
<div on:mouseenter={enter} on:mouseleave={leave}>
  <slot hovering={hovering}>
</div>
```

> Remember you can also use `<slot {hovering}>` shorthand, if you prefer.

Then, to expose `hovering` to the contents of the `<Hoverable>` component, we use the `let` directive:

```svelte
<Hoverable let:hovering={hovering}>
  <div class:active={hovering}>
    {#if hovering}
      <p>I am begin hovered upon.</p>
    {:else}
      <p>Hover over me!</p>
    {/if}
  </div>
</Hoverable>
```

You can rename the variable, if you want - let's call it `active` in the parent component:

```svelte
<Hoverable let:hovering={active}>
  <div class:active>
    {#if active}
      <p>I am being hovered upon.</p>
    {:else}
      <p>Hover over me!</p>
    {/if}
  </div>
</Hoverable>
```

You can use as many of these components as you like, and the slotted props will remain local to the component where they're declared.

> Named slots can also have props; use the `let` directive on an element with a `slot="..."` attribute, instead of on the component itself.

**Hoverable.svelte**
```svelte
<script>
	let hovering;

	function enter() {
		hovering = true;
	}

	function leave() {
		hovering = false;
	}
</script>

<div on:mouseenter={enter} on:mouseleave={leave}>
	<slot {hovering}></slot>
</div>
```

**App.svelte**
```svelte
<script>
	import Hoverable from './Hoverable.svelte';
</script>

<style>
	div {
		padding: 1em;
		margin: 0 0 1em 0;
		background-color: #eee;
	}

	.active {
		background-color: #ff3e00;
		color: white;
	}
</style>

<Hoverable let:hovering={active}>
	<div class:active>
		{#if active}
			<p>I am being hovered upon.</p>
		{:else}
			<p>Hover over me!</p>
		{/if}
	</div>
</Hoverable>
```