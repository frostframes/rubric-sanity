<script>
  import { beforeUpdate } from 'svelte';
  import docx from 'docx';
  import FileSaver from 'file-saver';
  export let rubric;
  export let label = 'Download Word document';
  const bodyFontName = 'Calibri';
  const cellBorders = {
    left: {
        style: docx.BorderStyle.NONE,
        size: 0,
        color: "ffffff",
    },
    right: {
        style: docx.BorderStyle.NONE,
        size: 0,
        color: "ffffff",
    },
    top: {
      style: docx.BorderStyle.SINGLE,
      size: 1,
      color: "aaaaaa",
    },
    bottom: {
      style: docx.BorderStyle.SINGLE,
      size: 1,
      color: "aaaaaa",
    }
  };
  const paragraphStyles = [
    {
        id: "descriptor",
        name: "Descriptor",
        basedOn: "Normal",
        next: "Normal",
        quickFormat: true,
        run: {
            font: bodyFontName
        },
        paragraph: {},
    },
    {
        id: "number",
        name: "Number",
        basedOn: "Normal",
        next: "Normal",
        quickFormat: true,
        run: {
            italics: true,
            color: "999999",
            font: bodyFontName
        },
        paragraph: {},
    },
    {
      id: "cellheading",
      name: "Cell Heading",
      basedOn: "Normal",
      next: "Normal",
      quickFormat: true,
      run: {
          size: 26,
          color: "336699",
          font: bodyFontName
      },
      paragraph: {
          spacing: {
              before: 0,
              after: 0
          },
      },
    },
  ];

  const headerCellShading = {
      fill: "eeeeee",
  };

  function onClick() {
    const doc = new docx.Document({
        styles: {
          paragraphStyles: paragraphStyles
        }
      });

    let paragraphs = [new docx.Paragraph({
      text: rubric.title,
      heading: docx.HeadingLevel.TITLE,
    })];

    // Create table. Will be added once content is inserted.
    if (rubric.scales.length > 0) {
        let tableRows = [];
        // Add table header row
        let tableCells = [new docx.TableCell({
            borders: cellBorders,
            children: [
              new docx.Paragraph({
                text: '',
              })
            ],
            shading: headerCellShading,
          })
        ];

        for (let scale of rubric.scales) {
          let tableCell = new docx.TableCell({
            borders: cellBorders,
            children: [
              new docx.Paragraph({
                text: scale.name,
                style: 'cellheading',
              }),
              new docx.Paragraph({
                text: scale.value > 0 ? scale.value : '',
                style: 'number',
              }),
            ],
            shading: headerCellShading,
          });
          tableCells.push(tableCell);
        }
        tableRows.push(new docx.TableRow({
          children: tableCells
        }));

        // Add criteria
        for (let criterion of rubric.criteria) {
          // Add criterion descriptors
          tableCells = [new docx.TableCell({
            borders: cellBorders,
            children: [
              new docx.Paragraph({
                text: criterion.name,
                style: 'cellheading',
              }),
              new docx.Paragraph({
                text: criterion.description !== null ? criterion.description : '',
                style: 'descriptor',
              }),
              new docx.Paragraph({
                text: criterion.value !== '0' && criterion.value !== null ? criterion.value + '%' : '',
                style: 'number',
              }),
            ],
            shading: headerCellShading,
          })];
          // add criterion scales
          for (let criterionScale of criterion.scales) {
            let tableCell = new docx.TableCell({
              borders: cellBorders,
              children: [
                new docx.Paragraph({
                  text: criterionScale.description,
                  style: 'descriptor',
                }),
                new docx.Paragraph({
                  text: criterionScale.value !== '0' && criterionScale.value !== null ? criterionScale.value : '',
                  style: 'number',
                }),
              ],
            });
            tableCells.push(tableCell);
          }
          tableRows.push(new docx.TableRow({
            children: tableCells
          }));
        }

        paragraphs.push(new docx.Table({
          rows: tableRows,
        }));
    } else {
      for (let criterion of rubric.criteria) {
        paragraphs.push(new docx.Paragraph({
          text: criterion.name,
          style: 'cellheading',
        }));
        paragraphs.push(new docx.Paragraph({
          text: criterion.description !== null ? criterion.description : '',
          style: 'descriptor',
        }));
      }
    }

    // Add the content
    doc.addSection({
      orientation: docx.PageOrientation.LANDSCAPE,
      top: 500,
      right: 500,
      bottom: 500,
      left: 500,
      children: paragraphs,  
    });

    // Download the word doc as a file to the client web browser
    docx.Packer.toBlob(doc).then((blob) => {
      FileSaver.saveAs(blob, `${rubric.title}.docx`);
    });
  }


</script>

{#if rubric !== undefined && rubric.title !== ''}
  <button on:click={onClick}>{ label }</button>
{/if}