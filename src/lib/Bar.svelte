<script>
  import * as d3 from 'd3';

  // data comes from parent now
  export let data = [];

  let width = 500;
  let height = 350;

  let margin = { top: 40, right: 150, bottom: 80, left: 60 };
  let innerWidth  = width  - margin.left - margin.right;
  let innerHeight = height - margin.top  - margin.bottom;

  let xAxis, yAxis;

  // scales
  $: xScale = d3.scaleBand()
    .domain(data.map(d => d.label))
    .range([0, innerWidth])
    .padding(0.2);

  $: yScale = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.value) || 1])
    .range([innerHeight, 0]);

  $: colorScale = d3.scaleOrdinal(d3.schemeTableau10)
    .domain(data.map(d => d.label));

  // axes
  $: if (xAxis && yAxis) {
    d3.select(xAxis).call(d3.axisBottom(xScale));

    d3.select(yAxis).call(
      d3.axisLeft(yScale)
        .tickFormat(d => Number.isInteger(d) ? d : "")
        .tickValues(d3.range(0, (d3.max(data, d => d.value) || 0) + 1))
    );
  }

  // find max bar
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
      Projects per Year
    </text>

    <!-- x-axis -->
    <g
      transform="translate({margin.left}, {margin.top + innerHeight})"
      bind:this={xAxis}
    />

    <!-- y-axis -->
    <g
      transform="translate({margin.left}, {margin.top})"
      bind:this={yAxis}
    />

    <!-- bars + labels + annotation -->
    <g transform="translate({margin.left}, {margin.top})">

      {#each data as d}
        <rect
          x={xScale(d.label)}
          y={yScale(d.value)}
          width={xScale.bandwidth()}
          height={innerHeight - yScale(d.value)}
          fill={colorScale(d.label)}
        />
      {/each}

      <!-- annotation -->
      {#if maxBar}
        <rect
          x={xScale(maxBar.label)}
          y={yScale(maxBar.value)}
          width={xScale.bandwidth()}
          height={innerHeight - yScale(maxBar.value)}
          fill="none"
          stroke="currentColor"
          stroke-width="2"
        />

        <line
          x1={xScale(maxBar.label) + xScale.bandwidth()}
          y1={yScale(maxBar.value) + (innerHeight - yScale(maxBar.value)) / 2 -35}
          x2={xScale(maxBar.label) + xScale.bandwidth() + 45}
          y2={yScale(maxBar.value) + (innerHeight - yScale(maxBar.value)) / 2 -35}
          stroke="currentColor"
        />

        <text
          x={xScale(maxBar.label) + xScale.bandwidth() + 45}
          y={yScale(maxBar.value) + (innerHeight - yScale(maxBar.value)) / 2 - 35}
          dominant-baseline="middle"
          class="annotation">
          Year with most projects
        </text>
      {/if}

      <!-- axis labels -->
      <text
        x={innerWidth / 2}
        y={innerHeight + margin.bottom - 10}
        text-anchor="middle"
        class="axis-label">
        Year
      </text>

      <text
        x={-(innerHeight / 2)}
        y={-margin.left + 20}
        text-anchor="middle"
        transform="rotate(-90)"
        class="axis-label">
        Number of Projects
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
    flex: 1;
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
    fill: currentColor;
  }

  .axis-label {
    font-size: 0.8em;
    fill: currentColor;
  }

  .annotation {
    font-size: 0.7em;
    font-style: italic;
  }
</style>