<script>
  // Svelte
  import { Router, Route, navigate } from "svelte-routing";
  import { onMount } from "svelte";
  import dayjs from "dayjs";

  // Vendors
  import Spinner from "./components/spinner/spinner.svelte";
  import * as ComposiGestures from "@composi/gestures";

  // Containers
  import Interactions from "./containers/interactions/interactions.svelte";
  import LibraryModal from "./containers/library/library.svelte";
  import PersonModal from "./containers/People/person-modal.svelte";
  // import Modal from "./components/modal/modal.svelte";
  import StatsModal from "./containers/stats/stats-modal.svelte";
  import StreakModal from "./containers/steak/streak-modal.svelte";
  import WhatsNewModal from "./containers/whats-new/whats-new-modal.svelte";
  import OnThisDayModal from "./containers/on-this-day/on-this-day.svelte";

  import SetupRoute from "./routes/setup.svelte";

  // Utils
  import Logger from "./utils/log/log";
  import Timer from "./utils/timer/timer";

  import RouterView from "./routes/routes.svelte";

  // Stores
  import { UserStore } from "./store/user-store"; //  user auth and state
  import { Interact } from "./store/interact"; //  global alerts, popmenus, confirms, etc
  import { BoardStore } from "./store/boards"; // board state  and methods
  import { Device } from "./store/device-store"; // board state  and methods
  import { TrackerStore } from "./store/tracker-store"; // tracker state and methods
  import { TrackerLibrary } from "./store/tracker-library";
  import { CommanderStore } from "./store/commander"; // commander - /?note=hi&lat=35&lng=-81.32
  import { NomieAPI } from "./store/napi"; // Store for interacting with the Nomie API
  import { PeopleStore } from "./store/People-store"; // Store for holding People
  import { ContextStore } from "./store/context-store"; // Store for holding Post Context (categories)
  import { DashboardStore } from "./store/dashboard-store"; // Store for holding Post Context (categories)
  import { AppStore } from "./store/app-store";
  import { Locations } from "./store/locations";
  import config from "./config/appConfig";
  import { OfflineQueue } from "./store/offline-queue-store";
  // import Storage from "./containers/storage/storage.svelte";
  import Storage from "./modules/storage/storage";
  import { LastUsed } from "./store/last-used";
  import { SearchStore } from "./store/search-store";
  import PinLock from "./containers/pin-lock/pin-lock.svelte";
  import tick from "./utils/tick/tick";

  // Set a better console
  const console = new Logger("APP");
  const timer = new Timer("App.svelte", false);

  timer.start();
  ComposiGestures.default.gestures();

  /**
   * Day / Time Change Monitoring
   * Fire off the MinuteChecker30 every 30 minutes
   * This will check if the day changed
   */
  let todayCheckPeriod = 1000 * 60 * 10;
  let todayCheckFormat = "YYYY-MM-DD";
  let todayKey = dayjs().format(todayCheckFormat);
  let newDay = false; // View reacts to this value

  // Check every X minutes
  const todayCheckInteval = setInterval(() => {
    // Get now key
    let checkKey = dayjs().format(todayCheckFormat);
    // Compare now key to today key
    if (todayKey !== checkKey) {
      // It's new - trigger some reactions
      newDay = true;
      // Show toast notification
      Interact.toast(`It's ${dayjs().format("dddd")}!`);
      // Set today key to check key
      todayKey = checkKey;
      // Wait 500 ms
      setTimeout(() => {
        newDay = false;
      }, 500);
    }
    // Check if the theme has Changed
    methods.setDocParams();
  }, todayCheckPeriod);

  const appVersion = "APP_VERSION";

  // This should be reworked
  $: if (window && $TrackerStore && !window.$TrackerStore) {
    window.$TrackerStore = $TrackerStore;
  }

  // Offline monitor

  const methods = {
    hideSplashScreen() {
      timer.check("Hide Splashscreen");
      document.querySelectorAll(".delete-on-app").forEach((d) => {
        d.classList.add("deleted");
        setTimeout(() => {
          d.remove();
        }, 500);
      });
    },
    setDocParams(options) {
      let isDarkMode = window.matchMedia("(prefers-color-scheme: dark)").matches;
      // let isDarkMode = false;
      let theme = localStorage.getItem(config.theme_key) || "auto";
      let theme_accent = localStorage.getItem(`${config.theme_key}-accent`) || "default";
      document.body.className = "";
      if (theme === "auto" && isDarkMode) {
        document.body.classList.add("theme-dark");
      } else if (theme === "auto") {
        document.body.classList.add("theme-light");
      } else {
        document.body.classList.add(`theme-${theme}`);
      }
      document.body.classList.add(theme_accent);
      tick(100, methods.hideSplashScreen);
    },
  };

  /**
   * App to Forground
   *
   * Document Change State Monitoring
   * In hopes of triggering events when the
   * state of the window changes be it from
   * the browser, or switching apps on a phone
   *
   * it kinda works.
   */
  let hidden, visibilityChange, router;
  if (typeof document.hidden !== "undefined") {
    hidden = "hidden";
    visibilityChange = "visibilitychange";
  } else if (typeof document.msHidden !== "undefined") {
    hidden = "msHidden";
    visibilityChange = "msvisibilitychange";
  } else if (typeof document.webkitHidden !== "undefined") {
    hidden = "webkitHidden";
    visibilityChange = "webkitvisibilitychange";
  }

  window.addEventListener("load", () => {
    methods.setDocParams();
  });

  let ready = false;

  // Used to make sure that boards and trackers are loaded
  UserStore.onReady(async () => {
    timer.check("UserStore.onReady fired");
    console.log("🌳🌳🌳🌳🌳🌳🌳🌳🌳🐘🌳🌳🌳🌳🌳🌳🌳🌳🌳");
    console.log("🌳🌳🌳🌳🌳🌳 Welcome to 🌳🌳🌳🌳🌳🌳🌳🌳");
    console.log("🌳🌳🌳🌳🌳🌳 GRADER APP_VERSION 🌳🌳🌳🌳🌳🌳🌳");
    console.log("🌳🌳🌳🌳🌳🌳🌳🌳🌳🐘🌳🌳🌳🌳🌳🌳🌳🌳🌳🌳");
    // Set the user if they're logged in
    ready = true;
    timer.check("Starting Store Intialization sync");
    PeopleStore.init(); // Initialize the People Store
    Locations.init(); // Initialize Location Store
    ContextStore.init(); // check if this is a new version
    DashboardStore.init(); // Initilize Dashboards
    Device.init(); // Initialize Device
    LastUsed.init();
    SearchStore.init();

    // Run any commands if needed
    setTimeout(() => {
      // Pull upp the offline queue
      OfflineQueue.init();
      // If there are any URL caommands, it will run here.
      CommanderStore.run();
      // If they have the API - it will load here
      NomieAPI.load();
    }, 500);
  });

  onMount(() => {
    timer.check("onMount");
    UserStore.initialize();
  });
</script>

{#if !ready && $UserStore.signedIn === false}
  <SetupRoute />
{/if}

<!-- {#if ready} -->
{#if $UserStore.signedIn === true && !newDay}
  <RouterView />
  <WhatsNewModal />
{:else if $UserStore.signedIn == undefined || newDay}
  <div class="empty-notice" style="height:calc(100vh - 75px)">
    <Spinner />
  </div>
{/if}

<!-- Global Modals, alerts, menus, etc-->
{#if ready && $Interact.stats.terms.length}
  <StatsModal />
{/if}
{#if ready && $TrackerLibrary.show}
  <LibraryModal />
{/if}
{#if ready && $Interact.People.active}
  <PersonModal />
{/if}
{#if $Interact.blocker.show}
  <div id="ui-blocker" class="full-screen bg-translucent n-panel center-all">
    <Spinner size="16" />
    <span class="text-white ml-2">{$Interact.blocker.message}</span>
  </div>
{/if}
<Interactions />
<StreakModal />
<OnThisDayModal />
<PinLock />

{#if $UserStore.storageType == 'blockstack' && $Device.offline}
  <div class="offline-notice text-center">No connection to Blockstack.</div>
{/if}
<div id="photo-holder">
  <img id="photo-holder-image" alt="avatar-holder" />
</div>
<!-- {/if} -->
