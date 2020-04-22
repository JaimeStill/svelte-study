# Debugging

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

Occasionally, it's useful to inspect a piece of data as it flows through your app.

One approach is to use `console.log(...)` inside your markup. If you want to pause execution, though, you can use the `{@debug ...}` tag with a comma-separated list of values you want to inspect:

```svelte
{@debug user}

<h1>Hello {user.firstname}</h1>
```

If you now open your devtools and start interacting with the `<input>` elements, you'll trigger the debugger as the value of `user` changes.

**App.svelte**
```svelte
<script>
	let user = {
		firstname: 'Ada',
		lastname: 'Lovelace'
	};
</script>

<input bind:value={user.firstname}>
<input bind:value={user.lastname}>

{@debug user}

<h1>Hello {user.firstname}!</h1>
```