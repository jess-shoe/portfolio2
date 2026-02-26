<script>
  export let data = {};
</script>

<article>
  <h2>{data.title}</h2>

  <img src={data.image} alt={data.title} />

  <p>
    {#if data.link}
      {data.description}
      <a href={data.link}>View project</a>
    {:else}
      {data.description}
    {/if}
  </p>
</article>

<style>
  article {
    display: grid;
    grid-template-rows: auto auto 1fr;
    gap: 0.75em;
    padding: 0.75em;
    border: 1px solid oklch(85% 3% 200);
    border-radius: 0.5em;
    background: oklch(98% 1% 300);
  }

  h2 {
    margin: 0;
    font-size: 120%;
  }

  img {
    border-radius: 0.4em;
  }

  p {
    margin: 0;
    font-size: 95%;
  }

  a {
    margin-left: 0.5em;
    color: var(--color-accent);
    text-decoration: none;
    font-weight: 500;
  }

  a:hover {
    text-decoration: underline;
  }
</style>