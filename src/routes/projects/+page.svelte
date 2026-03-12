<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import ProjectNarrative from "$lib/ProjectNarrative.svelte";

  let years = projects.map(proj => proj.year);
  let range = Math.max(...years) - Math.min(...years);
</script>

<svelte:head>
  <title>Projects | Jessica Shoemaker</title>
</svelte:head>

<h1>{projects.length} Projects over {range} Years</h1>

<p>
Scroll down to see a timeline of my projects and how they’ve helped me grow
as a programmer and designer.
</p>

<ProjectNarrative />

<p class="outro">
Thanks for scrolling through my project story! Feel free to explore all the
projects below.
</p>

<div class="projects">
  {#each projects as p}
    <Project data={p} />
  {/each}
</div>

<style>
.outro {
  margin-bottom: 3rem;
}
</style>