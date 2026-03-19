<script>
  import * as d3 from 'd3';

  export let data = [];

  let width = 600;
  let height = 400;

  let margin = { top: 40, right: 150, bottom: 50, left: 100 };
  let innerWidth  = width  - margin.left - margin.right;
  let innerHeight = height - margin.top  - margin.bottom;

  let xAxis, yAxis;

  // scales
  $: xScale = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.value) || 1])
    .range([0, innerWidth]);

  $: yScale = d3.scaleBand()
    .domain(data.map(d => d.label))
    .range([0, innerHeight])
    .padding(0.2);

  $: colorScale = d3.scaleOrdinal(d3.schemeTableau10)
    .domain(data.map(d => d.label));

  // axes
  $: if (xAxis && yAxis) {
    d3.select(xAxis).call(d3.axisBottom(xScale));

    d3.select(yAxis).call(d3.axisLeft(yScale));
  }

  // max bar
  $: maxBar = d3.greatest(data, d => d.value);
</script>

<div class="container">
  <svg viewBox="0 0 {width} {height}">

    <!-- title -->
    <text
      x={margin.left + innerWidth / 2}
      y={margin.top / 2}
      text-anchor="middle"
      class="chart-title">
      Lines of Code per Language
    </text>

    <!-- axes -->
    <g
      transform="translate({margin.left}, {margin.top + innerHeight})"
      bind:this={xAxis}
    />

    <g
      transform="translate({margin.left}, {margin.top})"
      bind:this={yAxis}
    />

    <!-- bars -->
    <g transform="translate({margin.left}, {margin.top})">
      {#each data as d}
        <rect
          x={0}
          y={yScale(d.label)}
          width={xScale(d.value)}
          height={yScale.bandwidth()}
          fill={colorScale(d.label)}
        />
      {/each}

      <!-- annotation -->
      {#if maxBar}
        <rect
          x={0}
          y={yScale(maxBar.label)}
          width={xScale(maxBar.value)}
          height={yScale.bandwidth()}
          fill="none"
          stroke="black"
          stroke-width="2"
        />

        <line
          x1={xScale(maxBar.value)}
          y1={yScale(maxBar.label) + yScale.bandwidth()/2}
          x2={xScale(maxBar.value) + 40}
          y2={yScale(maxBar.label) + yScale.bandwidth()/2}
          stroke="black"
        />

        <text
          x={xScale(maxBar.value) + 45}
          y={yScale(maxBar.label) + yScale.bandwidth()/2}
          dominant-baseline="middle"
          class="annotation">
          Most lines of code
        </text>
      {/if}

      <!-- x label -->
      <text
        x={innerWidth / 2}
        y={innerHeight + margin.bottom - 10}
        text-anchor="middle"
        class="axis-label">
        Lines of Code
      </text>
    </g>
  </svg>

  <!-- legend -->
  <ul class="legend">
    {#each data as d}
      <li style="--color: {colorScale(d.label)}">
        <span class="swatch"></span>
        {d.label} <em>({d.value})</em>
      </li>
    {/each}
  </ul>
</div>

<style>
  svg {
    max-width: 100%;
    height: auto;
    overflow: visible;
  }

  .container {
    display: flex;
    gap: 2rem;
  }

  .legend {
    list-style: none;
    padding: 0;
    margin: 0;
    display: grid;
    gap: 0.5rem;
  }

  li {
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }

  .swatch {
    width: 12px;
    height: 12px;
    background-color: var(--color);
  }

  .chart-title {
    font-weight: bold;
  }

  .axis-label {
    font-size: 0.8em;
  }

  .annotation {
    font-size: 0.7em;
    font-style: italic;
  }
</style>