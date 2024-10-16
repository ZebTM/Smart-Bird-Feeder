<script>
  import SmartFeeder from './SmartFeeder.svelte';
</script>

<style>
  @import './app.css';

  .app-container {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    background-color: var(--bg-color);
  }

  .app-content {
    width: 90%;
    max-width: 1200px;
  }
</style>

<div class="app-container">
  <div class="app-content">
    <SmartFeeder />
  </div>
</div>
