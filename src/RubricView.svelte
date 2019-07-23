<script>
  import { beforeUpdate } from 'svelte';
	import RubricToWord from './RubricToWord.svelte';
  export let data;
  let rubric = {};
  beforeUpdate(function() {
    if (
      data.Rubric !== undefined &&
      data.Rubric[0] !== undefined &&
      data.RubricScale !== undefined
    ) {
      rubric = {
        criteria: [],
        descriptors: data.RubricCriterionScale,
        title: data.Rubric[0].name,
        scales: data.RubricScale
      };
      for (let criterionId of data.Rubric[0].criterion) {
        rubric.criteria.push(
          data.RubricCriterion.filter(element => element.id === criterionId)[0]
        );
      }
    } else {
      rubric = {
        criteria: [],
        descriptors: [],
        scales: [],
        title: ''
      };
    }
  });
</script>

<style>
  td,
  th {
    font-size: small;
    text-align: left;
    vertical-align: top;
    border-bottom: 1px solid grey;
    padding: 10px;
    font-weight: normal;
    width: 15%;
    position: relative;
  }
  th {
    background-color: #eee;
  }
  h2,
  p {
    margin: 0;
  }
  table {
    border-collapse: collapse;
    border: lightgray 1px solid;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.25);
  }
  code {
    font-size: 1.5em;
    opacity: 0.75;
    font-style: italic;
  }
</style>

<RubricToWord label={'Download as Word'} rubric={rubric} />

<h1>{rubric.title}</h1>
<table>
  <thead>
    <tr>
    {#if rubric.scales.length > 0}
      <th></th>
    {/if}
      {#each rubric.scales as scale}
        <th>
          <h2>{scale.name}</h2>
        </th>
      {/each}
    </tr>
    <tr>
    {#if rubric.scales.length > 0}
      <th></th>
    {/if}
      {#each rubric.scales as scale}
        <th>
          <code>{scale.value}</code>
        </th>
      {/each}
    </tr>
  </thead>
  {#each rubric.criteria as criterion}
    <tr>
      <th>
        <h2>{criterion.name}</h2>
        <p>
        {#if criterion.value !== null}
          <code>{criterion.value}%</code>
        {/if}
        {#if criterion.description !== null}
          <p>{criterion.description}</p>
        {/if}
        </p>
      </th>
      {#each rubric.scales as scale}
        <td>{rubric.descriptors.filter(element => (element.criterion === criterion.id && element.scale_value === scale.id))[0].description}</td>
      {/each}
    </tr>
  {/each}
</table>