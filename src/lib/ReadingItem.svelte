<script>
  export let data = {};
</script>

<div class="reading-item">
  <img src={data.image} alt={data.title} />
  <div class="details">
    <h3>{data.title}</h3>
    <p class="author">{data.author}</p>
  </div>
</div>

<style>
  .reading-item {
    display: flex;
    align-items: flex-start;
    gap: 1em;
    padding: 0.75em;
    border: 1px solid oklch(85% 3% 200);
    border-radius: 0.5em;
    background: oklch(98% 1% 300);
  }

  img {
    width: 80px;
    height: auto;
    border-radius: 0.4em;
    flex-shrink: 0;
  }

  .details h3 {
    margin: 0;
    font-size: 1.1em;
  }

  .details .author {
    margin: 0.2em 0 0 0;
    font-size: 0.9em;
    color: oklch(50% 3% 200);
  }
</style>