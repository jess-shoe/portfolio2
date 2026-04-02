<script>
  import * as d3 from 'd3';

  export let data = [];
  export let title = '';

  let width = 600;
  let height = 240;

  let margin = { top: 35, right: 95, bottom: 55, left: 95 };
  let innerWidth = width - margin.left - margin.right;
  let innerHeight = height - margin.top - margin.bottom;

  let xAxis, yAxis;

  $: xMax = d3.max(data, d => d.value) || 1;

  $: xScale = d3.scaleLinear()
    .domain([0, xMax])
    .range([0, innerWidth]);

  $: yScale = d3.scaleBand()
    .domain(data.map(d => d.label))
    .range([0, innerHeight])
    .padding(0.2);

  $: colorScale = d3.scaleOrdinal(d3.schemeTableau10)
    .domain(data.map(d => d.label));

  $: tickCount = Math.max(1, Math.min(10, xMax));

  $: if (xAxis && yAxis) {
    d3.select(xAxis).call(
      d3.axisBottom(xScale).ticks(tickCount).tickFormat(d3.format('d'))
    );

    d3.select(yAxis).call(d3.axisLeft(yScale));
  }

  $: maxBar = d3.greatest(data, d => d.value);
</script>

<div class="container">
  <svg viewBox={`0 0 ${width} ${height}`}>
    <text
      x={margin.left + innerWidth / 2}
      y={20}
      text-anchor="middle"
      class="chart-title"
    >
      {title}
    </text>

    <g
      transform={`translate(${margin.left}, ${margin.top + innerHeight})`}
      bind:this={xAxis}
    />

    <g
      transform={`translate(${margin.left}, ${margin.top})`}
      bind:this={yAxis}
    />

    <g transform={`translate(${margin.left}, ${margin.top})`}>
      {#each data as d}
        <rect
          x={0}
          y={yScale(d.label)}
          width={xScale(d.value)}
          height={yScale.bandwidth()}
          fill={colorScale(d.label)}
        />
      {/each}

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

        <text
          x={-(innerHeight / 2)}
          y={-60}
          text-anchor="middle"
          transform="rotate(-90)"
          class="axis-label"
        >
          Programming
          <tspan x={-(innerHeight / 2)} dy="1.2em">Language</tspan>
        </text>

        <text
          x={xScale(maxBar.value) + 12}
          y={yScale(maxBar.label) + yScale.bandwidth() / 2}
          dominant-baseline="middle"
          text-anchor="start"
          class="annotation"
        >
          Most lines of code
        </text>
      {/if}

      <text
        x={innerWidth / 2}
        y={innerHeight + 42}
        text-anchor="middle"
        class="axis-label"
      >
        Lines of Code
      </text>
    </g>
  </svg>

  <ul class="legend">
    {#each data as d}
      <li style={`--color: ${colorScale(d.label)}`}>
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
    gap: 1.25rem;
    align-items: flex-start;
    flex-wrap: wrap;
  }

  .legend {
    list-style: none;
    padding: 0;
    margin: 0;
    display: grid;
    gap: 0.35rem;
    font-size: 0.9rem;
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
    flex: none;
  }

  .chart-title {
    font-weight: 700;
    font-size: 0.95rem;
  }

  .axis-label {
    font-size: 0.72rem;
  }

  .annotation {
    font-size: 0.68rem;
    font-style: italic;
  }
</style>