<script>
  import projects from "$lib/projects.json";
</script>

<nav>
    <a href=".">Home</a>
    <a class="current" href="projects">Projects</a>
    <a href="contact">Contact</a>
    <a href="resume">Resume</a>
    <a href="https://github.com/jtshoe" target="_blank">GitHub</a>
</nav>

<h1>Projects ({projects.length})</h1>

<div class="projects">
  {#each projects as p}
    <article>
      <h2>{p.title}</h2>

      <img src={p.image} alt={p.title} />

      <p>
        {#if p.link}
          {p.description} 
          <a href={p.link}>View project</a>
        {:else}
          {p.description}
        {/if}
      </p>
    </article>
  {/each}
</div>