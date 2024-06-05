<script>
  //store
  import {
    growthFactorData,
    cashFlowData,
    financingOptionsData,
    totalInvestmentByType1,
    cashFlowOptionsData,
  } from "./../../store/store";
  //utils
  import { data } from "./../../utils/calculate";
  export let year; // año para analizar
  export let elements; // intereses por periodo de cada recurso

  let viewInFlows = false;
  let viewOutFlows = false;
  let viewNetIncome = false;
  let viewExpenses = false;
  const periods = data[$financingOptionsData.periodicity];
  const units = $growthFactorData.units;
  const prices = $growthFactorData.prices;
  const costs = $growthFactorData.costs;

  function seeInFlows() {
    viewInFlows = !viewInFlows;
  }

  function seeOutFlows() {
    viewOutFlows = !viewOutFlows;
  }

  function seeNetIncome() {
    viewNetIncome = !viewNetIncome;
  }

  function seeExpenses() {
    viewExpenses = !viewExpenses;
  }

  // calcular totales generales de flujo de entrada y salida
  $: totalIncome = $cashFlowData
    .filter((el) => el.flowType === "Flujo de entrada")
    .reduce(
      (acc, el) =>
        acc +
        el.value *
          Math.pow((prices + 100) / 100, year - 1) *
          el.qty *
          Math.pow((units + 100) / 100, year - 1),
      0
    );

  $: totalExpenses = $cashFlowData
    .filter((el) => el.flowType === "Flujo de salida")
    .reduce((acc, el) => acc + (el.value / 100) * totalIncome, 0);

  // calcular totales individuales de flujo de entrada y salida
  $: totalInFlow = (flow) => {
    return Math.round(
      flow.value *
        Math.pow((prices + 100) / 100, year - 1) *
        flow.qty *
        Math.pow((units + 100) / 100, year - 1)
    ).toLocaleString();
  };

  $: totalOutFlow = (flow) => {
    return Math.round((flow.value / 100) * totalIncome).toLocaleString();
  };

  function filterByFlowType(flowType) {
    try {
      return $cashFlowData.filter((el) => el.flowType === flowType);
    } catch (error) {
      console.error(error);
    }
  }

  const inflows = filterByFlowType("Flujo de entrada");
  const outflows = filterByFlowType("Flujo de salida");

  // depreciacion y amortizacion
  $: totalAmortization = Math.round(
    ($cashFlowOptionsData.amortization / 100) * $totalInvestmentByType1
  );

  // gastos financieros
  $: elementsByPeriod = elements.map((el) => [
    el[0],
    el[1].slice((year - 1) * periods, year * periods),
  ]);

  $: sumElementsByPeriod = elementsByPeriod.reduce((acc, curr) => {
    return acc[1].map((num, idx) => num + curr[1][idx]);
  });

  // utilidad neta
  $: totalMargin = Array(periods).fill(Math.round(totalIncome - totalExpenses));
  let totalUtility;
  $: {
    totalUtility = [];
    for (let i = 0; i < periods; i++) {
      totalUtility.push(Math.round(totalMargin[i] - sumElementsByPeriod[i]));
    }
  }
  $: totalTaxProvision = totalUtility.map((el) =>
    Math.round(el * ($cashFlowOptionsData.taxProvision / 100))
  );

  let totalNetIncome;
  $: {
    totalNetIncome = [];
    for (let i = 0; i < periods; i++) {
      totalNetIncome.push(Math.round(totalUtility[i] - totalTaxProvision[i]));
    }
  }
</script>

<div class="year-analysis">
  <div class="element-year-analysis">
    <div class="item-main-analysis" on:click={seeInFlows}>
      <h4>Flujo de entrada</h4>
      {#each Array(periods).fill(Math.round(totalIncome).toLocaleString()) as el}
        <h4>{el}</h4>
      {/each}
    </div>
    {#if viewInFlows}
      <div class="flow-list-analysis">
        {#each inflows as el}
          <div class="item item-secondary-analysis">
            <p><b>{el.description}</b></p>
            {#each Array(periods).fill(totalInFlow(el)) as d}
              <p>{d}</p>
            {/each}
          </div>
        {/each}
      </div>
    {/if}
  </div>

  <div class="element-year-analysis">
    <div class="item-main-analysis" on:click={seeOutFlows}>
      <h4>Flujo de salida</h4>
      {#each Array(periods).fill(Math.round(totalExpenses).toLocaleString()) as el}
        <h4>{el}</h4>
      {/each}
    </div>
    {#if viewOutFlows}
      <div class="flow-list-analysis">
        {#each outflows as el}
          <div class="item item-secondary-analysis">
            <p><b>{el.description}</b></p>
            {#each Array(periods).fill(totalOutFlow(el)) as d}
              <p>{d}</p>
            {/each}
          </div>
        {/each}
        <div class="item item-secondary-analysis">
          <p><b>Depreciación y amortización</b></p>
          {#each Array(periods).fill(totalAmortization.toLocaleString()) as d}
            <p>{d}</p>
          {/each}
        </div>
      </div>
    {/if}
  </div>

  <div class="element-year-analysis">
    <div class="item-main-analysis" on:click={seeExpenses}>
      <h4>Gastos financieros</h4>
      {#each sumElementsByPeriod as el}
        <h4>{Math.round(el).toLocaleString()}</h4>
      {/each}
    </div>
    {#if viewExpenses}
      <div class="flow-list-analysis">
        {#each elementsByPeriod as el}
          <div class="item item-secondary-analysis">
            <p><b>{el[0]}</b></p>
            {#each el[1] as d}
              <p>{Math.round(d).toLocaleString()}</p>
            {/each}
          </div>
        {/each}
      </div>
    {/if}
  </div>

  <div class="element-year-analysis">
    <div class="item-main-analysis" on:click={seeNetIncome}>
      <h4>Utilidad Neta</h4>
      {#each totalNetIncome as el}
        <h4>{el.toLocaleString()}</h4>
      {/each}
    </div>
    {#if viewNetIncome}
      <div class="flow-list-analysis">
        <div class="item item-secondary-analysis">
          <p><b>Margen operacional</b></p>
          {#each totalMargin as el}
            <p>{el.toLocaleString()}</p>
          {/each}
        </div>
        <div class="item item-secondary-analysis">
          <p><b>Utilidad antes de impuestos</b></p>
          {#each totalUtility as el}
            <p>{el.toLocaleString()}</p>
          {/each}
        </div>
        <div class="item item-secondary-analysis">
          <p><b>Provisión de impuestos</b></p>
          {#each totalTaxProvision as el}
            <p>{el.toLocaleString()}</p>
          {/each}
        </div>
      </div>
    {/if}
  </div>
</div>

<style>
  .year-analysis {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
    min-width: max-content;
  }

  .element-year-analysis {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
  }

  .flow-list-analysis {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
    margin-left: 1rem;
  }

  .item-main-analysis {
    display: flex;
    justify-content: space-between;
    gap: 1rem;
    align-items: center;
    padding: 1rem;
    background-color: var(--color6);
    color: white;
    border-radius: 8px;
    cursor: pointer;
  }

  .item-main-analysis h4 {
    margin: 0;
  }

  .item-secondary-analysis {
    box-shadow: none;
    padding: 0.5rem;
  }

  .item-secondary-analysis p {
    margin: 5px;
  }
</style>