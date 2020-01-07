<script>
    import { jsonData } from "./stores.js";    
    import BbRubrics from './BbRubrics.svelte';
    let ok;
    let bbFile = {};
    const errorMessage = "I'm sorry, that file doesn't look right. ";
    export let message = "Please choose a file";
    function onchange() {
      const file = this.files[0];
      if (file !== undefined) {
        const fileReader = new FileReader();
        if (file.type !== 'application/x-zip-compressed') {
          // Not a Blackboard export file, try TurnItIn format
          fileReader.readAsText(file);
          fileReader.onloadend = function() {
            try {
              jsonData.set(JSON.parse(fileReader.result));
              ok = true;
            } catch (e) {
              jsonData.set({});
              ok = false;
            }
          };
        } else {
          // Update Blackboard rubric convertor/chooser
          bbFile = file;
        }
      } else {
        jsonData.set({});
        ok = false;
      }
    }
</script>

<style>
    input {
      width: 100%;
      padding: 30px;
      background-color: cornsilk;
      border: 2px dashed peru;
    }
    h2 {
      margin: 0;
      padding: 20px 0;
    }
    .error {
      background-color: pink;
      border-color: darksalmon;
    }
    iframe {
      display: block;
      width: 660px;
      height: 540px;
      margin-left: auto;
      margin-right: auto;
      margin-top: 20px;
    }
    p {
      text-align: center;
    }
</style>
<h2>{ok === false ? errorMessage : ''}{message}</h2>
<input type="file" on:change={onchange} class={ok === false ? 'error' : ''} />
<BbRubrics file={bbFile} />

{#if !ok}
<p>How to get a .rbc file from TurnItIn</p>
<iframe src="https://www.youtube.com/embed/gxqLRZq-0p4" title="Video: how to get your .rbc files" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
{/if}