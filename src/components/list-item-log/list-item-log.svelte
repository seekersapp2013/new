<script lang="ts">
  // Svelte
  import { createEventDispatcher, onMount } from "svelte";

  // Modules
  import NLog from "../../modules/nomie-log/nomie-log";
  import Tracker from "../../modules/tracker/tracker";

  // components
  import NItem from "../list-item/list-item.svelte";
  import NBall from "../tracker-ball/ball.svelte";
  import Dymoji from "../dymoji/dymoji.svelte";
  import LocationBadge from "../location-badge/location-badge.svelte";
  import NIcon from "../icon/icon.svelte";
  import NPoints from "../points/points.svelte";
  import NText from "../text/text.svelte";
  import NNoteTextualizer from "../note-textualizer/note-textualizer.svelte";

  import NTrackerSmallBlock from "../tracker-ball/tracker-small-block.svelte";

  // utils
  import NomieUOM from "../../utils/nomie-uom/nomie-uom";
  import time from "../../utils/time/time";

  import { TrackerStore } from "../../store/tracker-store";
  import { UserStore } from "../../store/user-store";
  import { Interact } from "../../store/interact";
  import { PeopleStore } from "../../store/People-store";

  // vendors
  import dayjs from "dayjs";
  import Text from "../text/text.svelte";
  import TrackableElement from "../../modules/trackable-element/trackable-element";
  import { getEmojiFromScore } from "../../utils/positivity/positivity";
  import Button from "../button/button.svelte";

  // props
  export let log = undefined;
  // export let trackers = {};
  export let className = "";
  export let focus = false;
  export let fullDate = false;
  export let hideMore = undefined;
  export let moreOveride: boolean = false;
  export let hideDelete: boolean = false;

  // consts
  const dispatch = createEventDispatcher();

  let displayLog: NLog;
  let logMeta: any;

  let trackers = $TrackerStore.trackers;

  let state = {
    showPhoto: false,
  };

  $: if (log && log !== displayLog) {
    displayLog = new NLog(log);
    logMeta = displayLog.getMeta();
    logMeta.trackers = logMeta.trackers.map((trackerElement) => {
      trackerElement.obj = TrackerStore.getByTag(trackerElement.id);
      return trackerElement;
    });
  }

  let dtFormat;

  $: if ($UserStore.meta.is24Hour) {
    dtFormat = {
      date: "ddd Do MMM YYYY",
      time: "H:mm",
    };
  } else {
    dtFormat = {
      date: "ddd MMM Do YYYY",
      time: "h:mma",
    };
  }

  function shouldShowValue(trackerElement) {
    if (trackerElement.obj.type == "picker") {
      return false;
    } else if (trackerElement.obj.type == "tick") {
      return trackerElement.value !== 1;
    } else {
      return true;
    }
  }
</script>

<style lang="scss">
  .divider {
    font-size: 0.9rem;
    font-weight: 500;
  }

  .n-row.context {
    justify-content: flex-start;
    flex-wrap: wrap;
  }
  .more-button {
    margin-right: -8pt;
  }
</style>

<!--glow glow-{time.dateToDesc(displayLog.end)}-->
{#if displayLog}
  <NItem className="{className} n-item-log">
    <!-- Show the Trackers within this Log Item -->
    <div class="n-row time-row">
      <div class="time truncate">
        <div class="filler n-row justify-content-start">
          {#if displayLog.lat}
            <LocationBadge
              location={displayLog}
              on:click={(event) => {
                Interact.showLocations([displayLog]);
                event.stopPropagation();
              }} />
          {/if}
          <Text inline size="sm" className="" faded>{time.fromNow(logMeta.endDate)}</Text>
        </div>
        <Text size="sm" medium className="filler">
          {logMeta.endDate.format(`${dtFormat.time}`)}
          {#if fullDate}
            <Text inline size="sm" faded>{logMeta.endDate.format(`${dtFormat.date}`)}</Text>
          {/if}
        </Text>
      </div>
      <div class="filler" />
      <!-- If they have location-->

      <!-- SCORE display -->
      {#if displayLog.score}
        <div class="score-mark {displayLog.score > 0 ? 'positive' : 'negative'}">{getEmojiFromScore(displayLog.score, true)}</div>
      {/if}

      {#if hideMore !== true}
        <Button
          shape="circle"
          color="transparent"
          on:click={(event) => {
            if (moreOveride) {
              dispatch('more', displayLog);
            } else {
              Interact.logOptions(displayLog, { hideDelete });
            }
          }}
          className="ml-2"
          style="margin-right:-10px;">
          <NIcon name="more" className="fill-primary-bright" size="32" />
        </Button>
      {/if}

    </div>
    <!-- Process the Note Content wi th the Textualizer 
    This really isn't special right now -->
    {#if displayLog.note.length}
      <NNoteTextualizer
        on:textClick={(evt) => {
          dispatch('textClick', evt.detail);
        }}
        bind:note={displayLog.note}
        {trackers}
        className={logMeta.trackers.length ? '' : 'pb-2'} />
    {/if}

    {#if logMeta.trackers.length || logMeta.People.length}
      <div class="tracker-grid n-row">
        {#each displayLog.People as person}
          <NTrackerSmallBlock
            truncate={true}
            element={person}
            on:click={() => {
              Interact.openStats(`@${person.id}`);
              dispatch('personClick', { person: person, log });
            }}>
            <span slot="emoji" class="emoji">
              {#if $PeopleStore.People[person.id]}
                <NBall size={40} radius={0.3} avatar={$PeopleStore.People[person.id].avatar} username={person.id} className="ml-2" />
              {:else}
                <NBall size={40} username={person.id} className="ml-2" radius={0.3} />
              {/if}
            </span>
          </NTrackerSmallBlock>
        {/each}
        {#each logMeta.trackers.filter((trk) => {
          if (focus) {
            return trk.id == focus;
          } else {
            return true;
          }
        }) as trackerElement}
          <NTrackerSmallBlock
            truncate
            element={trackerElement}
            on:click={() => {
              Interact.openStats(`#${trackerElement.id}`);
              dispatch('trackerClick', { tracker: trackerElement.obj, log });
            }} />
        {/each}
      </div>
    {/if}
    {#if logMeta.context.length}
      <div class="context n-row">
        {#each logMeta.context as context}
          <button
            class="btn btn-badge btn-xs faded"
            on:click={() => {
              Interact.openStats(context.raw);
              dispatch('contextClick', { context: context, log });
            }}>
            +{context.id}
          </button>
        {/each}
      </div>
    {/if}

  </NItem>
{/if}
