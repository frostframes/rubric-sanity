<script>
    import { jsonData } from "./stores.js";    
    import JSZip from 'jszip/dist/jszip.js';
    export let file;

    let zipObject = {};
    let parser = new DOMParser();
    let xmlDoc;
    let rubrics = [];
    $: onChange(file);

    function onChange(file) {
        JSZip.loadAsync(file).
        then(function (zip) {
            // Use the manifest to find which file contains the rubric data
            zipObject = zip;
            return zipObject.file("imsmanifest.xml").async("text");
        }).then(function (txt) {
            xmlDoc = parser.parseFromString(txt, "text/xml");
            let resources = xmlDoc.firstChild.lastChild.children;
            let rubricFilename = '';
            for (let resource of resources) {
                if (resource.attributes['bb:title'].nodeValue === 'LearnRubrics') {
                rubricFilename = resource.attributes['bb:file'].nodeValue;
                break;
                }
            }
            // Load the list of rubrics
            return zipObject.file(rubricFilename).async("text");
        }).then(function (txt) {
            xmlDoc = parser.parseFromString(txt, "text/xml");
            let rubricNodes = xmlDoc.firstChild.children;
            for (let rubric of rubricNodes) {
                let title = '';
                let description = '';
                let id = '';
                for (let child of rubric.children) {
                    if (child.nodeName === "Title") {
                        title = child.attributes.value;
                    } else if (child.nodeName === "Description") {
                        description = child.attributes.value;
                    }
                }
                rubrics.push({
                    'title': title.value,
                    'description': description.value,
                    'id': rubric.id
                });
            }
            // Force rendering the list
            rubrics = rubrics;
        }, function (e) {
            jsonData.set({});
        });
    }
</script>
{#if rubrics.length > 0}
<section>
<p>Please select one of the rubric contained within the uploaded file</p>
<ul>
    {#each rubrics as rubric}
    <li><label><input type="radio" name="rubric" value="{rubric.id}">{rubric.title}</label>
        <br /><span>{rubric.description}</span></li>
    {/each}
</ul>
</section>
{/if}

<style>
    section {
        padding: 0;
    }
    li {
        padding: 0.5em;
        border-bottom: 1px solid #9cf;
        list-style-type: none;
        background-color: #def;
        margin: 0;
    }
    ul {
        padding: 0;
        margin: 0;
    }
    p {
        background-color: #e6d5b4;
        padding: 0.5em;
        margin: 0;
    }
    label {
       color: #48c;
       font-weight: bold;
    }
    span {
        display: block;
        padding-left: 1.4em;
    }
</style>