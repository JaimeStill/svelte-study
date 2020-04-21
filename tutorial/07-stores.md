# Stores

**Topics**  
* [Introduction](./readme.md)
* [Reactivity](./01-reactivity.md)
* [Props](./02-props.md)
* [Logic](./03-logic.md)
* [Events](./04-events.md)
* [Bindings](./05-bindings.md)
* [Lifecycle](./06-lifecycle.md)
* [Motion](./08-motion.md)
* [Transitions](./09-transitions.md)
* [Animations](./10-animations.md)
* [Actions](./11-actions.md)
* [Classes](./12-classes.md)

**Contents**  
* [Writable Stores](#writable-stores)
* [Auto-Subscriptions](#auto-subscriptions)
* [Readable Stores](#readable-stores)
* [Derived Stores](#derived-stores)
* [Custom Stores](#custom-stores)
* [Store Bindings](#store-bindings)

Not all application state belongs inside your application's component hierarchy. Sometimes you'll have values that need to be accessed by multiple unrelated components, or by a regular JavaScript module.

In Svelte, we do this with *stores*. A store is simply an object with a `subscribe` method that allows interested parties to be notified whenever the store value changes.

## Writable Stores
[Back to Top](#stores)

In `App.svelte`, `count` is a store, and we're setting `count_value` in the `count.subscribe` callback.

In `stores.js`, `count` is a *writable* store, which means it has `set` and `update` methods in addition to `subscribe`.

`Incrementer.svelte` increments the value of `count`, `Decrementer.svelte` decrements the value of `count`, and `Resetter.svelte` resets the value to **0**.

**stores.js**  
```js
import { writable } from 'svelte/store';

export const count = writable(0);
```

**App.svelte**  
```svelte
<script>
  import { count } from './stores.js';
  import Incrementer from './Incrementer.svelte';
  import Decrementer from './Decrementer.svelte';
  import Resetter from './Resetter.svelte';

  let count_value;

  const unsubscribe = count.subscribe(value => {
    count_value = value;
  });
</script>

<h1>The count is {count_value}</h1>

<Incrementer />
<Decrementer />
<Resetter />
```

**Incrementer.svelte**  
```svelte
<script>
  import { count } from './stores.js';

  function increment() {
    count.update(n => ++n);
  }
</script>

<button on:click={increment}>
  +
</button>
```

**Decrementer.svelte**  
```svelte
<script>
  import { count } from './stores.js';

  function decrement() {
    count.update(n => --n);
  }
</script>

<button on:click={decrement}>
  -
</button>
```

**Resetter.svelte**  
```svelte
<script>
  import { count } from './stores.js';

  function reset() {
    count.set(0);
  }
</script>

<button on:click={reset}>
  reset
</button>
```

## Auto-Subscriptions
[Back to Top](#stores)

The app in the previous section works, but there's a subtle bug - the `unsubscribe` function never gets called. If the component was instantiated and destroyed many times, this would result in a *memory leak*.

One way to fix it would be to use the `onDestroy` lifecycle hook:

**App.svelte**  
```svelte
<script>
  import { onDestroy } from 'svelte';
  import { count } from './stores.js';
  import Incrementer from './Incrementer.svelte';
  import Decrementer from './Decrementer.svelte';
  import Resetter from './Resetter.svelte';

  let count_value;

  const unsubscribe = count.subscribe(value => {
    count_value = value;
  });

  onDestroy(unsubscribe);
</script>

<h1>The count is {count_value}</h1>
```

It starts to get a bit boilerplatey though, especially if your component subscribes to multiple stores. Instead, Svelte has a trick up its sleeve - you can reference a store value by prefixing the store name with `$`:

```svelte
<script>
  import { count } from './stores.js';
  import Incrementer from './Incrementer.svelte';
  import Decrementer from './Decrementer.svelte';
  import Resetter from './Resetter.svelte';
</script>

<h1>The count is {$count}</h1>
```

> Auto-subscription only works with store variables that are declared (or imported) at the top-level scope of a component.

You're not limited to using `$count` inside the markup, either - you can use it anywhere in the `<script>` as well, such as in event handlers or reactive declarations.

> Any name beginning with `$` is assumed to refer to a store value. It's effectively a reserved character - Svelte will prevent you from declaring your own variables with a `$` prefix.

## Readable Stores
[Back to Top](#stores)

Not all stores should be writable by whoever has a reference to them. For example, you might have a store representing the mouse position or the user's geolocation, and it doesn't make sense to be able to set those values from 'outside'. For those cases, we have *readable* stores.

The first argument to `readable` is an initial value, which can be `null` or `undefined` if you don't have one yet. The second argument is a `start` function that takes a `set` callback and returns a `stop` function. The `start` function is called when the store gets its first subscriber; `stop` is called when the last subscriber unsubscribes.

**stores.js**  
```js
import { readable } from 'svelte/store';

export const time = readable(null, function start(set) {
  const interval = setInterval(() => {
    set(new Date());
  }, 1000);

  return function stop() {
    clearInterval(interval);
  };
});
```

**App.svelte**  
```svelte
<script>
  import { time } from './stores.js';

  const formatter = new Intl.DateTimeFormat('en', {
    hour12: true,
    hour: 'numeric',
    minute: '2-digit',
    second: '2-digit'
  });
</script>

<h1>The time is {formatter.format($time)}</h1>
```

## Derived Stores
[Back to Top](#stores)

You can create a store whose value is based on teh value of one or more *other* stores with `derived`. Building on the previous example, we can create a store that derives how the duration has been open.

**stores.js**  
```js
import {
  readable,
  derived
} from 'svelte/store';

export const time = readable(new Date(), function start(set) {
  const interval = setInterval(() => {
    set(new Date());
  }, 1000);

  return function stop() {
    clearInterval(interval);
  };
});

const start = new Date();

export const elapsed = derived(
  time,
  $time => Math.round(($time - start) / 1000);
);
```

**App.svelte**  
```svelte
<script>
  import {
    time,
    elapsed
  } from './stores.js';

  const formatter = new Intl.DateTimeFormat('en', {
    hour12: true,
    hour: 'numeric',
    minute: '2-digit',
    second: '2-digit'
  });
</script>

<h1>The time is {formatter.format($time)}</h1>

<p>
  This page has been open fro
  {$elapsed} {$elapsed === 1 ? 'second' : 'seconds'}
</p>
```

> It's possible to derive a store from multiple inputs, and to explicitly `set` a value instead of returning it (which is useful for deriving values asynchronously). Consult the [API reference](https://svelte.dev/docs#derived) for more information.

## Custom Stores
[Back to Top](#stores)

As long as an object correctly implements the `subscribe` method, it's a store. Beyond that, anything goes. It's very easy, therefore, to create custom stores with domain-specific logic.

The `count` store from the earlier exmaple could include `increment`, `decrement`, and `reset` methods and avoid exposing `set` and `update`.

**stores.js**  
```js
import { writable } from 'svelte/store';

function createCount() {
  const { subscribe, set, update } = writable(0);

  return {
    subscribe,
    increment: () => update(n => ++n),
    decrement: () => update(n => --n),
    reset: () => set(0)
  };
}

export const count = createCount();
```

**App.svelte**  
```svelte
<script>
  import { count } from './stores.js';
</script>

<h1>The count is {$count}</h1>

<button on:click={count.increment}>+</button>
<button on:click={count.decrement}>-</button>
<button on:click={count.reset}>reset</button>
```

## Store Bindings
[Back to Top](#stores)

If a store is writable - i.e. it has a `set` method - you can bind to its value, just as you can bind to local component state.

In this example we have a writable store `name` and a derived store `greeting`, and an input that binds to `$name`. Changing the input value will update `name` and all its dependents.

We can also assign directly to store values inside a component. A button with a click event of: `on:click={() => $name += '!'}`. The `$name += '!'` assignment is equivalent to `name.set($name + '!')`.

**stores.js**  
```js
import {
  writable,
  derived
} from 'svelte/store';

export const name = writable('world');

export const greeting = derived(
  name,
  $name => `Hello ${$name}!`
);
```

**App.svelte**  
```svelte
<script>
  import {
    name,
    greeting
  } from './stores.js'
</script>

<h1>{$greeting}</h1>
<input bind:value={$name}>
<button on:click={() => $name += '!'}>
  Add exclamation mark!
</button>
```