<script lang="ts">
  import favicon from "$lib/assets/favicon.svg";
  import { onMount } from "svelte";
  import { Buffer } from "buffer";

  onMount(() => {
    if (!window.Buffer) {
      window.Buffer = Buffer;
    }
  });

  let { children } = $props();
</script>

<svelte:head>
  <link rel="icon" href={favicon} />
</svelte:head>

{@render children()}
