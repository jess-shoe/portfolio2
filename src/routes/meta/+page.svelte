<script>
  import { base } from '$app/paths';
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import { computePosition, autoPlacement, offset } from '@floating-ui/dom';
  import BarHorizontal from '$lib/BarHorizontal.svelte';
  import LineChart from '$lib/LineChart.svelte';

  let locData = [];
  let commits = [];
  let clickedCommits = [];
  let brushSelection = null;
  let brushedCommits = [];
  let selectedCommits = [];
  let barData = [];
  let linesByDate = [];

  let width = 900;
  let height = 420;

  let margin = { top: 30, right: 30, bottom: 50, left: 70 };

  let usableArea = {
    top: margin.top,
    right: width - margin.right,
    bottom: height - margin.bottom,
    left: margin.left
  };
  usableArea.width = usableArea.right - usableArea.left;
  usableArea.height = usableArea.bottom - usableArea.top;

  let svg;
  let xAxis;
  let yAxis;
  let yAxisGridlines;

  let hoveredIndex = -1;
  let hoveredCommit = {};
  let commitTooltip;
  let tooltipPosition = { x: 0, y: 0 };

  $: hoveredCommit = commits[hoveredIndex] ?? hoveredCommit ?? {};

  $: minDate = d3.min(commits, d => d.datetime);
  $: maxDate = d3.max(commits, d => d.datetime);
  $: maxDatePlusOne = maxDate ? d3.timeDay.offset(maxDate, 1) : null;

  $: xScale = d3.scaleTime()
    .domain(minDate && maxDatePlusOne ? [minDate, maxDatePlusOne] : [new Date(), new Date()])
    .range([usableArea.left, usableArea.right])
    .nice();

  $: yScale = d3.scaleLinear()
    .domain([0, 24])
    .range([usableArea.bottom, usableArea.top]);

  $: rScale = d3.scaleSqrt()
    .domain(d3.extent(commits, d => d.totalLines))
    .range([5, 30]);

  function brushed(evt) {
    brushSelection = evt.selection;
  }

  function isCommitBrushed(commit) {
    if (!brushSelection) return false;

    let [[x0, y0], [x1, y1]] = brushSelection;
    let x = xScale(commit.datetime);
    let y = yScale(commit.hourFrac);

    return x >= x0 && x <= x1 && y >= y0 && y <= y1;
  }

  $: brushedCommits = brushSelection ? commits.filter(isCommitBrushed) : [];

  $: selectedCommits = Array.from(new Set([...clickedCommits, ...brushedCommits]));

  $: selectedLines =
    (selectedCommits.length > 0 ? selectedCommits : commits).flatMap(d => d.lines);

  $: selectedCounts = d3.rollup(
    selectedLines,
    lines => lines.length,
    d => d.type
  );

  $: allLanguages = Array.from(new Set(locData.map(d => d.type)));

  $: barData = allLanguages.map(language => ({
    label: language,
    value: selectedCounts.get(language) ?? 0
  }));

  $: {
    if (xAxis && yAxis && yAxisGridlines) {
      d3.select(xAxis).call(d3.axisBottom(xScale));
      d3.select(yAxis).call(
        d3.axisLeft(yScale).tickFormat(d => String(d % 24).padStart(2, '0') + ':00')
      );
      d3.select(yAxisGridlines).call(
        d3.axisLeft(yScale)
          .tickFormat('')
          .tickSize(-usableArea.width)
      );
    }
  }

  $: {
    if (svg) {
      d3.select(svg)
        .call(
          d3.brush()
            .extent([[usableArea.left, usableArea.top], [usableArea.right, usableArea.bottom]])
            .on('start brush end', brushed)
        );

      d3.select(svg).selectAll('.dots, .overlay ~ *').raise();
    }
  }

  $: {
    let rolled = d3.rollups(
      locData,
      v => v.length,
      d => d3.timeDay.floor(d.datetime)
    ).map(([date, count]) => ({ date, count }));

    let dateExtent = d3.extent(rolled, d => d.date);

    if (!dateExtent[0] || !dateExtent[1]) {
      linesByDate = [];
    } else {
      let allDays = d3.timeDays(dateExtent[0], d3.timeDay.offset(dateExtent[1], 1));

      linesByDate = allDays.map(date => ({
        date,
        count: rolled.find(d => +d.date === +date)?.count ?? 0
      }));
    }
  }

  async function dotInteraction(index, evt) {
    let hoveredDot = evt.target;

    if (evt.type === 'mouseenter') {
      hoveredIndex = index;
      tooltipPosition = await computePosition(hoveredDot, commitTooltip, {
        strategy: 'fixed',
        middleware: [
          offset(5),
          autoPlacement()
        ]
      });
    } else if (evt.type === 'mouseleave') {
      hoveredIndex = -1;
    } else if (evt.type === 'click') {
      let commit = commits[index];

      if (!clickedCommits.includes(commit)) {
        clickedCommits = [...clickedCommits, commit];
      } else {
        clickedCommits = clickedCommits.filter(c => c !== commit);
      }
    }
  }

  onMount(async () => {
    locData = await d3.csv(`${base}/loc.csv`, row => ({
      ...row,
      line: Number(row.line),
      length: Number(row.length),
      depth: Number(row.depth),
      date: new Date(row.date + 'T00:00' + row.timezone),
      datetime: new Date(row.datetime)
    }));

    commits = d3.groups(locData, d => d.commit).map(([commit, lines]) => {
      let first = lines[0];
      let { author, date, time, timezone, datetime } = first;

      return {
        id: commit,
        url: `https://github.com/jtshoe/portfolio/commit/${commit}`,
        author,
        date,
        time,
        timezone,
        datetime,
        hourFrac: datetime.getHours() + datetime.getMinutes() / 60,
        totalLines: lines.length,
        lines
      };
    });

    commits = d3.sort(commits, d => -d.totalLines);
  });
</script>

<h1>Codebase Analysis</h1>

<p>
  This page shows when commits happened, how large they were, and how selected commits
  change the language breakdown of the site.
</p>

<svg bind:this={svg} viewBox={`0 0 ${width} ${height}`}>
  <g
    class="gridlines"
    transform={`translate(${usableArea.left}, 0)`}
    bind:this={yAxisGridlines}
  />

  <g
    transform={`translate(0, ${usableArea.bottom})`}
    bind:this={xAxis}
  />

  <g
    transform={`translate(${usableArea.left}, 0)`}
    bind:this={yAxis}
  />

  <g class="dots">
    {#each commits as commit, index (commit.id)}
      <circle
        cx={xScale(commit.datetime)}
        cy={yScale(commit.hourFrac)}
        r={rScale(commit.totalLines)}
        fill="steelblue"
        fill-opacity="0.7"
        class:selected={selectedCommits.includes(commit)}
        on:mouseenter={evt => dotInteraction(index, evt)}
        on:mouseleave={evt => dotInteraction(index, evt)}
        on:click={evt => dotInteraction(index, evt)}
      />
    {/each}
  </g>
</svg>

<dl
  class="info tooltip"
  bind:this={commitTooltip}
  hidden={hoveredIndex === -1}
  style={`top: ${tooltipPosition.y}px; left: ${tooltipPosition.x}px;`}
>
  <dt>Commit</dt>
  <dd><a href={hoveredCommit.url} target="_blank" rel="noreferrer">{hoveredCommit.id}</a></dd>

  <dt>Date</dt>
  <dd>{hoveredCommit.datetime?.toLocaleString('en', { dateStyle: 'full' })}</dd>

  <dt>Time</dt>
  <dd>{hoveredCommit.datetime?.toLocaleString('en', { timeStyle: 'short' })}</dd>

  <dt>Author</dt>
  <dd>{hoveredCommit.author}</dd>

  <dt>Lines edited</dt>
  <dd>{hoveredCommit.totalLines}</dd>
</dl>

<BarHorizontal
  data={barData}
  title={selectedCommits.length > 0
    ? `Language Breakdown: ${selectedCommits.length} Selected Commits`
    : 'Language Breakdown of Entire Website'}
/>

<LineChart data={linesByDate} />

<style>
  svg {
    max-width: 100%;
    height: auto;
    overflow: visible;
    display: block;
    margin-bottom: 1.5rem;
  }

  .gridlines {
    stroke-opacity: 0.2;
  }

  circle {
    transition: 200ms;
  }

  circle:hover {
    fill: darkgreen;
  }

  .selected {
    fill: var(--color-accent);
  }

  dl.info {
    display: grid;
    grid-template-columns: auto 1fr;
    gap: 0.25rem 0.75rem;
    margin: 1rem 0;
    padding: 0;
    transition-duration: 500ms;
    transition-property: opacity, visibility;
  }

  dl.info dt,
  dl.info dd {
    margin: 0;
  }

  dl.info dt {
    font-weight: 600;
    opacity: 0.7;
  }

  .tooltip {
    position: fixed;
    z-index: 10;
    background: oklch(100% 0 0 / 0.85);
    color: black;
    padding: 0.75rem 1rem;
    border-radius: 0.5rem;
    box-shadow: 0 6px 24px oklch(0% 0 0 / 0.15);
    backdrop-filter: blur(6px);
  }

  dl.info[hidden]:not(:hover, :focus-within) {
    opacity: 0;
    visibility: hidden;
  }

  @keyframes marching-ants {
    to {
      stroke-dashoffset: -8;
    }
  }

  svg :global(.selection) {
    fill-opacity: 0.1;
    stroke: black;
    stroke-opacity: 0.7;
    stroke-dasharray: 5 3;
    animation: marching-ants 2s linear infinite;
  }
</style>