<script lang="ts">
  import { createEventDispatcher } from "svelte";
  import { get } from "svelte/store";

  export let size = "md";
  // export let color = undefined;
  export let bold = undefined;
  export let medium = undefined;
  export let tag = "div";
  export let className = "";
  export let faded = false;
  export let style = "";
  export let center = false;
  export let truncate = false;
  export let truncate2 = false;
  export let truncate3 = false;
  export let color = "";
  export let underline = false;
  export let inline = false;
  export let lineHeightMd = false;

  const dispatch = createEventDispatcher();
</script>

<style type="text/scss">
  .n-text {
    color: var(--color-inverse);
    line-height: 116%;
    white-space: normal;
    line-height: 125%;

    &.line-height-md {
      line-height: 140%;
    }

    &.xs {
      font-size: 0.7rem;
    }
    &.sm {
      font-size: 0.85rem;
    }
    &.md {
      font-size: 1rem;
    }
    &.lg {
      font-size: 1.25rem;
    }

    &.faded {
      opacity: 0.7;
    }
    &.xl {
      font-size: 1.5rem;
    }
    &.xxl {
      font-size: 2rem;
    }
    &.font-weight-bold {
      font-weight: bold;
    }
    &.font-weight-medium {
      font-weight: 500;
    }
  }
</style>

{#if tag === 'span' || inline}
  <span
    {style}
    on:click={() => {
      dispatch('click');
    }}
    class="n-text {` ${size || 'sm'} ${className} text-${color} ${truncate ? 'truncate' : ''} ${truncate2 ? 'truncate-2' : ''} ${truncate3 ? 'truncate-3' : ''} ${lineHeightMd ? 'line-height-md' : ''} ${center ? 'text-center' : ''} ${underline ? 'text-underline' : ''} ${faded ? 'faded ' : ''} ${bold ? 'font-weight-bold ' : ''} ${medium ? 'font-weight-medium ' : ''}`.trim()}">
    <slot />
  </span>
{:else}
  <div
    {style}
    on:click={() => {
      dispatch('click');
    }}
    class="n-text {` ${size || 'sm'} ${className} text-${color} ${truncate ? 'truncate' : ''} ${truncate2 ? 'truncate-2' : ''} ${truncate3 ? 'truncate-3' : ''} ${lineHeightMd ? 'line-height-md' : ''} ${center ? 'text-center' : ''} ${underline ? 'text-underline' : ''} ${faded ? 'faded ' : ''} ${bold ? 'font-weight-bold ' : ''} ${medium ? 'font-weight-medium ' : ''}`.trim()}">
    <slot />
  </div>
{/if}
