<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";

  import reading from "$lib/reading.json";
  import ReadingItem from "$lib/ReadingItem.svelte";
</script>

<svelte:head>
  <title>Home | Jessica Shoemaker</title>
</svelte:head>

<h1>Jessica Shoemaker</h1>
<p>Hello! My name is Jessica and I am currently a senior majoring in 6-3 and 21M music. I am excited to learn about data visualizations as they are the creative intersection of computer science and design! I love to sing, solve puzzles, and go on interesting sidequests in Boston. </p>
<img src="images/brandenburggate.jpeg" alt="A picture of me at the Brandenburg Gate in Berlin this January">

<section class="reading">
  <h2>What I'm Reading</h2>
  <div class="reading-list">
    {#each reading as item}
      <ReadingItem data={item} />
    {/each}
  </div>
</section>

<h2>Latest Projects</h2>

<div class="projects highlights">
  {#each projects.slice(0, 3) as p}
    <Project data={p} />
  {/each}
</div>