<script lang="ts">
  import { Link } from "svelte-routing";
  import NPaths from "../../paths";
  import { UserStore } from "../../store/user-store";
  import Ripple from "../button/ripple.svelte";
  import Icon from "../icon/icon.svelte";

  $: hideLabels = $UserStore.meta.hideLabels;

  export let label: string;
  export let icon: string;
  export let link: string;

  let hit: any;
</script>

<style lang="scss">
  .tab-wrap {
    display: flex;
    flex-grow: 1;
    height: 100%;
    align-items: center;
  }
  :global(.tab-wrap a) {
    position: relative;
    color: var(--color-inverse-2);
    display: inline-flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    position: relative;
    padding: 6px 10px;
    font-size: 0.7rem;
    height: 100%;
    text-decoration: none;
    flex-grow: 1;
    flex-shrink: 1;
    // opacity: 0.6;
    transition: all 0.2s ease-in-out;
    min-width: 30px;
    width: 100%;
    i {
      pointer-events: none;
      font-size: 1.2rem;
      transition: all 0.2s ease-in-out;
    }
    label {
      pointer-events: none;
      margin-bottom: 0;
    }

    // When Active
    &[aria-current="page"] {
      color: var(--color-primary-bright);
      svg {
        transition: all 0.2s ease-in-out;
      }
      &:after {
        content: "";
      }
    }
  }
</style>

<div class="tab-wrap tab-{label}">
  <Link to={link}>
    <slot />
    <Icon name={icon} />
    {#if !hideLabels}
      <label>{label}</label>
    {/if}
  </Link>
</div>
