# Svelte Tutorial Notes

* [Introduction](#introduction)
    * [Adding Data](#adding-data)
    * [Dynamic Attributes](#dynamic-attributes)
    * [Styling](#styling)
    * [Nested Components](#nested-components)
    * [HTML Tags](#html-tags)
* [Reactivity](#reactivity)
    * [Assignments](#assignments)
    * [Declarations](#declarations)
    * [Statements](#statements)
    * [Updating Arrays and Objects](#updating-arrays-and-objects)
* [Props](#props)
    * [Default Values](#default-values)
    * [Spread Props](#spread-props)
* [Logic](#logic)
    * [if blocks](#if-blocks)
    * [else blocks](#else-blocks)
    * [else-if blocks](#else-if-blocks)
    * [each blocks](#each-blocks)
    * [Keyed each blocks](#keyed-each-blocks)
    * [await blocks](#await-blocks)
* [Events](#events)
    * [Inline Event Handlers](#inline-event-handlers)
    * [Event Modifiers](#event-modifiers)
    * [Component Events](#component-events)
    * [Event Forwarding](#event-forwarding)
    * [DOM Event Forwarding](#dom-event-forwarding)

## Introduction
[Back to Top](#svelte-tutorial-notes)

### Adding Data
[Back to Top](#svelte-tutorial-notes)

```svelte
<script>
    let name = 'world';
</script>

<h1>Hello {name}!</h1>
```

### Dynamic Attributes
[Back to Top](#svelte-tutorial-notes)

```svelte
<script>
    let src = 'tutorial/image.gif';
</script>

<img src={src} />
```

> or (if the property and attribute names are the same)

```svelte
<img {src} />
```

### Styling
[Back to Top](#svelte-tutorial-notes)

```svelte
<style>
    p {
        color: purple;
        font-family: 'Comic Sans MS', cursive;
        font-size: 2em;
    }
</style>

<p>This is a paragraph.</p>
```

### Nested Components
[Back to Top](#svelte-tutorial-notes)

Styles do not leak between components.

```svelte
<script>
    import Nested from './Nested.svelte';
</script>

<style>
    p {
        color: purple;
        font-family: 'Comic Sans MS', cursive;
        font-size: 2em;
    }
</style>

<p>This is a paragraph.</p>
<Nested />
```

[![nested-components](./images/nested-components.png)](./images/nested-components.png)

### HTML Tags
[Back to Top](#svelte-tutorial-notes)

> Svelte doesn't perform sanitization before it inserts into the DOM. Manually escape HTML originating from sources you don't trust, otherwise you risk exposing users to XSS attacks.

```svelte
<script>
    let string = `this string contains some <strong>HTML!!!</strong>`;
</script>

<p>{@html string}</p>
```

## Reactivity
[Back to Top](#svelte-tutorial-notes)

Keep the DOM in sync with your application state using *reactivity*.

### Assignments
[Back to Top](#svelte-tutorial-notes)

```svelte
<script>
    let count = 0;

    function handleClick() {
        count += 1;
    }
</script>

<button>
    Clicked {count} { count === 1 ? 'time' : 'times' }
</button>
```

### Declarations
[Back to Top](#svelte-tutorial-notes)

Often, some parts of a component's state need to be computed from *other* parts, and recomputed whenever they change. THese are called *reactive declarations*.

```svelte
<script>
    let count = 0;
    $: doubled = count * 2;

    function handleClick() {
        count += 1;
    }
</script>

<button on:click={handleClick}>
    Clicked {count} { count === 1 ? 'time' : 'times' }
</button>
<p>{count} doubled is {doubled}</p>
```

### Statements
[Back to Top](#svelte-tutorial-notes)


```svelte
$: console.log(`the count is ${count}`);
```

Statements can be grouped with a block:

```svelte
$: {
    console.log(`the count is ${count}`);
    alert(`I SAID THE COUNT IS ${count}`);
}
```

You can put the `$:` in front of things like `if` blocks:

```svelte
$: if (count >= 10) {
    alert(`count is dangerously high!`);
    count = 9;
}
```

### Updating Arrays and Objects
[Back to Top](#svelte-tutorial-notes)

Because Svelte's reactivity is triggered by assignments, using array methods like `push` and `splice` won't automatically cause updates.

```svelte
function addNumber() {
    numbers.push(numbers.length + 1);
    numbers = numbers; // redundant assignment to trigger update
}
```

A better solution

```svelte
function addNumber() {
    numbers = [...numbers, numbers.length + 1];
}
```

> Similar patterns can be used to replace `pop`, `shift`, `unshift`, and `splice`.

A simple rule: The name of the updated variable must appear on the left hand side of the assignment.

```svelte
const foo = obj.foo;
foo.bar = 'baz';

obj = obj; // necessary to reflect updated foo
```

## Props
[Back to Top](#svelte-tutorial-notes)

In any real application, you'll need to pass data from one component down to its children. To do that, declare *properties*. This is done with the `export` keyword.

**Nested.svelte**  
```svelte
<script>
    export let answer;
</script>

<p>The answer is {answer}</p>
```

**App.svelte**  
```svelte
<script>
    import Nested from './Nested.svelte';
</script>

<Nested answer={42}>
```

### Default Values
[Back to Top](#svelte-tutorial-notes)

```svelte
<script>
    export let answer = 'a mystery';
</script>
```

### Spread Props
[Back to Top](#svelte-tutorial-notes)

If you have an object of properties, you can 'spread' them onto a component instead of specifying each one:

**Info.svelte**  
```svelte
<script>
    export let name;
    export let version;
    export let speed;
    export let website;
</script>

<p>
    The <code>{name}</code> package is {speed} fast.
    Download version {version} from <a href="https://www.npmjs.com/package/{name}">npm</a> and <a href={website}>learn more here</a>
</p>
```

## Logic
[Back to Top](#svelte-tutorial-notes)

HTML doesn't have a way to express *logic*, like conditionals and loops. Svelte does.

### if blocks
[Back to Top](#svelte-tutorial-notes)

```svelte
{#if user.loggedIn}
    <button on:click={toggle}>
        Log out
    </button>
{/if}

{#if !user.loggedIn}
    <button on:click={toggle}>
        Log in
    </button>
{/if}
```

### else Blocks
[Back to Top](#svelte-tutorial-notes)

```svelte
{#if user.loggedIn}
    <button on:click={toggle}>
        Log out
    </button>
{:else}
    <button on:click={toggle}>
        Log in
    </button>
{/if}
```

### else-if blocks
[Back to Top](#svelte-tutorial-notes)

```svelte
{#if x > 10}
    <p>{x} is greater than 10
{:else if 5 > x}
    <p>{x} is less than 5</p>
{:else}
    <p>{x} is between 5 and 10</p>
{/if}
```

### each blocks
[Back to Top](#svelte-tutorial-notes)

If you need to loop over lists of data, use an `each` block.

```svelte
<ul>
    {#each cats as cat}
        <li><a target="_blank"
               href="https://www.youtube.com/watch?v={cat.id}">
            {cat.name}
        </a></li>
    {/each}
</ul>
```

You can also get the current *index* as a second argument:

```svelte
{#each cats as cat, i}
    <li><a>
        {i + 1}: {cat.name}
    </li></a>
{/each}
```

### Keyed each blocks
[Back to Top](#svelte-tutorial-notes)

By default, when you modify the value of an `each` block, it will add and remove items at the *end* of the block, and update any values that have changed. That might not be what you want.

```svelte
{#each things as thing (thing.id)}
    <Thing current={thing.color} />
{/each}
```

The `(thing.id)` tells Svelte how to figure out what changed.

### await blocks
[Back to Top](#svelte-tutorial-notes)

```svelte
{#await promise}
    <p>...waiting</p>
{:then number}
    <p>The number is {number}</p>
{:catch error}
    <p style="color: red">{error.message}</p>
{/await}
```

If you know that your promise can't reject, you can omit the `catch` block. You can also omit the first block if you don't want to show anything until the promise resolves:

```svelte
{#await promise then value}
    <p>The value is {value}</p>
{/await}
```

## Events
[Back to Top](#svelte-tutorial-notes)

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

### Inline Event Handlers
[Back to Top](#svelte-tutorial-notes)

```svelte
<div on:mousemove={e => m = { x: e.clientX, y: e.clientY }}>
    The mouse position is {m.x} x {m.y}
</div>
```

### Event Modifiers
[Back to Top](#svelte-tutorial-notes)

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

### Component Events
[Back to Top](#svelte-tutorial-notes)

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

### Event Forwarding
[Back to Top](#svelte-tutorial-notes)

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

### DOM Event Forwarding
[Back to Top](#svelte-tutorial-notes)

Event fowarding works for DOM events too.

We want to get notified on clicks on our `<CustomButton>` - to do that, we just need to forward `click` events on the `<button>` element in `CustomButton.svelte`:

```svelte
<button on:click>
    Click me
</button>
```