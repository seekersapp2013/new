<script>
  // Vendors
  import { onMount } from "svelte";
  import { Link } from "svelte-routing";

  // Components
  import Icon from "../../components/icon/icon.svelte";
  import AppTab from "../../components/app-tab/app-tab.svelte";

  import { Lang } from "../../store/lang";
  import { TrackerStore } from "../../store/tracker-store";
  import { FeatureStore } from "../../store/feature-store";
  import NPaths from "../../paths";
  import { UserStore } from "../../store/user-store";
  import Features from "../settings/features.svelte";
  const state = {
    mounted: false,
  };

  $: hideLabels = $UserStore.meta.hideLabels;

  onMount(() => {
    state.mounted = true;
  });
</script>

<style lang="scss">
  @import "../../scss/utils/__utils.scss";

  #app-tabs {
    --tab-height: 65px;

    height: calc(#{var(--tab-height)} + env(safe-area-inset-bottom));
    padding-bottom: calc(env(safe-area-inset-bottom));
    background-color: var(--footer-background);
    display: flex;
    justify-content: space-between;
    align-items: stretch;
    z-index: 1200;
    flex-shrink: 0;
    // padding-top: 10px;

    &.compact {
      --tab-height: 55px;
      height: calc(#{var(--tab-height)} + env(safe-area-inset-bottom));
      .n-row {
        max-height: var(--tab-height);
        min-height: var(--tab-height);
        height: var(--tab-height);
      }
    }

    .n-row {
      z-index: 10;
      max-height: var(--tab-height);
      min-height: var(--tab-height);
      height: var(--tab-height);
      flex-shrink: 0;
    }
  }
  :global(#app-tabs .notification) {
    position: absolute;
    top: 8px;
    right: calc(50% - 15px);
    width: 6px;
    height: 6px;
    background-color: var(--color-red);
    border-radius: 3px;
  }

  // :global(#app-tabs a svg) {
  //   height: 24px !important;
  //   width: 24px !important;
  //   margin-bottom: 3px;
  // }

  // :global(#app-tabs a[aria-current="page"] svg) {
  //   stroke: var(--color-primary-bright) !important;
  //   transform: scale(1.1);
  //   transition: all 0.2s ease-in-out;
  // }

  // :global(#app-tabs a[aria-current="page"] svg .fill) {
  //   stroke: var(--color-primary-bright) !important;
  // }
</style>

{#if state.mounted}
  <nav id="app-tabs" class={hideLabels ? 'compact' : ''}>
    <div class="n-row mw-500px mx-auto">

      <AppTab link={NPaths.routes.history()} icon="calendar" label={Lang.t('tabs.history')} />
      {#if $FeatureStore.dashboard}
        <AppTab link={NPaths.routes.dashboard()} icon="report" label={Lang.t('tabs.dashboard', 'Dash')} />
      {/if}
      <AppTab link="/" icon="grid" label={Lang.t('tabs.track', 'Track')}>
        {#if $TrackerStore.timers.length}
          <div class="notification" />
        {/if}
      </AppTab>
      {#if $FeatureStore.People}
        <AppTab link={NPaths.routes.People()} icon="user" label={Lang.t('tabs.People', 'People')} />
      {/if}
      <AppTab link={NPaths.routes.settings()} icon="settings" label={Lang.t('tabs.settings', 'Settings')} />

    </div>
  </nav>
{/if}
