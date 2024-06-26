<script>
  import { v4 as uuidv4 } from "uuid";
  import { toast } from "svelte-easy-toast";
  //components
  import ResourceList from "./ResourceList.svelte";
  import Modal from "./../Elements/Modal.svelte";
  import Table from "./Table.svelte";
  import WarningModal from "./../Elements/WarningModal.svelte";
  //store
  import {
    resourceData,
    totalInvestment,
    totalResources,
  } from "./../../store/store";

  const types = ["Propio", "Externo"];

  let showModal = false;
  let showTableModal = false;
  let showWarningModal = false;
  let selectedResource = null;
  let description = "";
  let value = 0;
  let rate = 0;
  let type = types[0];
  let filteredElements = [];
  let selectedType = "";

  function resetFields() {
    description = "";
    value = 0;
    rate = 0;
    type = types[0];
    selectedResource = null;
  }

  function openModal() {
    showModal = true;
  }

  function closeModal() {
    showModal = false;
  }

  function openTableModal() {
    showTableModal = true;
  }

  function closeTableModal() {
    showTableModal = false;
  }

  function openWarningModal() {
    showWarningModal = true;
  }

  function closeWarningModal() {
    showWarningModal = false;
  }

  function createResource(newResource) {
    try {
      const { value } = newResource;
      if ($totalResources + value > $totalInvestment) {
        openWarningModal();
        return;
      }
      resourceData.update((data) => [
        ...data,
        { _id: uuidv4(), ...newResource },
      ]);
      closeModal();
      toast({
        type: "success",
        position: "bottom-right",
        text: "Se ha creado el recurso correctamente!",
        title: "Recurso creado",
        delay: 3000,
      });
    } catch (error) {
      console.error(error);
    }
  }

  function updateResource(updatedResource) {
    try {
      const { value } = updatedResource;
      const resource = $resourceData.find(
        (resource) => resource._id === selectedResource._id
      );
      const oldValue = resource.value;
      if ($totalResources - oldValue + value > $totalInvestment) {
        openWarningModal();
        return;
      }
      resourceData.update((data) =>
        data.map((resource) =>
          resource._id === selectedResource._id
            ? { ...resource, ...updatedResource }
            : resource
        )
      );
      closeModal();
      toast({
        type: "success",
        position: "bottom-right",
        text: "Se ha actualizado el recurso correctamente!",
        title: "Recurso actualizado",
        delay: 3000,
      });
    } catch (error) {
      console.error(error);
    }
  }

  function deleteResource(resourceId) {
    try {
      resourceData.update((data) =>
        data.filter((resource) => resource._id !== resourceId)
      );
      toast({
        type: "success",
        position: "bottom-right",
        text: "Se ha eliminado el recurso correctamente!",
        title: "Recurso eliminado",
        delay: 3000,
      });
    } catch (error) {
      console.error(error);
    }
  }

  $: filteredElements =
    selectedType === ""
      ? $resourceData
      : $resourceData.filter((resource) => resource.type === selectedType);
</script>

<div class="resources">
  <div class="options">
    <button
      class:active={selectedType === ""}
      on:click={() => {
        selectedType = "";
      }}
    >
      Todos los recursos
    </button>
    <button
      class:active={selectedType === "Propio"}
      on:click={() => {
        selectedType = "Propio";
      }}
    >
      Recursos Propios
    </button>
    <button
      class:active={selectedType === "Externo"}
      on:click={() => {
        selectedType = "Externo";
      }}
    >
      Recursos Externos
    </button>
  </div>

  <div class="save-data">
    <button
      on:click={() => {
        resetFields();
        openModal();
      }}>Crear Recurso</button
    >
  </div>

  <ResourceList
    resources={filteredElements}
    updateResource={(resource) => {
      selectedResource = resource;
      description = resource.description;
      value = resource.value;
      rate = resource.rate;
      type = resource.type;
      openModal();
    }}
    {deleteResource}
    seeResource={(resource) => {
      selectedResource = resource;
      openTableModal();
    }}
  />

  {#if showModal}
    <Modal>
      <div class="modal-header">
        <h2>{selectedResource ? "Editar" : "Crear"} Recurso</h2>
      </div>
      <div class="modal-content">
        <div class="form-element">
          <label for="description">Descripción</label>
          <input
            id="description"
            type="text"
            bind:value={description}
            placeholder="Descripción"
          />
        </div>
        <div class="form-element">
          <label for="value">Capital</label>
          <input id="value" type="number" bind:value placeholder="Capital" />
        </div>
        <div class="form-element">
          <label for="rate">Tasa efectiva anual</label>
          <input
            id="rate"
            type="number"
            bind:value={rate}
            placeholder="Tasa efectiva anual"
          />
        </div>
        <div class="form-element">
          <label for="type">Tipo de recurso</label>
          <select id="type" bind:value={type}>
            {#each types as t}
              <option value={t}>{t}</option>
            {/each}
          </select>
        </div>
      </div>
      <div class="modal-footer">
        <button on:click={closeModal}>Cancelar</button>
        <button
          type="button"
          on:click={() => {
            const elementData = {
              description,
              value,
              rate,
              type,
            };
            selectedResource
              ? updateResource(elementData)
              : createResource(elementData);
          }}>{selectedResource ? "Editar" : "Crear"}</button
        >
      </div>
    </Modal>
  {/if}

  {#if showTableModal}
    <Modal>
      <div class="modal-header">
        <h2>Tabla de Amortización</h2>
      </div>
      <div class="modal-content">
        <Table item={selectedResource} />
      </div>
      <div class="modal-footer">
        <button on:click={closeTableModal}>Cerrar</button>
      </div>
    </Modal>
  {/if}

  <WarningModal
    title="Advertencia"
    message="No se puede crear este recurso porque está superando el total de inversión."
    show={showWarningModal}
    close={closeWarningModal}
  />
</div>

<style>
  .resources {
    display: flex;
    flex-direction: column;
    gap: 10px;
  }

  .modal-content {
    display: flex;
    flex-direction: column;
    gap: 10px;
    overflow-y: auto;
    max-height: 300px;
    min-width: 500px;
  }

  .modal-footer {
    display: flex;
    justify-content: flex-end;
    gap: 1rem;
  }

  .options {
    display: flex;
    justify-content: center;
    gap: 1rem;
  }

  .active {
    background-color: var(--color2);
  }

  .save-data {
    display: flex;
    justify-content: flex-end;
    gap: 1rem;
  }
</style>
