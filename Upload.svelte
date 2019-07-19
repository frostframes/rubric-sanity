<script>
    import { jsonData } from "./stores.js";
    let ok = true;
    const errorMessage = "I'm sorry, that file doesn't look right. ";
    export let message = "Please choose a file";
    function onchange() {
      const file = this.files[0];
      if (file !== undefined) {
        const fileReader = new FileReader();
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
      padding: 20px;
    }
    .error {
      background-color: pink;
      border-color: darksalmon;
    }
</style>
<h2>{ok ? '' : errorMessage}{message}</h2>
<input type="file" on:change={onchange} class={ok ? '' : 'error'} />