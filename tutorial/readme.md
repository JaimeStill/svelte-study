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

### Inline event handlers

```svelte
<div on:mousemove={e => m = { x: e.clientX, y: e.clientY }}>
    The mouse position is {m.x} x {m.y}
</div>
```