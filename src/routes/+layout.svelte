<script>
  import { base } from "$app/paths";
  import { page } from "$app/stores";

  let pages = [
    { url: "/", title: "Home" },
    { url: "/projects", title: "Projects" },
    { url: "/resume", title: "Resume" },
    { url: "/contact", title: "Contact" },
    { url: "https://github.com/jtshoe", title: "GitHub" }
  ];

  // access browser storage safely
  let localStorage = globalThis.localStorage ?? {};

  // default theme
  let colorScheme = "light dark";

  if (localStorage.colorScheme) {
    colorScheme = localStorage.colorScheme;
  }

  // reference to <html>
  let root = globalThis.document?.documentElement;

  // reactive: apply theme
  $: root?.style.setProperty("color-scheme", colorScheme);

  // reactive: save theme
  $: localStorage.colorScheme = colorScheme;
</script>

<label class="color-scheme-switch">
  Theme:
  <select bind:value={colorScheme}>
    <option value="light dark">Automatic</option>
    <option value="light">Light</option>
    <option value="dark">Dark</option>
  </select>
</label>

<nav>
  {#each pages as p}
    <a
      href={p.url.startsWith("http") ? p.url : base + p.url}

      class:current={
        p.url === "/"
        ? $page.url.pathname === (base + "/")
        : $page.url.pathname.startsWith(base + p.url)
      }

      target={p.url.startsWith("http") ? "_blank" : null}
    >
      {p.title}
    </a>
  {/each}
</nav>

<slot />

<style>
nav {
  display: flex;
  gap: 1rem;
  margin-bottom: 1.5rem;
}

a {
  text-decoration: none;
  color: inherit;
}

.current {
  font-weight: bold;
  border-bottom: 2px solid currentColor;
}
</style>