# Logic

**Topics**  
* [Introdution](./readme.md)
* [Reactivity](./01-reactivity.md)
* [Props](./02-props.md)
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
* [Debugging](./17-debugging.md)
* [Congratulations](./18-congratulations)

**Contents**  
* [if blocks](#if-blocks)
* [else blocks](#else-blocks)
* [else-if blocks](#else-if-blocks)
* [each blocks](#each-blocks)
* [Keyed each blocks](#keyed-each-blocks)
* [await blocks](#await-blocks)

HTML doesn't have a way to express *logic*, like conditionals and loops. Svelte does.

## if blocks
[Back to Top](#logic)

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

## else blocks
[Back to Top](#logic)

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

## else-if blocks
[Back to Top](#logic)

```svelte
{#if x > 10}
  <p>{x} is greater than 10
{:else if 5 > x}
  <p>{x} is less than 5</p>
{:else}
  <p>{x} is between 5 and 10</p>
{/if}
```

## each blocks
[Back to Top](#logic)

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

## Keyed each blocks
[Back to Top](#logic)

By default, when you modify the value of an `each` block, it will add and remove items at the *end* of the block, and update any values that have changed. That might not be what you want.

```svelte
{#each things as thing (thing.id)}
  <Thing current={thing.color} />
{/each}
```

The `(thing.id)` tells Svelte how to figure out what changed.

## await blocks
[Back to Top](#logic)

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