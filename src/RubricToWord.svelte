<script>
  import { beforeUpdate } from 'svelte';
  import docx from 'docx';
  import FileSaver from 'file-saver';
  export let rubric;
  const bodyFontName = 'Calibri';

  function setRomanParagraph(string) {
    const text = new docx.TextRun(string).font(bodyFontName);
    return new docx.Paragraph(text);
  }

  function setItalicsParagraph(string) {
    const text = new docx.TextRun(string).font(bodyFontName).italics();
    return new docx.Paragraph(text);
  }

  function removeVerticalBorders(cell) {
    const borderStyle = docx.BorderStyle.NONE;
    cell.Borders.addStartBorder(borderStyle, 0, '#ffffff');
    cell.Borders.addEndBorder(borderStyle, 0, '#ffffff');
  }

  function onClick() {
    const doc = new docx.Document(undefined, {
        orientation: "landscape",
        top: 500,
        right: 500,
        bottom: 500,
        left: 500,
    });
    const packer = new docx.Packer();
    let cell;
    let criterion;
    let descriptionText = '';

    // Add title
    doc.addParagraph(setRomanParagraph(rubric.name).title());

    // Create table. Will be added once content is inserted.
    let table = new docx.Table({
      rows: rubric.criteria.length + 2,
      columns: rubric.scales.length + 1,
    });

    // Format empty cells at top-left
    cell = table.getCell(0, 0);
    removeVerticalBorders(cell);
    cell.Borders.addBottomBorder(docx.BorderStyle.NONE, 0, '#ffffff');
    cell = table.getCell(1, 0);
    removeVerticalBorders(cell);
    cell.Borders.addTopBorder(docx.BorderStyle.NONE, 0, '#ffffff');

    // Add header row (scale titles and values)
    for (let scale of rubric.scales) {
      cell = table.getCell(0, scale.position);
      cell.addParagraph(setRomanParagraph(scale.name).center().heading3());
      removeVerticalBorders(cell);
      cell.Borders.addBottomBorder(docx.BorderStyle.NONE, 0, '#ffffff');
      cell = table.getCell(1, scale.position);
      cell.addParagraph(setItalicsParagraph(scale.value).center());
      removeVerticalBorders(cell);
      cell.Borders.addTopBorder(docx.BorderStyle.NONE, 0, '#ffffff');
    }

    // Add criteria
    for (let criterion of rubric.criteria) {
      cell = table.getCell((criterion.position + 1), 0);
      // Add criterion title
      cell.addParagraph(setRomanParagraph(criterion.name).heading3());
      // Add criterion weighting
      cell.addParagraph(setItalicsParagraph(`${criterion.value}%`));
      // Add criterion description
      cell.addParagraph(setRomanParagraph(criterion.description));
      removeVerticalBorders(cell);
      // Add cells
      for (let scaleIndex in criterion.criterion_scales) {
        // index must be a number
        scaleIndex = Number(scaleIndex);
        descriptionText = rubric.descriptors.filter(item => item.id === criterion.criterion_scales[scaleIndex])[0].description;
        cell = table.getCell((criterion.position + 1), (scaleIndex + 1));
        cell.addParagraph(setRomanParagraph(descriptionText));
        removeVerticalBorders(cell);
      }
    }

    // Add the table into the document
    doc.addTable(table);

    // Download the word doc as a file to the client web browser
    packer.toBlob(doc).then((blob) => {
      FileSaver.saveAs(blob, `${rubric.title}.docx`);
    });
  }

</script>
{#if rubric !== undefined && rubric.title !== ''}
  <button on:click={onClick}>Download as Word</button>
{/if}