<script>
  import { onMount } from "svelte";
  let theme = "light";
  let icon = "🌞";

  // Check for saved theme or system preference
  onMount(() => {
    const saved = localStorage.getItem("theme");
    if (saved) {
      theme = saved;
    } else if (window.matchMedia("(prefers-color-scheme: dark)").matches) {
      theme = "dark";
    }
    updateTheme();
  });

  function toggleTheme() {
    theme = theme === "light" ? "dark" : "light";
    localStorage.setItem("theme", theme);
    updateTheme();
  }

  function updateTheme() {
    document.documentElement.setAttribute("data-theme", theme);
    icon = theme === "dark" ? "🌙" : "🌞";
  }
</script>

<header class="header">
  <div class="container">
    <div class="header-content">
      <div class="logo">
        <h1>SwiftResize</h1>
      </div>
      <div class="theme-toggle" aria-label="Theme toggle">
        <button
          id="themeBtn"
          class="btn-ghost"
          type="button"
          title="Switch theme"
          on:click={toggleTheme}
        >
          <span id="themeIcon" aria-hidden="true">{icon}</span>
          <span class="sr-only">Toggle theme</span>
        </button>
      </div>
    </div>
  </div>
</header>
