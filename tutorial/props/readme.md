# Props

**Topics**  
* [Introduction](../readme.md)
* [Reactivity](../reactivity/readme.md)
* [Logic](../logic/readme.md)
* [Events](../events/readme.md)
* [Bindings](../bindings/readme.md)
* [Lifecycle](../lifecycle/readme.md)
* [Stores](../stores/readme.md)
* [Motion](../motion/readme.md)
* [Transitions](../transitions/readme.md)

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