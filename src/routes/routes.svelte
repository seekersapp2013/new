<script>
  import Dashboard from "./dashboard.svelte";
  import { Router, Route, navigate } from "svelte-routing";

  import TrackRoute from "./track.svelte";
  import HistoryRoute from "./history.svelte";
  import SearchRoute from "./search.svelte";
  import SettingsRoute from "./settings.svelte";
  import BoardEditorRoute from "./board-editor.svelte";
  import FAQRoute from "./faq.svelte";
  import NomieAPIRoute from "./nomie-api.svelte";
  import ExportRoute from "./export.svelte";
  import PeopleRoute from "./People.svelte";
  import TrackerDesigner from "./tracker-designer.svelte";
  import Setup from "./setup.svelte";

  import FileBrowser from "./file-browser.svelte";
  import CsvImport from "./csv-import.svelte";
</script>

<Router>
  <Route path="/" component={TrackRoute} />
  <Route path="/history" component={HistoryRoute} />
  <Route path="/history/:date" component={HistoryRoute} />
  <Route path="/dashboard" component={Dashboard} />
  <Route path="/People" component={PeopleRoute} />
  <Route path="/search" component={SearchRoute} />
  <Route path="/settings" component={SettingsRoute} />
  <Route path="/board/:id" component={BoardEditorRoute} />
  <Route path="/faq" component={FAQRoute} />
  <Route path="/api" component={NomieAPIRoute} />
  <Route path="/settings/export/:type" component={ExportRoute} />
  <Route path="/settings/import/csv" component={CsvImport} />
  <Route path="/settings/export" component={ExportRoute} />
  <Route path="/tracker/design" component={TrackerDesigner} />
  <Route path="/files/*path" let:params>
    <FileBrowser path={params.path} />
  </Route>
  <Route path="/setup" component={Setup} />
</Router>
