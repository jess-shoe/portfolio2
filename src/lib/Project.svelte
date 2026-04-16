<script>
  export let data = {};
</script>

<article>
  <header>
    <h2>{data.title}</h2>
    <span class="year">{data.year}</span>
  </header>

  <img src={data.image} alt={data.title} />

  <p>{data.description}</p>

  {#if data.url}
    <p class="project-link">
      <a href={data.url} target="_blank" rel="noopener noreferrer">View project</a>
    </p>
  {/if}
</article>

<style>
  article {
    display: grid;
    grid-template-rows: auto auto auto 1fr;
    gap: 0.75em;
    padding: 0.75em;

    border: 1px solid color-mix(in oklch, CanvasText 15%, Canvas);
    border-radius: 0.5em;

    background: Canvas;
  }

  header {
    display: flex;
    justify-content: space-between;
    align-items: baseline;
  }

  h2 {
    margin: 0;
    font-size: 120%;
  }

  .year {
    font-size: 90%;
    opacity: 0.7;
  }

  img {
    width: 100%;
    border-radius: 0.4em;
  }

  p {
    margin: 0;
    font-size: 95%;
  }

  .project-link a {
    color: var(--color-accent);
    text-decoration: none;
    font-weight: 500;
  }

  .project-link a:hover {
    text-decoration: underline;
  }
</style>