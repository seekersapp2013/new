<script lang="ts">
  import RadioButtonChecked24 from "carbon-icons-svelte/lib/RadioButtonChecked24";
  import Shuffle24 from "carbon-icons-svelte/lib/Shuffle24";
  import RadioButton24 from "carbon-icons-svelte/lib/RadioButton24";
  // Hero Icons
  import {
    ArrowCircleUp,
    ArrowLeft,
    ArrowUp,
    ArrowsExpand,
    Calendar,
    ChartBar,
    Check,
    CheckCircle,
    ChevronDown,
    ChevronLeft,
    ChevronRight,
    ChevronUp,
    ClipboardCopy,
    Clock,
    Cog,
    DotsHorizontal,
    Download,
    FolderAdd,
    Link,
    LocationMarker,
    Menu,
    MinusCircle,
    PaperAirplane,
    Pencil,
    Plus,
    PlusCircle,
    PresentationChartLine,
    Refresh,
    Search,
    Share,
    SortAscending,
    Star,
    SwitchHorizontal,
    Trash,
    User,
    UserAdd,
    UserGroup,
    ViewBoards,
    ViewGrid,
    X,
    XCircle,
    DotsVertical,
    ChartPie,
  } from "svelte-hero-icons";

  export let name = "add";
  export let style = undefined;
  export let className = "";
  export let title = "";
  export let size = "";

  const icons = {
    add: Plus,
    addOutline: PlusCircle,
    arrowBack: ArrowLeft,
    arrowUp: ArrowUp,
    arrowsLeftRight: SwitchHorizontal,
    calendar: Calendar,
    chart: ChartBar,
    checkmark: Check,
    checkmarkFilled: CheckCircle,
    checkmarkOutline: CheckCircle,
    chevronDown: ChevronDown,
    chevronLeft: ChevronLeft,
    chevronRight: ChevronRight,
    chevronUp: ChevronUp,
    close: X,
    closeFilled: XCircle,
    closeOutline: XCircle,
    copy: ClipboardCopy,
    dashboard: ViewBoards,
    delete: Trash,
    download: Download,
    edit: Pencil,
    pieChart: ChartPie,
    grid: ViewGrid,
    link: Link,
    menu: Menu,
    more: DotsHorizontal,
    moreVertical: DotsVertical,
    newTab: FolderAdd,
    People: UserGroup,
    pin: LocationMarker,
    radio: RadioButton24,
    radioFilled: RadioButtonChecked24,
    refresh: Refresh,
    remove: MinusCircle,
    report: PresentationChartLine,
    search: Search,
    sendFilled: ArrowCircleUp,
    airplane: PaperAirplane,
    settings: Cog,
    share: Share,
    shuffle: Shuffle24,
    sort: SortAscending,
    star: Star,
    time: Clock,
    tracker: ViewGrid,
    userAdd: UserAdd,
    user: User,
  };
</script>

<style>
  .n-icon {
    display: inline-flex;
  }
</style>

{#if icons[name]}
  <span class="n-icon n-icon-{name} {className}" {style} aria-label={title}>
    <svelte:component this={icons[name]} size={`${size || 24}`} />
  </span>
{/if}
