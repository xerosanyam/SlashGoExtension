<script lang="ts">
  import { afterUpdate } from "svelte";
  let player: any;

  afterUpdate(() => {
    player.play();
  });
</script>

<video autoplay bind:this={player} class="arrow " alt="" controls>
  <source src="https://i.giphy.com/media/W3MyhCrgfBlXDV349M/giphy.webp" />
  <track kind="captions" />
</video>

<style>
  /* .arrow {
    transform: rotate(0.73turn) translate(-110%, 0px);
  } */
</style>
