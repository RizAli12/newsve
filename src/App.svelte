<script>
  import { time } from "./stores.js";

  const formatter = new Intl.DateTimeFormat("en", {
    hour12: true,
    hour: "numeric",
    minute: "2-digit",
    second: "2-digit",
  });

  import { count } from "./stores.js";
  import Incrementer from "./Incrementer.svelte";
  import Decrementer from "./Decrementer.svelte";
  import Resetter from "./Resetter.svelte";

  let count_value;

  count.subscribe((value) => {
    count_value = value;
  });

  import { tick } from "svelte";

  let text1 = `Select some text and hit the tab key to toggle uppercase`;

  async function handleKeydown11(event) {
    if (event.key !== "Tab") return;

    event.preventDefault();

    const { selectionStart, selectionEnd, value } = this;
    const selection = value.slice(selectionStart, selectionEnd);

    const replacement = /[a-z]/.test(selection)
      ? selection.toUpperCase()
      : selection.toLowerCase();

    text1 =
      value.slice(0, selectionStart) + replacement + value.slice(selectionEnd);

    await tick();
    this.selectionStart = selectionStart;
    this.selectionEnd = selectionEnd;
  }

  import Timer from "./Timer.svelte";

  let open = true;
  let seconds = 0;

  const toggle = () => (open = !open);
  const handleTick = () => (seconds += 1);

  import { onMount } from "svelte";

  let photos = [];

  onMount(async () => {
    const res = await fetch(
      `https://jsonplaceholder.typicode.com/photos?_limit=20`
    );
    photos = await res.json();
  });

  import InputField from "./InputField.svelte";

  let field;

  import Keypad from "./Keypad.svelte";

  let pin;
  $: view = pin ? pin.replace(/\d(?!$)/g, "•") : "enter your pin";

  function handleSubmit() {
    alert(`submitted ${pin}`);
  }

  export let name;

  let w;
  let h;
  let size = 42;
  let text = "edit me";

  let todos = [
    { done: false, text: "finish Svelte tutorial" },
    { done: false, text: "build an app" },
    { done: false, text: "world domination" },
  ];

  function add() {
    todos = todos.concat({ done: false, text: "" });
  }

  function clear() {
    todos = todos.filter((t) => !t.done);
  }

  $: remaining = todos.filter((t) => !t.done).length;

  // These values are bound to properties of the video
  let time = 0;
  let duration;
  let paused = true;

  let showControls = true;
  let showControlsTimeout;

  // Used to track time of last mouse down event
  let lastMouseDown;

  function handleMove(e) {
    // Make the controls visible, but fade out after
    // 2.5 seconds of inactivity
    clearTimeout(showControlsTimeout);
    showControlsTimeout = setTimeout(() => (showControls = false), 2500);
    showControls = true;

    if (!duration) return; // video not loaded yet
    if (e.type !== "touchmove" && !(e.buttons & 1)) return; // mouse not down

    const clientX = e.type === "touchmove" ? e.touches[0].clientX : e.clientX;
    const { left, right } = this.getBoundingClientRect();
    time = (duration * (clientX - left)) / (right - left);
  }

  // we can't rely on the built-in click event, because it fires
  // after a drag — we have to listen for clicks ourselves
  function handleMousedown(e) {
    lastMouseDown = new Date();
  }

  function handleMouseup(e) {
    if (new Date() - lastMouseDown < 300) {
      if (paused) e.target.play();
      else e.target.pause();
    }
  }

  function format(seconds) {
    if (isNaN(seconds)) return "...";

    const minutes = Math.floor(seconds / 60);
    seconds = Math.floor(seconds % 60);
    if (seconds < 10) seconds = "0" + seconds;

    return `${minutes}:${seconds}`;
  }
</script>

<main>
  <h1>The time is {formatter.format($time)}</h1>
  <h1>The count is {$count}</h1>

  <Incrementer />
  <Decrementer />
  <Resetter />

  <div>
    <button on:click={toggle}>{open ? "Close" : "Open"} Timer</button>
    <p>
      The Timer component has been open for
      {seconds}
      {seconds === 1 ? "second" : "seconds"}
    </p>
    {#if open}
      <Timer callback={handleTick} />
    {/if}
  </div>

  <h1>Hello {name}!</h1>
  <p>
    Visit the <a href="https://svelte.dev/tutorial">Svelte tutorial</a> to learn
    how to build Svelte apps.
  </p>

  <input type="range" bind:value={size} />
  <input bind:value={text} />

  <p>size: {w}px x {h}px</p>

  <div bind:clientWidth={w} bind:clientHeight={h}>
    <span style="font-size: {size}px">{text}</span>
  </div>

  <InputField bind:this={field} />

  <button on:click={() => field.focus()}>Focus field</button>

  <h1>Todos</h1>

  {#each todos as todo}
    <div class:done={todo.done}>
      <input type="checkbox" bind:checked={todo.done} />

      <input placeholder="What needs to be done?" bind:value={todo.text} />
    </div>
  {/each}

  <p>{remaining} remaining</p>

  <button on:click={add}> Add new </button>

  <button on:click={clear}> Clear completed </button>

  <h1>Caminandes: Llamigos</h1>
  <p>
    From <a href="https://studio.blender.org/films">Blender Studio</a>. CC-BY
  </p>

  <div>
    <video
      poster="https://sveltejs.github.io/assets/caminandes-llamigos.jpg"
      src="https://sveltejs.github.io/assets/caminandes-llamigos.mp4"
      on:mousemove={handleMove}
      on:touchmove|preventDefault={handleMove}
      on:mousedown={handleMousedown}
      on:mouseup={handleMouseup}
      bind:currentTime={time}
      bind:duration
      bind:paused
    >
      <track kind="captions" />
    </video>

    <div class="controls" style="opacity: {duration && showControls ? 1 : 0}">
      <progress value={time / duration || 0} />

      <div class="info">
        <span class="time">{format(time)}</span>
        <span>click anywhere to {paused ? "play" : "pause"} / drag to seek</span
        >
        <span class="time">{format(duration)}</span>
      </div>
    </div>
  </div>

  <h1 style="color: {pin ? '#333' : '#ccc'}">{view}</h1>

  <Keypad bind:value={pin} on:submit={handleSubmit} />

  <h1>Photo album</h1>

  <div class="photos">
    {#each photos as photo}
      <figure>
        <img src={photo.thumbnailUrl} alt={photo.title} />
        <figcaption>{photo.title}</figcaption>
      </figure>
    {:else}
      <!-- this block renders when photos.length === 0 -->
      <p>loading...</p>
    {/each}
  </div>

  <textarea value={text1} on:keydown={handleKeydown11} />
</main>

<style>
  main {
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  }

  h1 {
    color: #ff3e00;
    text-transform: uppercase;
    font-size: 4em;
    font-weight: 100;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }

  .done {
    opacity: 0.4;
  }

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

  .time:last-child {
    text-align: right;
  }

  progress {
    display: block;
    width: 100%;
    height: 10px;
    -webkit-appearance: none;
    appearance: none;
  }

  progress::-webkit-progress-bar {
    background-color: rgba(0, 0, 0, 0.2);
  }

  progress::-webkit-progress-value {
    background-color: rgba(255, 255, 255, 0.6);
  }

  video {
    width: 100%;
  }

  .photos {
    width: 100%;
    display: grid;
    grid-template-columns: repeat(5, 1fr);
    grid-gap: 8px;
  }

  figure,
  img {
    width: 100%;
    margin: 0;
  }

  textarea {
    width: 100%;
    height: 200px;
  }
</style>
