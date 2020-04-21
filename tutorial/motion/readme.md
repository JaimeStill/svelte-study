# Motion

**Topics**  
* [Introduction](../readme.md)
* [Reactivity](../reactivity/readme.md)
* [Props](../props/readme.md)
* [Logic](../logic/readme.md)
* [Events](../events/readme.md)
* [Bindings](../bindings/readme.md)
* [Lifecycle](../lifecycle/readme.md)
* [Stores](../stores/readme.md)
* [Transitions](../transitions/readme.md)
* [Animations](../animations/readme.md)

**Contents**  
* [Tweened](#tweened)
* [Spring](#spring)

## Tweened
[Back to Top](#motion)

Setting values and watching the DOM update automatically is cool. Know what's even cooler? *Tweening* thos values. Svelte includes tools to help you build slick user interfaces that use animation to communicate changes.

```svelte
<script>
  import { tweened } from 'svelte/motion';
  import { cubicOut } from 'svelte/easing';

  const progress = tweened(0, {
    duration: 400,
    easing: cubicOut
  });
</script>

<style>
  progress {
    display: block;
    width: 100%;
  }
</style>

<progress value={$progress}></progress>

<button on:click={() => progress.set(0)}>0%</button>
<button on:click={() => progress.set(0.25)}>25%</button>
<button on:click={() => progress.set(0.5)}>50%</button>
<button on:click={() => progress.set(0.75)}>75%</button>
<button on:click={() => progress.set(1)}>100%</button>
```

> The `svelte/easing` module contains the [Penner easing equations](https://web.archive.org/web/20190805215728/http://robertpenner.com/easing/), or you can supply your own `p => t` function where `p` and `t` are both values between 0 and 1.

The full set of options available to `tweened`:

* `delay` - milliseconds before the tween starts.
* `duration` - either the duration of the tween in milliseconds, or a `(from, to) => milliseconds` function allowing you to (e.g.) specify longer tweens for larger changes in value.
* `easing` - a `p => t` function.
* `interpolate` - a custom `(from, to) => t => value` function for interpolating between arbitrary values. By default, Svelte will interpolate between numbers, dates, and identically shaped arrays and objects (as long as they only contain numbers and dates or other value arrays and objects). If you want to interpolate (for example) colour strings or transformation matrices, supply a custom interpolator.

You can also pass these options to `progress.set` and `progress.update` as a second argument, in which case they will override the defaults. The `set` and `update` methods both return a promise that resolves when the tween completes.

## Spring
[Back to Top](#motion)

The `spring` function is an alternative to `tweened` that often works better for values that are frequently changing.

In this example we have two stores - one representing the circle's coordinates, and one representing its size.

Both springs have default `stiffness` and `damping` values, which control the spring's springiness.

```svelte
<script>
  import { spring } from 'svelte/motion';

  let coords = spring({ x: 50, y: 50 }, {
    stiffness: 0.1,
    damping: 0.25
  });

  let size = spring(10);
</script>

<style>
  svg { width: 100%; height: 100%; margin: -8px; }
  circle { filli: #ff3e00 }
</style>

<div style="position: absolute; right: 1em;">
  <label>
    <h3>stiffness ({coords.stiffness})</h3>
    <input bind:value={coords.stiffness} type="range" min="0" max="1" step="0.01">
  </label>
  <label>
    <h3>damping ({coords.damping})</h3>
    <input bind:value={coords.damping} type="range" min="0" max="1" step="0.01">
  </label>
</div>

<svg on:mousemove={e => coords.set({ x: clientX, y: e.clientY })}
     on:mousedown={() => size.set(30)}
     on:mouseup={() => size.set(10)}>
  <circle cx={$coords.x}
          cy={$coords.y}
          r={$size} />
</svg>
```