<!-- Parent component (e.g., App.svelte) -->
<script>
  import InaturalistQuery from "./lib/InaturalistQuery.svelte";
  let baseUrl = "https://api.inaturalist.org/v1/";
</script>

<main>
  <InaturalistQuery {baseUrl} />
</main>
