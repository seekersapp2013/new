# TODO

Generated Sat Mar 7 2020 1:41 pm

- **src/store/user.js**
   - TODO: Look at push notifications in the browser
   - TODO: Add 10 minute interval to check for day change - if change, fire a new user.ready
   - TODO: move this to modules/storage
- **src/store/trackers.js**
   - TODO: Move this to BoardStore
- **src/store/ledger.js**
   - TODO: this doesn't seem to be trigger a change in History.svetle
   - TODO: Make this use listBooks() array to only look for books that exist
- **src/store/boards.js**
   - TODO: figure out why duplicates happen on board saving
   - TODO: tried to make this a spread - but I keep breaking it.
- **src/scss/main.scss**
   - TODO: Organize main into components or containers.
- **src/scss/components/_list-item.scss**
   - TODO: see if making main overflow hidden was a bad idea
- **src/routes/stats.svelte**
   - TODO: Needs to be refactored and cleaned up
- **src/routes/stats-og.svelte**
   - TODO: Needs to be refactored and cleaned up
- **src/routes/settings.svelte**
   - TODO: figure out how to handle a cancel in the interact prompt
- **src/routes/history.svelte**
   - TODO: Have it react when the ledger change, not a hard refresh
   - TODO: refactor so it's clean and using the proper amount of Store vs local
   - TODO: This is really sloppy - clean it up.
- **src/plugins/goals/goals.svelte**
   - TODO: Finish Goals
- **src/modules/stats/stats.js**
   - TODO: Implement IgnoreZeros
- **src/modules/export/csv.js**
   - end TODO: See why end date is not working in query
- **src/containers/tracker/input/input.svelte**
   - TODO: Figure out wtf is going on with value and data.value - can we just use value?
- **src/containers/tracker/editor/editor.svelte**
   - TODO: Make edit tag work. It when saving the tracker we need to know it's original tag and replace it in the TrackerStore. Right not it just adds a new one since the tag is the key.
- **src/containers/setup/setup.svelte**
   - TODO: Make this design not suck! It's very boring.
   - TODO: UserSession shouldn't be in here - login should be fired by Storage.
- **src/containers/setup/setup copy.svelte**
   - TODO: Make this design not suck! It's very boring.
   - TODO: UserSession shouldn't be in here - login should be fired by Storage.
- **src/containers/map/map.svelte**
   - TODO: Look at making this curved dotted lines - and not just straight ones
- **src/containers/importer/importer.svelte**
   - Modal will be hidden in settings TODO: make this not hacky -->
- **src/containers/board/tracker-button.svelte**
   - TODO: Move this to components/tracker-button
- **src/components/capture-log.svelte**
   - TODO: Add a media/photo type of thing that can be added to a log..
   - await LedgerStore.saveLog($ActiveLogStore);  TODO: Make ledger task instead
- **src/App.svelte**
   - TODO: this is not working - why?