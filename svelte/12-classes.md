# Classes

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
* [Component Composition](./13-component-composition.md)
* [Context API](./14-context-api.md)
* [Special Elements](./15-special-elements.md)
* [Module Context](./16-module-context.md)
* [Debugging](./17-debugging.md)
* [Congratulations](./18-congratulations)

**Contents**  
* [The class Directive](#the-class-directive)
* [Shorthand class Directive](#shorthand-class-directive)

## The class Directive
[Back to Top](#classes)

Like any other attribute, you can specify classes with a JavaScript attribute:

```svelte
<button class={current === 'foo' ? 'active' : ''}
        on:click={() => current = 'foo'}>
  foo
</button>
```

This is such a common pattern in UI development that Svelte includes a special directive to simplify it:

```svelte
<button class:active={current === 'foo'}
        on:click={() => current = 'foo'}>
  foo
</button>
```

The `active` class is added to the element whenever the value of the expression is truthy, and removed when it's falsy.

```svelte
<script>
	let current = 'foo';
</script>

<style>
	button {
		display: block;
	}

	.active {
		background-color: #ff3e00;
		color: white;
	}
</style>

<button class:active="{current === 'foo'}"
				on:click="{() => current = 'foo'}">
	foo
</button>

<button class:active="{current === 'bar'}"
				on:click="{() => current = 'bar'}">
	bar
</button>

<button class:active="{current === 'baz'}"
				on:click="{() => current = 'baz'}">
	baz
</button>
```

## Shorthand class Directive
[Back to Top](#classes)

Often, the name of the class will be the same as the name of the value it depends on:

```svelte
<div class:big={big}></div>
```

In those cases we can use a shorthand form:

```svelte
<div class:big></div>
```

```svelte
<script>
	let big = false;
</script>

<style>
	.big {
		font-size: 4em;
	}
</style>

<label>
	<input type=checkbox bind:checked={big}>
	big
</label>

<div class:big>
	some {big ? 'big' : 'small'} text
</div>
```