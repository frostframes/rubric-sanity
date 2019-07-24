<script>
  import { beforeUpdate } from 'svelte';
  import docx from 'docx';
  import FileSaver from 'file-saver';
  export let rubric;
  export let label = 'Download Word document';
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

    // Add title
    doc.addParagraph(setRomanParagraph(rubric.title).title());

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
      cell.addParagraph(setItalicsParagraph(scale.value > 0 ? scale.value : '').center());
      removeVerticalBorders(cell);
      cell.Borders.addTopBorder(docx.BorderStyle.NONE, 0, '#ffffff');
    }

    // Add criteria
    for (let criterion of rubric.criteria) {
      cell = table.getCell((criterion.position + 1), 0);
      // Add criterion title
      cell.addParagraph(setRomanParagraph(criterion.name).heading3());
      // Add criterion weighting
      if (criterion.value !== '0' && criterion.value !== null) {
        cell.addParagraph(setItalicsParagraph(`${criterion.value}%`));
      }
      // Add criterion description
      cell.addParagraph(setRomanParagraph(criterion.description));
      removeVerticalBorders(cell);
      // Add cells
      for (let scaleIndex in criterion.scales) {
        // index must be a number
        scaleIndex = Number(scaleIndex);
        let scale = criterion.scales[scaleIndex];
        cell = table.getCell((criterion.position + 1), (scaleIndex + 1));
        if (criterion.value === '0') {
          cell.addParagraph(setItalicsParagraph(`${scale.value} marks`));
        }
        cell.addParagraph(setRomanParagraph(scale.description));
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

<style>
button {
  font-size: large;
  padding: 10px;
}
</style>

{#if rubric !== undefined && rubric.title !== ''}
  <button on:click={onClick}>{ label }</button>
{/if}