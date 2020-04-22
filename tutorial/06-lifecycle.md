# Lifecycle

**Topics**  
* [Introduction](./readme.md)
* [Reactivity](./01-reactivity.md)
* [Props](./02-props.md)
* [Logic](./03-logic.md)
* [Events](./04-events.md)
* [Bindings](./05-bindings.md)
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

**Contents**  
* [onMount](#onmount)
* [onDestroy](#ondestroy)
* [beforeUpdate and afterUpdate](#beforeupdate-and-afterupdate)
* [tick](#tick)

Every component has a *lifecycle* that starts when it is created, and ends when it is destroyed. There are a handful of functions that allow you to run code at key moments during that lifecycle.

## onMount
[Back to Top](#lifecycle)

```svelte
<script>
  import { onMount } from 'svelte';

  let photos = [];

  onMount(async () => {
    const res = await fetch(`https://jsonplaceholder.typicode.com/photos?_limit=20`);
    photos = await res.json();
  });
</script>
```

> If the `onMount` callback returns a function, that function will be called when the component is destroyed.

## onDestroy
[Back to Top](#lifecycle)

To run code when your component is destroyed, use `onDestroy`:

```svelte
<script>
  import { onDestroy } from 'svelte';

  let seconds = 0;
  const interval = setInterval(() => seconds += 1, 1000);

  onDestroy(() => clearInterval(interval));
</script>
```

While it's important to call lifecycle functions during the component's initialization, it doesn't matter *where* you call them from. We could abstract the interval logic into a helper function in `utils.js`:

```js
import { onDestroy } from 'svelte';

export function onInterval(callback, milliseconds) {
  const interval = setInterval(callback, milliseconds);
  onDestroy(() => clearInterval(interval));
}
```

and import it into our component:

```svelte
<script>
  import { onInterval } from './utils.js';

  let seconds = 0;
  onInterval(() => seconds += 1, 1000);
</script>
```

## beforeUpdate and afterUpdate
[Back to Top](#lifecycle)

The `beforeUpdate` function schedules work to happen immediately before the DOM has been updated. `afterUpdate` is its counterpart, used for running code once the DOM is in sync with your data.

Together, they're useful for doing things imperatively that are difficult to achieve in a purely state-driven way, like updating the scroll position of an element.

```svelte
import { beforeUpdate, afterUpdate } from 'svelte';

let div;
let autoscroll;

beforeUpdate(() => {
  autoscroll = div && (div.offsetHeight + div.scrollTop) > (div.scrollHeight - 20);
});

afterUpdate(() => {
  if (autoscroll) div.scrollTo(0, div.scrollHeight);
});
```

## tick
[Back to Top](#lifecycle)

The `tick` function is unlike other lifecycle functions in that you can call it any time, not just when the component first initializes. It returns a promise that resolves as soon as any pending state changes have been applied to the DOM (or immediately, if there are no pending state changes).

Wheny ou invalidate component state in Svelte, it doesn't update the DOM immediately. Instead, it waits until the next *microtask* to see if there are any other changes that need to be applied, including in other components. Doing so avoids unnecessary work and allows the browser to batch things more effectively.

You can see that behavior in this example. Select a range of text and hit the tab key. Because the `<textarea>` value changes, the current selection is cleared and the cursor jumps, annoyingly, to the ned. We can fix this by importing `tick` and running it immediately before we set `this.selctionStart` and `this.selectionEnd` at the end of `handleKeydown`:

```svelte
<script>
  import { tick } from 'svelte';

  let text = `Select some text and hit the tab key to toggle uppercase`;

  async function handleKeydown(event) {
    if (event.which !== 9) return;

    const { selectionStart, selectionEnd, value } = this;
    const selection = value.slice(selectionStart, selectionEnd);

    const replacement = /[a-z]/.test(selection)
      ? selection.toUpperCase()
      : selection.toLowerCase();

    text = (
      value.slice(0, selectionStart) +
      replacement +
      value.slice(selectionEnd)
    );

    await tick();
    this.selectionStart = selectionStart;
    this.selectionEnd = selectionEnd;
  }
</script>

<style>
  textarea {
    width: 100%;
    height: 200px;
  }
</style>

<textarea value={text}
          on:keydown|preventDefault={handleKeydown}></textarea>
```