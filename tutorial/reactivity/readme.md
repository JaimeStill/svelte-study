# Reactivity

**Topics**  
* [Introduction](../readme.md)
* [Props](../props/readme.md)
* [Logic](../logic/readme.md)
* [Events](../events/readme.md)
* [Bindings](../bindings/readme.md)
* [Lifecycle](../lifecycle/readme.md)
* [Stores](../stores/readme.md)
* [Motion](../motion/readme.md)

**Contents**  
* [Assignments](#assignments)
* [Declarations](#declarations)
* [Statements](#statements)
* [Updating Arrays and Objects](#updating-arrays-and-objects)

Keep the DOM in sync with your application state using *reactivity*.

## Assignments
[Back to Top](#reactivity)

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

## Declarations
[Back to Top](#reactivity)

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

## Statements
[Back to Top](#reactivity)


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

## Updating Arrays and Objects
[Back to Top](#reactivity)

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