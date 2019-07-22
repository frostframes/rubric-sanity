<script>
	import { jsonData } from "./stores.js";
	import Upload from "./Upload.svelte";
	import RubricView from "./RubricView.svelte";
	import RubricToWord from "./RubricToWord.svelte";
</script>

<style>
  body {
    font-family: "Roboto", "Helvetica Neue", Helvetica, Arial, sans-serif;
  }
</style>
<body lang="en">
<Upload message="Please choose an .rbc file exported from TurnItIn." />
<RubricView data={$jsonData} />
<RubricToWord data={$jsonData} />
</body>