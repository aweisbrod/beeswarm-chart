<script>
  export let data;
  export let colorScale;
  export let width;

  import { fly, fade } from "svelte/transition";

  let tooltipWidth;

  const xNudge = 5;
  const yNudge = 5;

  $: xPosition = data.x + tooltipWidth + xNudge > width ? data.x - tooltipWidth - xNudge : data.x + xNudge;
  $: yPosition = data.y + yNudge;
</script>

<!-- Tooltip content -->
<div
  class="tooltip"
  style="position: absolute; top: {yPosition}px; left: {xPosition}px;"
  bind:clientWidth={tooltipWidth}
  in:fly={{y: 10, duration: 200, delay: 200}}
  out:fade={{duration: 200}}
>
  <h1>{data.country}</h1>
  <div class="info">
    <span class="score">{data.happiness}/10</span>
    <span class="continent" style="background: {colorScale(data.continent)}"
      >{data.continent}</span
    >
  </div>

  <!-- Inline bar chart border -->
  <span class="bar background"></span>
  <span
    class="bar foreground"
    style="background: {colorScale(data.continent)}; width: {data.happiness *
      10}%"
  ></span>
</div>

<style>
  .tooltip {
    background-color: white;
    padding: 0.5rem;
    border-radius: 4px;
    box-shadow: 2px 2px 8px rgba(0, 0, 0, 0.1);
    pointer-events: none; /* Prevents tooltip from interfering with mouse events */
    transition: top 300ms ease, left 300ms ease;
  }

  h1 {
    font-weight: 600;
    font-size: 1rem;
  }

  .info {
    display: flex;
    justify-content: space-between;
    column-gap: 8px;
    margin-top: 0.25rem;
    align-items: center;
  }

  .score {
    font-size: 0.9rem;
  }

  .continent {
    padding: 3px;
    border-radius: 3px;
    text-transform: uppercase;
    font-size: 0.75rem;
  }

  .bar {
    position: absolute;
    bottom: 0;
    left: 0;
    height: 3px;
    width: 100%;
  }

  .bar.background {
    background: "lightgrey";
  }
</style>
