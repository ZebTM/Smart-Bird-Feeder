<script>
  import SmartFeeder from './SmartFeeder.svelte';
</script>

<main>
  <SmartFeeder />
</main>

<style>
  :global(body) {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  main {
    width: 100%;
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  :global(.project-info a) {
    margin-right: 10px;
  }
</style>
