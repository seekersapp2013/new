<script>
  // components

  import NItem from "../../../components/list-item/list-item.svelte";
  import NModal from "../../../components/modal/modal.svelte";
  import NIcon from "../../../components/icon/icon.svelte";
  import NToggle from "../../../components/toggle-switch/toggle-switch.svelte";
  import NInput from "../../../components/input/input.svelte";
  import ColorPicker from "../../../components/color-picker/color-picker.svelte";
  import AutoComplete from "../../../components/auto-complete/auto-complete.svelte";
  import PickerList from "../../../components/picker-list/picker-list.svelte";

  // Utils
  import { createEventDispatcher } from "svelte";
  import NomieUOM from "../../../utils/nomie-uom/nomie-uom";
  import tick from "../../../utils/tick/tick";

  // modules
  import Tracker from "../../../modules/tracker/tracker";
  import TrackerTypes from "../../../modules/tracker-types/tracker-types";

  // containers
  import PointsEditor from "../points-editor.svelte";

  // Stores
  import config from "../../../config/appConfig";
  import { UserStore } from "../../../store/user-store";
  import { Interact } from "../../../store/interact";
  import { TrackerStore } from "../../../store/tracker-store";
  import { Lang } from "../../../store/lang";
  import { BoardStore } from "../../../store/boards";
  import Text from "../../../components/text/text.svelte";
  import Icon from "../../../components/icon/icon.svelte";
  import Button from "../../../components/button/button.svelte";

  const dispatch = createEventDispatcher();

  export let tracker = new Tracker();
  // export let show = false;

  let data = {
    groupedUOMs: NomieUOM.toGroupedArray(),
    types: Object.keys(TrackerTypes).map((id) => {
      let type = TrackerTypes[id];
      type.id = id;
      return type;
    }),
    editTag: false,
    tracker: null,
  };

  let lastTracker;
  $: if (tracker && lastTracker != tracker.tag) {
    lastTracker = tracker.tag;
    data.tracker = new Tracker(tracker);
  }

  $: if (data.tracker.type === "timer") {
    data.tracker.uom = "timer";
    data.tracker.min = null;
    data.tracker.max = null;
  } else if (data.tracker.uom == "timer" && data.tracker.type != "timer") {
    data.tracker.uom = "num";
  } else if (data.tracker.type === "range" && isNaN(data.tracker.min)) {
    data.tracker.min = 1;
    data.tracker.max = 10;
  }

  const getTrackerInput = async (target) => {
    const response = await Interact.trackerInput(data.tracker, {
      value: data.tracker.default,
      allowSave: false,
    });
    if (response && response.value) {
      data.tracker[target] = response.value;
    }
  };

  async function duplicate() {
    let duplicated = await TrackerStore.duplicateTracker(data.tracker);
  }

  async function remove() {
    let confirmed = await Interact.confirm(
      Lang.t("general.delete-from-nomie", { thing: tracker.label }),
      Lang.t("tracker.delete-description")
    );
    if (confirmed) {
      await TrackerStore.deleteTracker(tracker);
      methods.cancel();
    }
  }

  const methods = {
    tracker_save() {
      if (!data.tracker.tag || !data.tracker.label) {
        Interact.alert("Missing Data", "Please fill out all required fields: title, tag and emoji");
      } else {
        dispatch("save", data.tracker);
        TrackerStore.saveTracker(data.tracker).then(() => {});
      }
    },
    editTag() {
      Interact.alert(
        "Not Supported",
        `Editing Tags is currently not supported. Your best option is to Export your data, and do a search/replace for the tag.`
      );

      // TODO: Make edit tag work. It when saving the tracker we need to know it's original tag and replace it in the TrackerStore. Right not it just adds a new one since the tag is the key.
      // Interact.confirm(
      //   "Change this Tag?",
      //   `If you've tracked with this in the past, use "Settings > Find and Replace" to replace #${data.tracker.tag} with your new tag.`
      // ).then(res => {
      //   if (res === true) {
      //     tracker._dirty = true;
      //   }
      // });
    },
    addTrackerToNote() {
      Interact.selectTrackers().then((trackers) => {
        if (trackers) {
          let trkString = trackers
            .filter((t) => t)
            .map((tkr) => {
              return `#${tkr.tag}`;
            })
            .join(" ");
          data.tracker.note = `${data.tracker.note || ""} ${trkString}`.trim() + " ";
        }
      });
    },
    labelChanged(event) {
      if (data.tracker._dirty) {
        let tag = event.target.value
          .trim()
          .replace(/[^A-Z0-9]/gi, "_")
          .toLowerCase();
        data.tracker.tag = tag;
      }
    },
    cancel() {
      data.tracker = new Tracker();
      dispatch("close");
    },
  };
</script>

<style lang="scss">
  :global(.n-tracker-editor .n-modal) {
    background-color: var(--color-bg) !important;
  }
  .n-tracker-editor {
    .item-divider.compact {
      // background-color: var(--color-solid);
    }
  }
</style>

{#if $Interact.trackerEditor.show}

  <div class="n-tracker-editor">
    <NModal type="fullscreen" allowClose on:close={methods.cancel} style="z-index:2002">

      <header slot="header" class="n-toolbar-grid">
        <button class="left btn btn-clear text-primary-bright" on:click|preventDefault={methods.cancel}>Cancel</button>
        <div class="main">Edit {data.tracker.label.length ? data.tracker.label : 'Tracker'}</div>
        <button class="right btn btn-clear text-primary-bright" on:click={methods.tracker_save}>{Lang.t('general.save')}</button>
      </header>

      <NItem className="item-divider compact" />
      <ColorPicker bind:value={data.tracker.color} />

      <div class="n-list solo p-2">
        <NInput
          className="mb-1"
          type="text"
          name="label"
          placeholder="Tracker Label"
          bind:value={data.tracker.label}
          on:keyup={methods.labelChanged} />

        <NInput
          solo
          type="text"
          name="emoji"
          on:focus={(event) => {
            event.detail.target.select();
          }}
          inputClass="text-lg"
          className="mb-1"
          bind:value={data.tracker.emoji}>
          <div slot="left" class="mr-2 ml-2">Emoji</div>
        </NInput>

        {#if data.tracker._dirty}
          <NInput
            type="text"
            name="tag"
            className="mb-1"
            placeholder={Lang.t('tracker.tag')}
            bind:value={data.tracker.tag}
            autocomplete="off"
            autocorrect="off"
            maxlength="10"
            autocapitalize="off"
            spellcheck="false" />
        {/if}
      </div>

      <div class="n-list solo p-2">

        <NInput type="select" name="type" className="mb-1" placeholder={Lang.t('tracker.type')} bind:value={data.tracker.type}>
          {#each data.types as type}
            <option value={type.id}>{type.label}</option>
          {/each}
        </NInput>

        {#if data.tracker.type == 'picker'}
          <PickerList mode="edit" canSelect={false} bind:tracker={data.tracker} className="px-1" itemClass="px-1" on:change={(evt) => {}} />
        {/if}

        {#if data.tracker.type == 'tick'}
          <NItem
            title={Lang.t('tracker.save-on-tap')}
            className="py-2 px-1"
            description={Lang.t('tracker.save-on-tap-description', 'Automatically save the value when you tap the button.')}>
            <div slot="right">
              <NToggle bind:value={data.tracker.one_tap} />
            </div>
          </NItem>
        {/if}
        {#if data.tracker.type == 'range'}
          <div class="n-row">
            <NInput
              pattern="[0-9]*"
              inputmode="numeric"
              className="mr-2"
              style="width:45%;"
              name="min"
              placeholder={Lang.t('tracker.min', 'Min value in range')}
              on:focus={(e) => {
                e.detail.target.select();
              }}
              bind:value={data.tracker.min}>
              <span slot="left" class="pl-2">
                <Text size="sm" faded>Min</Text>
              </span>
              <button
                class="btn btn-icon clickable mr-2"
                slot="right"
                on:click={() => {
                  getTrackerInput('min');
                }}>
                <NIcon name="addOutline" />
              </button>
            </NInput>
            <NInput
              pattern="[0-9]*"
              inputmode="numeric"
              className=""
              style="width:45%;"
              name="max"
              placeholder={Lang.t('tracker.max', 'Max value in range')}
              on:focus={(e) => e.detail.target.select()}
              bind:value={data.tracker.max}>
              <span slot="left" class="pl-2">
                <Text size="sm" faded>Max</Text>
              </span>
              <button
                class="btn btn-icon clickable mr-2"
                slot="right"
                on:click={() => {
                  getTrackerInput('max');
                }}>
                <NIcon name="addOutline" />
              </button>
            </NInput>
          </div>
        {/if}

        {#if data.tracker.type !== 'timer' && data.tracker.type !== 'note' && data.tracker.type !== 'picker'}
          <NInput placeholder={Lang.t('tracker.measure-by')} type="select" bind:value={data.tracker.uom} className="mb-1">
            {#each Object.keys(data.groupedUOMs) as groupKey (groupKey)}
              <option disabled>-- {groupKey}</option>
              {#each data.groupedUOMs[groupKey] as uom (`${groupKey}-${uom.key}`)}
                <option value={uom.key} disabled={uom.key == 'time' && data.tracker.type != 'timer'}>{NomieUOM.plural(uom.key)}</option>
              {/each}
            {/each}
          </NInput>
        {/if}
        {#if data.tracker.type !== 'note' && data.tracker.type !== 'picker'}
          <NInput
            type="select"
            className="mb-1"
            name="math"
            placeholder={Lang.t('tracker.calculate-total', 'Calculate Totals using:')}
            bind:value={data.tracker.math}>
            {#each [{ value: 'sum', label: Lang.t('general.sum', 'Sum') }, { value: 'mean', label: Lang.t('general.avg', 'Average') }] as math_key}
              <option value={math_key.value}>{math_key.label}</option>
            {/each}
          </NInput>
          <NInput
            pattern="[0-9]*"
            inputmode="numeric"
            label={Lang.t('tracker.value', 'Default Value')}
            placeholder={Lang.t('tracker.default-value', 'Default Value')}
            bind:value={data.tracker.default}
            className="mb-1">
            <span slot="right">
              {#if data.tracker.default}
                <Text size="xs" className="text-right text-primary-bright">{data.tracker.displayValue(data.tracker.default)}</Text>
              {/if}
            </span>
            <button
              class="btn btn-icon clickable mr-2"
              slot="right"
              on:click={() => {
                getTrackerInput('default');
              }}>
              <NIcon name="addOutline" />
            </button>
          </NInput>

          {#if data.tracker.math !== 'sum'}
            <NItem className="px-1 py-1" title="Ignore Zeros" description="Ignore zero values when averaging">
              <div slot="right">
                <NToggle bind:value={data.tracker.ignore_zeros} />
              </div>
            </NItem>
          {/if}
        {:else if data.tracker.type == 'note'}
          <NInput
            type="textarea"
            bind:value={data.tracker.note}
            placeholder={Lang.t('tracker.note-placeholder')}
            class="form-control w-100 mt-2">
            <span slot="right">
              <Button size="sm" icon shape="circle" color="transparent" on:click={methods.addTrackerToNote}>
                <Icon name="addOutline" />
              </Button>
            </span>
          </NInput>

          <AutoComplete
            input={data.tracker.note}
            scroller
            on:select={async (evt) => {
              data.tracker.note = evt.detail.note + '';
            }} />
          <NItem description={Lang.t('tracker.note-description')} />
        {/if}
      </div>

      <div class="n-list solo py-1 px-2">
        <PointsEditor tracker={data.tracker} />
      </div>

      {#if data.tracker.type !== 'note' && data.tracker.type !== 'picker'}
        <div class="n-list solo p-2">
          <NInput
            type="textarea"
            rows="2"
            label={Lang.t('tracker.include', 'Additional trackers or People to include')}
            placeholder={Lang.t('tracker.include-placeholder', 'Include other #trackers, @People, +context')}
            bind:value={data.tracker.include} />
          <AutoComplete
            input={data.tracker.include}
            scroller
            on:select={async (evt) => {
              data.tracker.include = evt.detail.note + '';
            }} />
          <Text size="xs" faded className="my-2">
            Automatically insert #trackers, @People, and +context when using this tracker. Pass the value using: #hashtag(*).
          </Text>
        </div>
      {/if}

      <div class="p-2" />
      <NItem bg="transparent" title={Lang.t('tracker.hide-on-all-board', 'Hide on All Board')}>
        <div slot="right">
          <NToggle bind:value={data.tracker.hidden} />
        </div>
      </NItem>
      <div class="p-2" />
      <NItem on:click={TrackerStore.download(data.tracker)} className="bottom-line">
        <div class="text-primary-bright">{Lang.t('general.download', 'Download')} .tkr</div>
        <div slot="right" class="text-faded-2">For Sharing</div>
      </NItem>
      <NItem on:click={duplicate} className="bottom-line">
        <div class="text-primary-bright">{Lang.t('tracker.duplicate-tracker', 'Duplicate Tracker')}</div>
      </NItem>

      <NItem on:click={remove} className="bottom-line">
        <div class="text-red">{Lang.t('tracker.remove-tracker', 'Delete Tracker')}</div>
      </NItem>

      <!-- <button slot="footer" on:click={methods.cancel} class="btn btn-light btn-lg flex-grow mr-1">{Lang.t('general.cancel')}</button>
      <button slot="footer" class="btn btn-primary btn-lg flex-grow ml-1" on:click={methods.tracker_save}>{Lang.t('general.save')}</button> -->
      <div slot="footer" />
    </NModal>

  </div>
{/if}
