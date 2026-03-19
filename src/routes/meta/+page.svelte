<script>
  import { base } from '$app/paths';
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import BarHorizontal from '$lib/BarHorizontal.svelte';

  let locData = [];
  let barData = [];

  onMount(async () => {
    locData = await d3.csv(`${base}/loc.csv`, row => ({
      ...row,
      line: Number(row.line),
      length: Number(row.length),
      depth: Number(row.depth)
    }));

    // WRANGLE
    barData = d3.rollups(
      locData,
      v => v.length,
      d => d.type
    ).map(([type, count]) => ({
      label: type,
      value: count
    }));
  });
</script>

<h1>Codebase Analysis</h1>

<p>
This chart shows how many lines of code are written in each language.
</p>

<BarHorizontal data={barData} />