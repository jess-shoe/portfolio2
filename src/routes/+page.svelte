<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";

  import reading from "$lib/reading.json";
  import ReadingItem from "$lib/ReadingItem.svelte";

  import { onMount } from "svelte";

  let githubData = null;
  let loading = true;
  let error = null;

  onMount(async () => {
    try {
      let response = await fetch("https://api.github.com/users/jtshoe");

      if (!response.ok) {
        throw new Error("GitHub API request failed");
      }

      githubData = await response.json();
    } catch (err) {
      error = err;
    }

    loading = false;
  });
</script>

<svelte:head>
  <title>Home | Jessica Shoemaker</title>
</svelte:head>

<h1>Jessica Shoemaker</h1>

<p>
Hello! My name is Jessica and I am currently a senior majoring in 6-3 and 21M music. 
I am excited to learn about data visualizations as they are the creative intersection 
of computer science and design! I love to sing, solve puzzles, and go on interesting 
sidequests in Boston.
</p>

<img src="images/brandenburggate.jpeg" alt="A picture of me at the Brandenburg Gate in Berlin this January">

{#if loading}
  <p>Loading GitHub stats...</p>

{:else if error}
  <p>Something went wrong: {error.message}</p>

{:else}
  <section class="github">
    <h2>My GitHub Stats</h2>

    <dl>
      <div>
        <dt>Followers</dt>
        <dd>{githubData.followers}</dd>
      </div>

      <div>
        <dt>Following</dt>
        <dd>{githubData.following}</dd>
      </div>

      <div>
        <dt>Public Repositories</dt>
        <dd>{githubData.public_repos}</dd>
      </div>

      <div>
        <dt>Public Gists</dt>
        <dd>{githubData.public_gists}</dd>
      </div>
    </dl>
  </section>
{/if}

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

<style>
.github dl {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 1em;
  margin-top: 0.5em;
}

.github div {
  text-align: center;
}

dt {
  font-size: 0.9em;
  opacity: 0.7;
}

dd {
  font-size: 1.4em;
  font-weight: bold;
  margin: 0;
}
</style>