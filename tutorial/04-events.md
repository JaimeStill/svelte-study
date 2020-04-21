# Events

**Topics**  
* [Introduction](./readme.md)
* [Reactivity](./01-reactivity.md)
* [Props](./02-props.md)
* [Logic](./03-logic.md)
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
* [Inline Event Handlers](#inline-event-handlers)
* [Event Modifiers](#event-modifiers)
* [Component Events](#component-events)
* [Event Forwarding](#event-forwarding)
* [DOM Event Forwarding](#dom-event-forwarding)

You can listen to any event on an element with the `on:` directive:

```svelte
<script>
  let m = { x: 0, y: 0 };

  function handleMousemove(event) {
    m.x = event.clientX;
    m.y = event.clientY;
  }
</script>

<style>
  div { width: 100%; height: 100%; }
</style>

<div on:mousemove={handleMousemove}>
  The mouse position is {m.x} x {m.y}
</div>
```

## Inline Event Handlers
[Back to Top](#events)

```svelte
<div on:mousemove={e => m = { x: e.clientX, y: e.clientY }}>
  The mouse position is {m.x} x {m.y}
</div>
```

## Event Modifiers
[Back to Top](#events)

DOM event handlers can have *modifiers* that alter their behavior:

* `preventDefault` - calls `event.preventDefault()` before running the handler. Useful for client-side form handling, for example.
* `stopPropogation` - calls `event.stopPropogation()`, preventhing the event from reaching the next element.
* `passive` - improves scrolling performance on touch / wheel events (Svelte will add it automatically where it's safe to do so).
* `capture` - fires the handler during the *capture* phase instead of the *bubbling* phase ([MDN docs](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Events#Event_bubbling_and_capture))
* `once` - remove the handler after the first time it runs
* `self` - only trigger handler if `event.target` is the element itself

You can chain modifiers together: `on:click|once|capture={...}`

```svelte
<script>
  function handleClick() {
    alert('clicked')
  }
</script>

<button on:click|once={handleClick}>
  Click me
</button>
```

## Component Events
[Back to Top](#events)

Components can also dispatch events, and must do so by creating an event dispatcher.

**Inner.svelte**  
```svelte
<script>
  import { createEventDispatcher } from 'svelte';

  const dispatch = createEventDispatcher();

  function sayHello() {
    dispatch('message', {
      text: 'Hello!'
    });
  }
</script>

<button on:click{sayHello}>
    Click to say hello
</button>
```

**App.svelte**  
```svelte
<script>
  import Inner from './Inner.svelte';

  function handleMessage(event) {
    alert(event.detail.text);
  }
</script>

<Inner on:message={handleMessage} />
```

> `createEventDispatcher` must be called when the component is first instantiated - you can't do it later inside e.g. a `setTimeout` callback. This links `dispatch` to the component instance.

## Event Forwarding
[Back to Top](#events)

Unlike DOM events, component events don't *bubble*. If you want to listen to an event on some deeply nested component, the intermediate components must *forward* the event.

In this case, we have the same `App.svelte` and `Inner.svelte` as in the previous example, but there's now an `Outer.svelte` component that contains `<Inner />`.

One way we could solve the problem is adding `createEventDispatcher` to `Outer.svelte`, listening for the `message` event, and creating a handler for it:

```svelte
<script>
  import Inner from './Inner.svelte';
  import { createEventDispatcher } from 'svelte';

  const dispatch = createEventDispatcher();

  function forward(event) {
    dispatch('message', event.detail);
  }
</script>

<Inner on:message={forward}/>
```

That's a lot of code to write, so Svelte gives an equivalent shorthand - an `on:message` event directive without a value means 'forward all `message` events':

```svelte
<script>
  import Inner from './Inner.svelte';
</script>

<Inner on:message/>
```

## DOM Event Forwarding
[Back to Top](#events)

Event fowarding works for DOM events too.

We want to get notified on clicks on our `<CustomButton>` - to do that, we just need to forward `click` events on the `<button>` element in `CustomButton.svelte`:

```svelte
<button on:click>
  Click me
</button>
```