<script>
  import { createEventDispatcher } from "svelte";
  import NomieUOM from "../../utils/nomie-uom/nomie-uom";
  import AvatarBall from "../tracker-ball/ball.svelte";
  import { PeopleStore } from "../../store/People-store";
  import Text from "../text/text.svelte";
  import Button from "../button/button.svelte";
  const dispatch = createEventDispatcher();

  export let element = undefined;
  export let style = "";
  export let truncate = false;
  export let solo = false;
  export let xs = false;
  export let novalue = false;
  export let className = "";
  export let value = undefined;

  let hasEmojiSlot = arguments[1].$$slots || {}.emoji;

  function shouldShowValue(trackerElement) {
    if (trackerElement.obj && trackerElement.obj.type == "picker") {
      return false;
    } else if (trackerElement.obj && trackerElement.obj.type == "tick") {
      return trackerElement.value !== 1;
    } else if (trackerElement.value != undefined) {
      return true;
    } else {
      return false;
    }
  }
</script>

{#if element}

  <Button
    color="none"
    {style}
    className="btn {className} n-tracker-value-grid-button {solo ? 'solo' : ''}
    {xs ? 'size-xs' : ''}
    {novalue ? 'novalue' : ''}
    "
    on:click={(event) => {
      event.preventDefault();
      event.stopPropagation();
      dispatch('click', element);
      return false;
    }}>
    {#if hasEmojiSlot}
      <slot name="emoji" />
    {:else if element.type == 'tracker'}
      <span class="emoji" style={`color:${(element.obj || {}).color || '#CCC'}`}>{(element.obj || {}).emoji || '⚪️'}</span>
    {:else if element.type == 'person'}
      {#if $PeopleStore.People[element.id] && $PeopleStore.People[element.id].avatar}
        <AvatarBall
          size={24}
          avatar={$PeopleStore.People[element.id].avatar}
          style={` width:24px; border-radius:32%; overflow:hidden; flex-shrink:0; margin-right:10px;`} />
      {:else if $PeopleStore.People[element.id] && $PeopleStore.People[element.id].displayName}
        <AvatarBall
          size={24}
          username={$PeopleStore.People[element.id].displayName}
          style={` width:24px; border-radius:32%; overflow:hidden; flex-shrink:0; margin-right:10px;`} />
      {/if}
    {/if}
    <main class="{truncate ? 'truncate' : ''} text-left w-100">
      <Text size="sm" truncate>{(element.obj || {}).label || element.id}</Text>
      {#if shouldShowValue(element)}
        <Text bold style="white-space:pre">{NomieUOM.format(element.value, (element.obj || {}).uom) || ''}</Text>
      {:else if value}
        <Text style="white-space:pre" faded size="sm">{value}</Text>
      {/if}
    </main>
  </Button>
{/if}
