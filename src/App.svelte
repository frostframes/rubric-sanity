<script>
	import { jsonData } from "./stores.js";
	import Upload from "./Upload.svelte";
	import RubricView from "./RubricView.svelte";
</script>

<header>
<h1>Rubric sanity - preview and convert to Word</h1>
<p><a href="mailto:c.frost1@uq.edu.au">Email Chris</a> if you want this app to do more.</p>
</header>

<section>
  <Upload message="Please choose an .rbc file exported from TurnItIn or .zip from Blackboard." />
  <RubricView data={$jsonData} />
</section>

<footer>
<p>Source code at <a href="https://github.com/frostframes/rubric-sanity" target="source">github.com/frostframes/rubric-sanity</a></p>
</footer>

<style>
header {
  background-color: #48c;
  margin: 0;
  padding: 30px 30px 20px 30px;
  box-shadow: 0 5px 5px 0 rgba(0, 0, 0, .3);
  border-bottom: #249 solid 1px;
  color: white;
}

header a {
  color: white;
  text-decoration: none;
  border-bottom: 1px solid #8cf
}

section {
    min-height: 100rem;
    padding: 20px;
}

footer {
  text-align: center;
}
</style>