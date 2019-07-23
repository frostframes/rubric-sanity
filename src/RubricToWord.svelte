<script>
  import { beforeUpdate } from 'svelte';
  import docx from 'docx';
  import FileSaver from 'file-saver';
  export let data;
  const bodyFontName = 'Calibri';
  let rubric = {};

  beforeUpdate(function() {
    // Clear any previouly loaded rubric data
    rubric = {};
  })

  function setRomanParagraph(string) {
    const text = new docx.TextRun(string).font(bodyFontName);
    return new docx.Paragraph(text);
  }

  function setItalicsParagraph(string) {
    const text = new docx.TextRun(string).font(bodyFontName).italics();
    return new docx.Paragraph(text);
  }

  function onClick() {
    const doc = new docx.Document();
    const packer = new docx.Packer();
    let cell;
    let criterion;
    let descriptionText = '';
    rubric = {
      criteria: [],
      title: data.Rubric[0].name,
      scales: data.RubricScale
    };

    // Add title
    doc.addParagraph(setRomanParagraph(rubric.title).title());

    // Create table. Will be added once content is inserted.
    let table = new docx.Table({
      rows: data.Rubric[0].criterion.length + 2,
      columns: rubric.scales.length + 1,
    });

    // Add header row (scale titles and values)
    for (let scale of rubric.scales) {
      cell = table.getCell(0, scale.position);
      cell.addParagraph(setRomanParagraph(scale.name).center().heading3());
      cell = table.getCell(1, scale.position);
      cell.addParagraph(setItalicsParagraph(scale.value).center());
    }

    // Add criteria
    for (let criterionId of data.Rubric[0].criterion) {
      criterion = data.RubricCriterion.filter(element => element.id === criterionId)[0];
      rubric.criteria.push(criterion);
      cell = table.getCell((criterion.position + 1), 0);
      // Add criterion title
      cell.addParagraph(setRomanParagraph(criterion.name).heading3());
      // Add criterion weighting
      cell.addParagraph(setItalicsParagraph(`${criterion.value}%`));
      // Add criterion description
      cell.addParagraph(setRomanParagraph(criterion.description));
      // Add cells
      for (let scaleIndex in criterion.criterion_scales) {
        // index must be a number
        scaleIndex = Number(scaleIndex);
        descriptionText = data.RubricCriterionScale.filter(item => item.id === criterion.criterion_scales[scaleIndex])[0].description;
        cell = table.getCell((criterion.position + 1), (scaleIndex + 1));
        cell.addParagraph(setRomanParagraph(descriptionText));
      }
    }

    // Add the table into the document
    doc.addTable(table);

    // Download the word doc as a file to the client web browser
    packer.toBlob(doc).then((blob) => {
      FileSaver.saveAs(blob, rubric.title + '.docx');
    });
  }

</script>
{#if data.Rubric !== undefined && data.Rubric[0] !== undefined && data.RubricScale !== undefined}
  <button on:click={onClick}>Download as Word</button>
{/if}