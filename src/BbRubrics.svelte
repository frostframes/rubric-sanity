<script>
    import { jsonData } from "./stores.js";    
    import JSZip from 'jszip/dist/jszip.js';
    export let file;

    const disclaimerPercent = 'Percentages have been converted to points to allow conversion to TurnItIn. Blackboard rubrics store percentages per cell not per column, unlike TurnItIn. ';
    const disclaimerRanges = 'Only the maximum values of ranges are shown to allow convervion to TurnItIn.';
    const prompt = 'Please choose one of the rubrics contained within the uploaded file.'; 
    let message = '';
    let parser = new DOMParser();
    let rubrics = [];
    let rubricId = '';
    let rubricJson = {}
    let rubricJsonTemplate = `{
        "Rubric": [
            {
                "criterion": [],
                "name": "",
                "scoring_method": 0
            }
        ],
        "RubricScale": [],
        "RubricCriterion": [],
        "RubricCriterionScale": []
    }`;
    let rubricsXml;
    let xmlDoc;
    let zipObject = {};

    $: onLoad(file);
    $: onChange(rubricId);

    function onLoad(file) {
        rubrics = [];
        JSZip.loadAsync(file).
        then(function (zip) {
            // Use the manifest to find which file contains the rubric data
            zipObject = zip;
            return zipObject.file("imsmanifest.xml").async("text");
        }).then(function (txt) {
            xmlDoc = parser.parseFromString(txt, "text/xml");
            // Find the name of the file storing rubric data
            let rubricFilename = '';
            let resources = xmlDoc.getElementsByTagName('resources')[0].children;
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
            rubricsXml = xmlDoc.getElementsByTagName('LEARNRUBRICS')[0].children;
            for (let rubric of rubricsXml) {
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

    function onChange(rubricId) {
        jsonData.set({});
        rubricJson = JSON.parse(rubricJsonTemplate);
        // Create TurnItIn formatted json
        if (rubricsXml !== undefined && rubricId !== undefined) {
            let type = rubricsXml[rubricId].getElementsByTagName('Type')[0].attributes.value.value
            rubricJson.Rubric[0].name = rubricsXml[rubricId].getElementsByTagName('Title')[0].attributes.value.value;
            let rubricRowsXml = rubricsXml[rubricId].getElementsByTagName('RubricRows')[0];
            for (let child of rubricRowsXml.children) {
                let rowId = numericalId(child.attributes.id.value);
                let weighting = type.indexOf('NUMERIC') === -1  ? ('' + Number(child.getElementsByTagName('Percentage')[0].attributes.value.value)) : null;
                rubricJson.Rubric[0].criterion.push(rowId);
                rubricJson.RubricCriterion.push(
                    {
                        "description": "",
                        "id": rowId,
                        "name": child.getElementsByTagName('Header')[0].attributes.value.value,
                        "value": weighting
                    }
                )
                let columns = child.getElementsByTagName('RubricColumns')[0].children;
                // Add scales info only if first time through the rows
                if (rubricJson.RubricScale.length === 0) {
                    for (let column of columns) {
                        rubricJson.RubricScale.push(
                            {
                                "id": column.getElementsByTagName('Position')[0].attributes.value.value,
                                "name": column.getElementsByTagName('Header')[0].attributes.value.value,
                                "rubric": numericalId(rubricId),
                                "position": column.getElementsByTagName('Position')[0].attributes.value.value
                            }
                        )
                    }
                }
                for (let column of columns) {
                    // Determine which cell value to get from the Bb rubric
                    let nodeTitle = "NumericPoints";
                    nodeTitle = type === 'NUMERIC_RANGE' ? 'NumericEndPointRange' : nodeTitle;
                    nodeTitle = type === 'PERCENTAGE' ? 'Percentage' : nodeTitle;
                    nodeTitle = type === 'PERCENTAGE_RANGE' ? 'PercentageMax' : nodeTitle;
                     
                    // Display all nonnumeric rubrics as point rubrics
                    rubricJson.Rubric[0].scoring_method = type !== 'NONNUMERIC' ? 4 : 0;
                    
                    // Show warnings if conversions have occured
                    message = type.indexOf('PERCENTAGE') !== -1 ? disclaimerPercent : '';
                    message += type.indexOf('RANGE') !== -1 ? disclaimerRanges : '';
                    
                    rubricJson.RubricCriterionScale.push(
                        {
                            "criterion": rowId,
                            "description": column.getElementsByTagName("CellDescription")[0].attributes.value.value,
                            "id": numericalId(column.getElementsByTagName("Cell")[0].id),
                            "scale_value": column.getElementsByTagName('Position')[0].attributes.value.value,
                            "value": '' + Number(column.getElementsByTagName(nodeTitle)[0].attributes.value.value)
                        }
                    )
                }
            }
            jsonData.set(rubricJson);
        }
    }

    function numericalId(id) {
        return Number(id.split('_').join(''));
    }

</script>

{#if rubrics.length > 0}
<section>
<p>{prompt}</p>
<ul>
    {#each rubrics as rubric}
    <li>
        <label>
            <input type=radio bind:group={rubricId} value="{rubric.id}">{rubric.title}
        </label>
        <br />
        <span>
            {rubric.description}
        </span>
    </li>
    {/each}
</ul>
</section>
{/if}

{#if message !== ''}
    <p><stong>Please note:</stong> {message}</p>
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