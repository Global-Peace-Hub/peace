<script>
  import { Router, Route } from "svelte5-router";
  import SubPage from "./routes/SubPage.svelte";
  import Home from "./routes/Home.svelte";
</script>

<Router>
  <Route basePath="/mend/" component={Home} />
  <Route path="/mend/:dynamicPath" component={SubPage} />
</Router>