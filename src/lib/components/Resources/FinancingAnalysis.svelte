<script>
  //components
  import Modal from "./../Elements/Modal.svelte";
  //utils
  import { calculateNominalRate, data } from "./../../utils/calculate";
  //store
  import {
    financingOptionsData,
    resourceData,
    totalInvestment,
    totalResourceByType1,
    totalResourceByType2,
  } from "../../store/store";

  let showModal = false;

  function openModal() {
    showModal = true;
  }

  function closeModal() {
    showModal = false;
  }

  let elements = [];
  let totalParticipation = 0;
  let finalTotalCPPC = 0;
  const types = ["Propio", "Externo"];
  const periods = data[$financingOptionsData.periodicity];

  $: {
    elements = [];
    totalParticipation = 0;
    finalTotalCPPC = 0;
    for (let i = 0; i < types.length; i++) {
      const description = "Recurso " + types[i];
      const value =
        types[i] === "Propio" ? $totalResourceByType1 : $totalResourceByType2;
      const participation =
        $totalInvestment && (value * 100) / $totalInvestment;
      const totalCPPC = $resourceData.reduce((total, resource) => {
        if (resource.type === types[i]) {
          const nominalRate = calculateNominalRate(resource.rate, periods);
          const CPPC =
            (nominalRate * (resource.value * 100)) / $totalInvestment;
          return total + CPPC;
        }
        return total;
      }, 0);
      totalParticipation += participation;
      finalTotalCPPC += totalCPPC;
      elements.push([
        description,
        `${value.toLocaleString()}$`,
        `${participation.toFixed(2)}%`,
        `${totalCPPC.toFixed(2)}%`,
      ]);
    }
    elements.push([
      "Total",
      `${($totalResourceByType1 + $totalResourceByType2).toLocaleString()}$`,
      `${totalParticipation.toFixed(2)}%`,
      `${finalTotalCPPC.toFixed(2)}%`,
    ]);
  }
</script>

<div class="analysis">
  <h2>Analisis de financiación</h2>
  <div class="see-data">
    <button on:click={openModal}>Analizar</button>
  </div>

  {#if showModal}
    <Modal>
      <div class="modal-header">
        <h2>Analisis</h2>
      </div>
      <div class="modal-content">
        <table>
          <thead>
            <tr>
              <th>Descripción</th>
              <th>Capital</th>
              <th>Participación%</th>
              <th>CPPC%</th>
            </tr>
          </thead>
          <tbody>
            {#each elements as el}
              <tr>
                {#each el as data}
                  <td>{data}</td>
                {/each}
              </tr>
            {/each}
          </tbody>
        </table>
      </div>
      <div class="modal-footer">
        <button on:click={closeModal}>Aceptar</button>
      </div>
    </Modal>
  {/if}
</div>

<style>
  .analysis {
    display: flex;
    flex-direction: column;
    gap: 1rem;
    padding: 10px;
    border: 1px solid #a9a9a9;
    border-radius: 8px;
    background-color: var(--color3);
    box-shadow: 0px 0px 5px 0px #a9a9a9;
  }

  .see-data {
    display: flex;
    justify-content: center;
  }

  .modal-footer {
    display: flex;
    justify-content: flex-end;
    gap: 1rem;
  }

  table {
    width: 100%;
    border-collapse: collapse;
    margin-bottom: 20px;
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  }

  th,
  td {
    border: 1px solid #ddd;
    padding: 12px;
    text-align: center;
    background-color: #fff;
  }

  th {
    background-color: #14213d;
    color: white;
  }
</style>
