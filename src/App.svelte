<script>
  import { Router, Route } from "svelte5-router";
  import SubPage from "./routes/SubPage.svelte";
  import Home from "./routes/Home.svelte";
</script>

<Router>
  <Route basePath="/" component={Home} />
  <Route path="/:dynamicPath" component={SubPage} />
</Router>