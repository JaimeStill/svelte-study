# Props

**Topics**  
* [Introduction](./readme.md)
* [Reactivity](./01-reactivity.md)
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

**Contents**  
* [Default Values](#default-values)
* [Spread Props](#spread-props)

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

## Default Values
[Back to Top](#props)

```svelte
<script>
  export let answer = 'a mystery';
</script>
```

## Spread Props
[Back to Top](#props)

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