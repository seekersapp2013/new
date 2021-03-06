<script lang="ts">
  import PositivityBar from "../../components/positivity-bar/positivity-bar.svelte";
  import BarChart from "../../components/charts/bar-chart-2.svelte";
  import TrackerSmallBlock from "../../components/tracker-ball/tracker-small-block.svelte";
  import Text from "../../components/text/text.svelte";
  import { Widget } from "../../modules/dashboard/widget";
  import { createEventDispatcher } from "svelte";
  import Pie from "../../components/charts/pie.svelte";
  import Icon from "../../components/icon/icon.svelte";
  import Button from "../../components/button/button.svelte";
  import { strToColor } from "../../components/dymoji/dymoji";
  import dayjs from "dayjs";
  import { element } from "svelte/internal";
  import nid from "../../modules/nid/nid";
  import Calendar from "../../components/calendar/calendar.svelte";

  // Widgets
  import WidgetWhatTime from "./widgets/widget-what-time.svelte";
  import WidgetLastUsed from "./widgets/widget-last-used.svelte";
  import WidgetBarChart from "./widgets/widget-bar-chart.svelte";
  import WidgetValue from "./widgets/widget-value-display.svelte";
  import WidgetMinMax from "./widgets/widget-min-max.svelte";
  import WidgetPositivityPie from "./widgets/widget-positivity-pie.svelte";
  import WidgetMap from "./widgets/widget-map.svelte";
  import WidgetStreak from "./widgets/widget-streak.svelte";

  import { LastUsed } from "../../store/last-used";
  import { Interact } from "../../store/interact";
  import { DashboardStore } from "../../store/dashboard-store";
  import Spinner from "../../components/spinner/spinner.svelte";
  import { Lang } from "../../store/lang";

  const dispatch = createEventDispatcher();
  const id = nid();
  export let widget: Widget;

  function getClass(widget: Widget): string {
    let classes = [`type-${widget.type}`];
    let value;
    if (widget.stats) {
      if (widget.type == "last-used") {
        value = widget.stats.daysPast;
      } else {
        value = widget.math !== "sum" ? widget.stats.avg : widget.stats.sum;
      }
    }
    value = value || 0;

    if (widget.compareValue) {
      if (value > widget.compareValue) {
        classes.push(`over widget-${widget.compareOverColor}`);
      } else if (value < widget.compareValue) {
        classes.push(`over widget-${widget.compareUnderColor}`);
      }
    }
    classes.push(`widget-size-${widget.size}`);
    return classes.join(" ");
  }
</script>

<style lang="scss">
  @import "../../scss/utils/_utils";

  :global(.dashboard-text) {
    padding: 4px 16px;
    min-width: 100% !important;
    flex-grow: 1;
    display: flex;
    width: 100%;
    font-size: 0.9rem;
    color: var(--color-inverse-2);
    align-items: center;
    text-align: center !important;
    justify-content: center;
    max-width: 100% !important;

    &.widget-size-md {
      font-size: 1rem;
      font-weight: 500;
      padding: 8px 16px;
      line-height: 1.1rem;
    }
    &.widget-size-lg {
      padding: 8px 16px;
      font-size: 1.6rem;
      line-height: 1.7rem;
      font-weight: 700;
      padding: 16px;
    }
  }

  :global(.dashboard-text + .dashboard-text) {
    padding-top: 0px !important;
    padding-bottom: 16px;
  }

  .dashboard-widget {
    max-height: 300px;
    position: relative;
    &:after {
      content: "";
      position: absolute;
      bottom: 0;
      left: 0;
      right: 0;
      height: 4px;
    }
  }

  :global(.dashboard-widget.over .widget-footer .n-text, .dashboard-widget.under .widget-footer .n-text) {
    color: #fff !important;
  }
  :global(.dashboard-widget.widget-red .widget-footer .n-text) {
    color: var(--color-red) !important;
  }
  :global(.dashboard-widget.widget-blue .widget-footer .n-text) {
    color: var(--color-blue) !important;
  }
  :global(.dashboard-widget.widget-green .widget-footer .n-text) {
    color: var(--color-green) !important;
  }
  :global(.dashboard-widget.widget-orange .widget-footer .n-text) {
    color: var(--color-orange) !important;
  }

  :global(.dashboard-widget.widget-red:after) {
    background-color: var(--color-red) !important;
  }
  :global(.dashboard-widget.widget-blue:after) {
    background-color: var(--color-blue) !important;
  }
  :global(.dashboard-widget.widget-green:after) {
    background-color: var(--color-green) !important;
  }
  :global(.dashboard-widget.widget-orange:after) {
    background-color: var(--color-orange) !important;
  }

  :global(.dashboard-widget) {
    margin: 8px;
    display: flex;
    flex-direction: column;
    background-color: var(--color-solid);
    border-radius: 16px;
    box-shadow: var(--box-shadow-float);
    overflow: hidden;
    flex-shrink: 1;
    flex-grow: 1;

    // min-width: 148px;
    // @include media-breakpoint-down(xs) {
    //   min-width: calc(50% - 16px);
    // }
    // width: calc(50% - 16px);
    // max-width: calc(50% - 16px);

    // @include media-breakpoint-up(md) {
    //   width: calc(25% - 16px);
    //   max-width: calc(50% - 16px);
    // }

    .widget-header,
    .widget-footer {
      padding: 6px 8px;
      flex-grow: 0;
      flex-shrink: 0;
      min-height: 32px;
    }
    .widget-footer {
      margin-top: 8px;
    }

    .widget-main {
      height: 90px;
      flex-grow: 1;
    }
  }

  .widget-size-lg {
    min-width: calc(100% - 16px) !important;
    // max-width: calc(100% - 16px);
  }

  .widget-size-md {
    min-width: calc(50% - 16px) !important;
    // max-width: calc(50% - 16px);
  }

  .widget-size-sm {
    min-width: calc(25% - 16px) !important;
    max-width: calc(50% - 16px);
  }

  @include media-breakpoint-down(sm) {
    .widget-size-sm {
      min-width: calc(50% - 16px) !important;
      max-width: calc(50% - 16px);
    }
    .widget-size-md,
    .widget-size-lg {
      min-width: calc(100% - 16px) !important;
      max-width: calc(100% - 16px);
    }
  }

  :global(.dashboard-widget .value) {
    display: flex;
    flex-grow: 1;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    color: var(--color-inverse);
    height: 100%;
    padding: 2px 8px;
    .current {
      font-size: 2rem;
      line-height: 100%;
      text-align: center;
    }
    &.value-sm {
      .current {
        font-size: 1.4rem;
      }
      .avg {
        opacity: 0.6;
      }
    }
    &.last-used {
      .current {
        font-size: 1.5rem;
      }
    }
  }
</style>

{#if widget && widget.type !== 'text'}
  <div class="dashboard-widget {getClass(widget)}" {id}>
    <div class="widget-header n-row">
      <TrackerSmallBlock
        xs
        truncate
        novalue
        element={widget.element}
        on:click={() => {
          Interact.openStats(widget.element.toSearchTerm());
        }} />
      <div class="filler" />
      <Button
        size="xs"
        color="clear"
        className="p-1 font-weight-normal"
        on:click={() => {
          DashboardStore.pickSize(widget);
        }}>
        {widget.size}
      </Button>
      <Button
        size="xs"
        color="clear"
        className="p-1"
        on:click={() => {
          dispatch('click');
        }}>
        <Icon name="settings" style="fill: var(--color-inverse-2)" size="16" />
      </Button>
    </div>
    <div class="widget-main">
      {#if widget.stats}
        {#if ['barchart', 'linechart'].indexOf(widget.type) > -1 && widget.stats && widget.stats.chart}
          <WidgetBarChart {widget} />
        {:else if widget.type == 'value' && widget.stats}
          <WidgetValue {widget} />
        {:else if widget.type == 'what-time'}
          <WidgetWhatTime {widget} />
        {:else if widget.type == 'last-used'}
          <WidgetLastUsed {widget} />
        {:else if widget.type == 'positivity' && widget.stats}
          <WidgetPositivityPie {widget} />
        {:else if widget.type == 'min-max' && widget.stats}
          <WidgetMinMax {widget} />
        {:else if widget.type == 'map' && widget.stats}
          <WidgetMap {widget} />
        {:else if widget.type == 'streak' && widget.stats}
          <WidgetStreak {widget} />
        {:else}
          <div class="value">Unknown {widget.type}</div>
        {/if}
      {:else}
        <div class="value n-row">
          <Spinner size={24} />
        </div>
      {/if}
    </div>
    <div class="widget-footer n-row">
      {#if widget.timeRange}
        <Text size="xs" className="text-center flex-grow text-uppercase font-weight-bold">{widget.getLabel()}</Text>
      {/if}
    </div>
  </div>
{:else}
  <div
    {id}
    class="dashboard-text type-text text-center widget-size-{widget.size}"
    on:click={() => {
      dispatch('click');
    }}>
    {widget.description}
  </div>
{/if}
