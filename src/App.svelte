<script>
	import { jsonData } from "./stores.js";
	import Upload from "./Upload.svelte";
	import RubricView from "./RubricView.svelte";
</script>

<style>
  section {
    font-family: "Roboto", "Helvetica Neue", Helvetica, Arial, sans-serif;
    min-height: 100rem;
  }
</style>

<section>
  <Upload message="Please choose an .rbc file exported from TurnItIn." />
  <RubricView data={$jsonData} />
</section>