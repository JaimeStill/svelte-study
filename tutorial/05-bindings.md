# Bindings

**Topics**
* [Introduction](./readme.md)
* [Reactivity](./01-reactivity.md)
* [Props](./02-props.md)
* [Logic](./03-logic.md)
* [Events](./04-events.md)
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
* [Text Inputs](#text-inputs)
* [Numeric Inputs](#numeric-inputs)
* [Checkbox Inputs](#checkbox-inputs)
* [Group Inputs](#group-inputs)
* [Textarea Inputs](#textarea-inputs)
* [Select Bindings](#select-bindings)
* [Select Multiple](#select-multiple)
* [Contenteditable Bindings](#contenteditable-bindings)
* [Each Block Bindings](#each-block-bindings)
* [Media Elements](#media-elements)
* [Dimensions](#dimensions)
* [This](#this)
* [Component Bindings](#component-bindings)

As a general rule, data flow in Svelte is *top down* - a parent component can set props on a child component, and a component can set attributes on an element, but not the other way around.

## Text Inputs
[Back to Top](#bindings)

Sometimes, it's useful to break that rule. Take the case of the `<input>` element. We *could* add an `on:input` event handler that sets the value of `name` to `event.target.value`, but it gets ugly. Instead, we can use the `bind:value` directive:

```svelte
<input bind:value={name}>
```

This means that not only will changes to the value of `name` update the input value, but changes to the input value will update `name`.

## Numeric Inputs
[Back to Top](#bindings)

In the DOM, everything is a string. When dealing with numeric inputs, you have to remember to coerce `input.value` before using it. With `bind:value`, Svelte takes care of it for you:

```svelte
<input type=number bind:value={a} min=0 max=10>
<input type=range bind:value={a} min=0 max=10>
```

## Checkbox Inputs
[Back to Top](#bindings)

Checkboxes are used for toggling between states. Instead of binding to `input.value`, we bind to `input.checked`:

```svelte
<input type=checkbox bind:checked={yes}>
```

## Group Inputs
[Back to Top](#bindings)

If you have multiple inputs relating to the same value, you can use `bind:group` along with teh `value` attribute.

Radio inputs int he same group are mutually exclusive; checkbox inputs in the same group form an array of selected values.

Add `bind:group` to each input:

```svelte
<input type=radio bind:group={scoops} value={1}>
```

Checkbox inputs can be rendered inside of an `each` block:

```svelte
<script>
  let flavours = ['Mint choc chip'];
  let menu = [
    'Cookies & cream',
    'Mint choc chip',
    'Prailine pecan'
  ];
</script>

<h2>Flavours</h2>

{#each menu as flavour}
  <label>
    <input type=checkbox bind:group={flavours} value={flavour}>
    {flavour}
  </label>
{/each}
```

## Textarea Inputs
[Back to Top](#bindings)

The `<textarea>` element behaves similarly to a text input in Svelte - use `bind:value`:

```svelte
<textarea bind:value={value}></textarea>
```

In cases like these, where the names match, we can also use a shorthand form:

```svelte
<textarea bind:value></textarea>
```

> This applies to all bindings, not just textareas.

## Select Bindings
[Back to Top](#bindings)

We can also use `bind:value` with `<select>` elements:

```svelte
<script>
  let questions = [
    { id: 1, text: `Where did you go to school?` },
    { id: 2, text: `What is your mother's name?` },
    { id: 3, text: `What is another personal fact that an attacker could easily find with Google?` }
  ];

  let selected;
</script>

<select bind:value={selected} on:change={() => answer = ''}>
  {#each questions as question}
    <option value={question}>
      {question.text}
    </option>
  {/each}
</select>
```

## Select Multiple
[Back to Top](#bindings)

A select can have a `multiple` attribute, in which case it will populate to an array rather than selecting a single value:

```svelte
<script>
  let flavours = ['Mint choc chip'];

  let menu = [
    'Cookies & cream',
    'Mint choc chip',
    'Prailine pecan'
  ];
</script>

<select multiple bind:value={flavors}>
  {#each menu as flavour}
    <option value={flavour}>
      {flavour}
    </option>
  {/each}
</select>
```

## Contenteditable Bindings
[Back to Top](#bindings)

Elements with a `contenteditable="true"` attribute support `textContext` and `innerHTML` bindings:

```
<script>
  let html = '<p>Write some text!</p>`;
</script>

<style>
  [contenteditable] {
    padding: 0.5em;
    border: 1px solid #eee;
    border-radius: 4px;
  }
</style>

<div contentedtiable="true"
     bind:innerHTML={html}></div>

<pre>{html}</pre>
```

## Each Block Bindings
[Back to Top](#bindings)

You can bind to properties inside an `each` block.

> Also notice how the `.done` class is conditionally set on the `<div>` inside of the `each` block using: `class:done={todo.done}`

```svelte
<script>
  let todos = [
    { done: false, text: 'finish Svelte tutorial' },
    { done: false, text: 'build an app' },
    { done: false, text: 'world domination' }
  ];

  function add() {
    todos = todos.concat({ done: false, text: '' });
  }

  function clear() {
    todos = todos.filter(t => !t.done);
  }

  $: remaining = todos.filter(t => !t.done).length;
</script>

<style>
  .done {
    opacity: 0.4;
  }
</style>

<h1>Todos</h1>

{#each todos as todo}
  <div class:done={todo.done}>
    <input type=checkbox
           bind:checked={todo.done}>
    <input placeholder="What needs to be done?"
           bind:value={todo.text}>
  </div>
{/each}

<p>{remaining} remaining</p>

<button on:click={add}>
  Add new
</button>

<button on:click={clear}>
  Clear completed
</button>
```

## Media Elements
[Back to Top](#bindings)

The `<audio>` and `<video>` elements have several properties that you can bind to.

Property | Return Value | Read Only?
---------|--------------|-----------
`duration` | the total duration of the video, in seconds. | yes
`buffered` | an array of `{start, end}` objects that indicates the ranges of the media source that the browser has buffered (if any) at the moment the `buffered` property is accessed. | yes
`seekable` | an array of `{start, end}` objects that the user is able to seek to, if any. | yes
`played` | an array of `{start, end}` objects that contains the ranges of the mdeia source that the browser has played, if any. | yes
`seeking` | boolean | yes
`ended` | boolean | yes
`videoHeight` | an unsigned integer value indicating the intrinsic height of the resource in CSS pixels, or 0 if no media is available yet. ***HTMLVideoElement only*** | yes
`videoWidth` | an unsigned integer value indicating the intrinsic width of the resource in CSS pixes, or 0 if no media is available yet. ***HTMLVideoElement only*** | yes
`currentTime` | the current point in the video, in seconds | no
`playbackRate` | how fast to play the video, where `1` is *normal* | no
`paused` | boolean | no
`volume` | a value between 0 and 1 | no

```svelte
<script>
  // These values are bound to properties of the video
  let time = 0;
  let duration;
  let paused = true;

  let showControls = true;
  let showControlsTimeout;

  function handleMousemove(e) {
    // Make the controls visible, but fade out after
    // 2.5 seconds of inactivity

    clearTimeout(showControlsTimeout);
    showControlsTimeout = setTimeout(() => showControls = false, 2500);
    showControls = true;

    if (!(e.buttons & 1)) return; // mouse not down
    if (!duration) return; // video not loaded yet;

    const { left, right } = this.getBoundingClientRect();
    time = duration * (e.clientX - left) / (right - left);
  }

  function handleMousedown(e) {
    // we can't rely on the built-in click event, because it fires
    // after a drag - we have to listen for clicks ourselves

    function handleMouseup() {
      if (paused) e.target.play();
      else e.target.pause();
      cancel();
    }

    function cancel() {
      e.target.removeEventListener('mouseup', handleMouseup);
    }

    e.target.addEventListener('mouseup', handleMouseup);

    setTimeout(cancel, 200);
  }

  function format(seconds) {
    if (isNaN(seconds)) return '...';

    const minutes = Math.floor(seconds / 60);
    seconds = Math.floor(seconds % 60);
    if (seconds < 10) seconds = `0${seconds}`;

    return `${minutes}:${seconds}`;
  }
</script>

<style>
  div {
    position: relative;
  }

  .controls {
    position: absolute;
    top: 0;
    width: 100%;
    transition: opacity 1s;
  }

  .info {
    display: flex;
    width: 100%;
    justify-content: space-between;
  }

  span {
    padding: 0.2em 0.5em;
    color: white;
    text-shadow: 0 0 8px black;
    font-size: 1.4em;
    opacity: 0.7;
  }

  .time {
    width: 3em;
  }

  .time:last-child { text-align: right }

  progress {
    display: block;
    width: 100%;
    height: 10px;
    -webkit-appearance: none;
    appearance: none;
  }

  progress::-webkit-progress-bar {
    background-color: rgba(0,0,0,0.2);
  }

  progress::-webkit-progress-value {
    background-color: rgba(255,255,255,0.6);
  }

  video {
    width: 100%;
  }
</style>

<h1>Caminandes: Llamigos</h1>
<p>From <a href="https://cloud.blender.org/open-projects">Blender Open Projects</a>. CC-BY</p>

<div>
  <video poster="https://sveltejs.github.io/assets/caminandes-llamigos.jpg"
         src="https://sveltejs.github.io/assets/caminandes-llamigos.mp4"
         on:mousemove={handleMousemove}
         on:mousedown={handleMousedown}
         bind:currentTime={time}
         bind:duration
         bind:paused></video>

  <div class="controls" style="opacity: {duration && showControls ? 1 : 0}">
    <progress value="{(time / duration) || 0}"/>
    <div class="info">
      <span class="time">{format(time)}</span>
      <span>click anywhere to {puased ? 'play' : 'pause'} / drag to seek</span>
      <span class="time">{format(duration)}</span>S
    </div>
  </div>
</div>
```

## Dimensions
[Back to Top](#bindings)

Every block-level element has `clientWidth`, `clientHeight`, `offsetWidth`, and `offsetHeight` bindings:

```svelte
<script>
  let w;
  let h;
  let size = 42;
  let text = 'edit me';
</script>

<style>
  input { display: block; }
  div { display: inline-block; }
  span { word-break: break-all; }
</style>

<input type=range bind:value={size}>
<input bind:value={text}>

<p>size: {w}px x {h}px</p>

<div bind:clientWidth={w}
     bind:clientHeight={h}>
  <span style="font-size: {size}px">{text}</span>
</div>
```

> These bindings are readonly - changing the values of `w` and `h` won't have any effect.

## This
[Back to Top](#bindings)

The readonly `this` binding applies to every element (and component) and allows you to obtain a reference to rendered elements. For example, we can get a reference to a `<canvas>` element:

```svelte
<canvas bind:this={canvas}
        width={32}
        height={32}></canvas>
```

> Note that the value of `canvas` will be `undefined` until the component has mounted, so we put the logic inside the `onMount` lifecycle function

```svelte
<script>
  import { onMount } from 'svelte';

  let canvas;

  onMount(() => {
    const ctx = canvas.getContext('2d');
    let frame;

    (function loop() {
      frame = requestAnimationFrame(loop);

      const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);

      for (let p = 0; p < imageData.length; p += 4) {
        const i = p / 4;
        const x = i % canvas.width;
        const y = i / canvas.height >>> 0;

        const t = window.performance.now();

        const r = 64 + (128 * x / canvas.width) + (64 * Math.sin(t / 1000));
        const g = 64 + (128 * y / canvas.height) + (64 * Math.cos(t / 1000));
        const b = 128;

        imageData.data[p + 0] = r;
        imageData.data[p + 1] = g;
        imageData.data[p + 2] = b;
        imageData.data[p + 3] = 255;
      }

      ctx.putImageData(imageData, 0, 0);
    }());

    return () => {
      cancelAnimationFrame(frame);
    };
  });
</script>

<style>
  canvas {
    width: 100%;
    height: 100%;
    background-color: #666;
    -webkit-mask: url(svelte-logo-mask.svg) 50% 50% no-repeat;
    mask: url(svelte-logo-mask.svg) 50% 50% no-repeat;
  }
</style>

<canvas bind:this={canvas}
        width={32}
        height={32}></canvas>
```

## Component Bindings
[Back to Top](#bindings)

Just as you can bind to properties of DOM elements, you can bind to component props. For example, we can bind to the `value` prop of `<Keypad>` as though it were a form element:

```svelte
<Keypad bind:value={pin}
        on:submit={handleSubmit}/>
```

> Use component bindings sparingly. It can be difficult to track the flow of data around your application if you have too many of them, especially if there is no 'single source of truth'.