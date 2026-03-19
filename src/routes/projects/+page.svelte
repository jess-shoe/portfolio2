<script>
  import Bar from '$lib/Bar.svelte';
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import ProjectNarrative from "$lib/ProjectNarrative.svelte";

  // existing project stats
  let years = projects.map(proj => proj.year);
  let range = Math.max(...years) - Math.min(...years);

  // new D3 data
  let rawData = [];
  let wrangled = [];
  let percentages = [];
  let totalLines = 0;

  onMount(async () => {
    try {
      rawData = await d3.json('/lab6_example.json');

      if (!rawData) {
        console.error("Failed to load data");
        return;
      }

      totalLines = d3.sum(rawData, d => d.lines);

      wrangled = d3.rollups(
        rawData,
        v => d3.sum(v, d => d.lines),
        d => d.language
      );

      percentages = d3.rollups(
        rawData,
        v => d3.sum(v, d => d.lines) / totalLines,
        d => d.language
      );

      console.log("wrangled:", wrangled);
    } catch (err) {
      console.error("Error loading JSON:", err);
    }
  });

  $: barData = d3.rollups(
    projects,
    v => v.length,
    d => d.year
  ).map(([year, count]) => ({
    label: String(year),
    value: count
  }));
</script>

<svelte:head>
  <title>Projects | Jessica Shoemaker</title>
</svelte:head>

<h1>{projects.length} Projects over {range} Years</h1>

<Bar data={barData} />

<p>
Scroll down to see a timeline of my projects and how they’ve helped me grow
as a programmer and designer.
</p>

<ProjectNarrative />

<p class="outro">
Thanks for scrolling through my project story! Feel free to explore all the
projects below.
</p>

<section class="wrangling">
  <h2>Data Wrangling Practice</h2>

  <h3>Totals by Language</h3>
  <pre>{JSON.stringify(wrangled, null, 2)}</pre>

  <h3>Percentages</h3>
  {#each percentages as [lang, pct]}
    <p>{lang}: {(pct * 100).toFixed(1)}%</p>
  {/each}
</section>

<!-- existing projects -->
<div class="projects">
  {#each projects as p}
    <Project data={p} />
  {/each}
</div>

<style>
.outro {
  margin-bottom: 3rem;
}

.wrangling {
  margin: 2rem 0;
  padding: 1rem;
  border: 1px solid #ccc;
  border-radius: 0.5rem;
}
</style>