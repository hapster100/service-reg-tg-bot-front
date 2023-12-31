<script lang="ts">
  import { writable } from "svelte/store"
  import type { Category } from "../../models/Category"
  import type { Service } from "../../models/Service"
  import { OrderStep, OrderStore } from "../../stores/newOrder"
  import ServiceInfo from "../ServiceInfo.svelte"
  import ServicesList from '../SirvicesList.svelte'
  import Time from "../Time.svelte";

  export let services : Service[]
  export let categories : Category[]
  export let store: OrderStore

  const { serviceIds, currentStep } = store

  let serviceById = {} as {[key: string]: Service}
  let listOpen = writable($serviceIds.length === 0)
  let totalCost = 0
  let totalDuration = 0

  $: {
    serviceById = services.reduce((acc, s) => (acc[s.id] = s, acc), {})
  }

  $: {
    totalCost = $serviceIds.reduce((acc, id) => serviceById[id]?.cost + acc, 0)
    totalDuration = $serviceIds.reduce((acc, id) => serviceById[id]?.durationMinutes + acc, 0)
  }


  function remove(serviceId: string) {
    const i = $serviceIds.findIndex(x => x === serviceId)
    if (i >= 0) {
      const next = [...$serviceIds.slice(0, i), ...$serviceIds.slice(i + 1)]
      $serviceIds = next
    }
  }
  
  function add({ detail: serviceId }) {
    $serviceIds = [...$serviceIds, serviceId]
    $listOpen = false
  }

</script>

{#if $listOpen}
  <h3 class="page-title">Выбор услуги</h3>
  <ServicesList 
    services={services}
    categories={categories}
    on:select={add}
  />
  {#if $serviceIds.length > 0}
    <button class="fullw" on:click={() => $listOpen = false}>Выбранные услуги</button>
  {/if} 
{:else}
  <h3 class="page-title">Выбранные услуги</h3> 
  {#each $serviceIds as serviceId}
    <div class='service-info-wrapper'>
      <ServiceInfo service={serviceById[serviceId]}/>
      <button class="remove-btn" on:click={() => remove(serviceId)}>⨯</button>
    </div>
  {/each}
  <div class="new-order-total">
    Итого: {totalCost}руб. <Time duration={totalDuration}/>
  </div>
  <div class="new-order-ctrl">
    <button class="outline-btn" on:click={() => $listOpen = true}>Добавить еще</button>
    <button
      class:disabled-btn={$serviceIds.length === 0}
      disabled={$serviceIds.length === 0} 
      on:click={() => $currentStep = OrderStep.Date}>Далее</button>
  </div>
{/if}

<style>
  .remove-btn {
    height: auto;
  }
  .service-info-wrapper {
    box-sizing: border-box;
    margin: auto;
    width: calc(100% - 8px);
    margin-bottom: 8px;
    border: 1px solid var(--color-text);
    border-radius: 8px;
    display: flex;
  }
  .service-info-wrapper  button {
    aspect-ratio: 1;
    font-size: 150%;
    border-bottom-left-radius: 0;
    border-top-left-radius: 0;
  }

  .new-order-total {
    font-size: 0.8rem;
    margin: 12px;
  }

  .new-order-ctrl {
    display: flex;
    flex-direction: row;
  }

  .new-order-ctrl button {
    flex: 1;
    margin: 4px 4px;
  }

  .new-order-ctrl button:last-child {
    margin-right: 0;
  }

  .new-order-ctrl button:first-child {
    margin-left: 0;
  }
</style>