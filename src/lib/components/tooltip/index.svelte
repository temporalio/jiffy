<script lang="ts">
  export let text = '';
  export let top = false;
  export let right = false;
  export let bottom = false;
  export let left = false;
  export let hide: boolean | null = false;
</script>

{#if hide}
  <slot />
{:else}
  <div class="wrapper relative inline-block">
    <slot />
    <div class="tooltip" class:left class:right class:bottom class:top>
      <div class="inline-block rounded-lg bg-gray-800 px-2 py-2">
        <span class="text-gray-100">{text}</span>
      </div>
    </div>
  </div>
{/if}

<style lang="postcss">
  .tooltip {
    @apply invisible absolute left-0 top-0 z-50 inline-block translate-x-12 whitespace-nowrap text-xs opacity-0 shadow-md transition-all;
  }

  .tooltip.top {
    @apply left-1/2 -mt-4 -translate-x-1/2 -translate-y-full;
  }
  .tooltip.bottom {
    @apply left-1/2 bottom-0 -mb-1 -translate-x-1/2 translate-y-full;
  }
  .tooltip.left {
    @apply left-0 -ml-2 -translate-x-full;
  }
  .tooltip.right {
    @apply right-0 -mr-2 translate-x-full;
  }

  .wrapper:hover .tooltip {
    @apply visible opacity-90;
  }
</style>
