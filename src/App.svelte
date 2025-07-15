<script>
  import data from "./data/data.js";

  import {
    forceSimulation, // For simulation
    forceLink, // For links between nodes
    forceManyBody, // For repulsion/attraction between nodes
    forceX, // For horizontal positioning
    forceY, // For vertical positioning
    forceCollide, // For collision detection
  } from "d3-force";

  const simulation = forceSimulation(data);

  $: {
    simulation
      .force(
        "x",
        forceX()
          .x((d) => xScale(d.happiness))
          .strength(0.8)
      )
      .force(
        "y",
        forceY()
          .y((d) => groupByContinent ? yScale(d.continent) : innerHeight / 2)
          .strength(0.2)
      )
      .force(
        "collide",
        forceCollide().radius((d) => radiusScale(d.happiness)) // Collision radius
      )
      .alpha(0.3) // The rate at which the simulation finishes.
      .alphaDecay(0.0005) // The rate at which the alpha approaches 0.
      .restart(); // Restart the simulation
  }

  // $: nodes = simulation.nodes();
  let nodes = [];
  simulation.on("tick", () => {
    nodes = simulation.nodes();
  });

  let width = 400;
  let height = 400;

  const margin = {
    top: 0,
    right: 0,
    bottom: 30,
    left: 0,
  };

  $: innerWidth = width - margin.left - margin.right;
  let innerHeight = height - margin.top - margin.bottom;

  import { scaleLinear, scaleBand, scaleOrdinal, scaleSqrt } from "d3-scale";
  import { extent, mean } from "d3-array";

  $: xScale = scaleLinear()
    .domain([1, 9]) // INPUT
    .range([0, innerWidth]); // OUTPUT

  import { rollups } from "d3-array";

  // Generate the average for each continent
  const continents = rollups(
    data,
    (v) => mean(v, (d) => d.happiness),
    (d) => d.continent
  ) // Group by continent
    .sort((a, b) => a[1] - b[1]) // Sort by average happiness
    .map((d) => d[0]); // Extract continent names

  // Add a color to each circle according to its continent
  const colorRange = [
    "#dda0dd", // Africa
    "#fe7f2d", // Asia
    "#fcca46", // South America
    "#a1c181", // North America
    "#8abfba", // Europe
    "#eae2b7", // Oceania
  ];
  const colorScale = scaleOrdinal().domain(continents).range(colorRange);

  // Scale for the y-axis
  let yScale = scaleBand()
    .domain(continents)
    .range([innerHeight, 0])
    .paddingOuter(0.5);

  // Size of circles based on happiness score
  $: radiusScale = scaleSqrt()
    .domain([1, 9]) // INPUT
    .range(width < 568 ? [2, 6] : [3, 8]); // OUTPUT... if width is below 568px, use smaller circles

  import AxisX from "./components/AxisX.svelte";
  import AxisY from "./components/AxisY.svelte";
  import Legend from "./components/Legend.svelte";
  import Tooltip from "./components/Tooltip.svelte";
  import { fade } from "svelte/transition";

  let hovered; // For hovered circles

  let hoveredContinent; // Create variable for continent hover event in legend
  let groupByContinent = false; // Toggle for grouping by continent

</script>

<!-- Title and Legend -->
<h1>The happiest countries in the world</h1>
<h2>Afghanistan is the unhappiest country in the world with a score of 2.4/10, while Finland is the happiest with a score of 7.8/10. Click on the chart for a breakdown by continent, and hover over the legend or circles for a detailed look at each country. (Note: The year this data was published and its source were not referenced in the course material.)</h2>
<Legend {colorScale} bind:hoveredContinent={hoveredContinent} />

<!-- Chart -->
<!-- svelte-ignore a11y-click-events-have-key-events -->
<div class="chart-container" bind:clientWidth={width}
  on:click={() => { groupByContinent = !groupByContinent; hovered = null; }}
>
  <svg {width} {height} on:mouseleave={() => (hovered = null)}>
    <g
      class="inner-chart"
      transform="translate({margin.left}, {margin.top})"
    >
      <AxisX {xScale} height={innerHeight} width={innerWidth} />
      <AxisY {yScale} {groupByContinent}/>
      {#if hovered}
        <line
          x1={hovered.x}
          y1={hovered.y}
          x2={hovered.x}
          y2={innerHeight}
          stroke={colorScale(hovered.continent)}
          stroke-width="2"
          transition:fade={{ duration: 200 }}
        />
      {/if}
      {#each nodes as node}
        <!-- svelte-ignore a11y-no-noninteractive-tabindex -->
        <circle
          cx={node.x}
          cy={node.y}
          r={radiusScale(node.happiness)}
          fill={colorScale(node.continent)}
          stroke={hovered || hoveredContinent ? hovered === node || hoveredContinent === node.continent ? "black" : "transparent" : "#00000090"}
          opacity={hovered || hoveredContinent ? hovered === node || hoveredContinent === node.continent ? 1 : 0.5 : 1}
          on:mouseover={() => {
            hovered = node;
          }}
          on:focus={() => {
            hovered = node;
          }}
          tabindex="0"
          on:click={(e) => e.stopPropagation()}
        />
      {/each}
    </g>
  </svg>

  {#if hovered}
    <Tooltip 
      data={hovered}
      colorScale={colorScale}
      {width}
    />
  {/if}
</div>

<style>
  :global(.tick text, .axis-title) {
    font-size: 0.8rem;
    fill: grey;
    user-select: none;
  }

  :global(.axis-title) {
    font-size: 1rem;
  }

  h1 {
    font-size: 1.65rem;
    font-weight: 500;
    line-height: 1.8rem;
    margin: 0 0 5px 0;
  }

  h2 {
    padding: 0 0 1rem 0;
    line-height: 1.35rem;
  }

  circle {
    transition: stroke 300ms ease, opacity 300ms ease;
    cursor: pointer;
  }
</style>
