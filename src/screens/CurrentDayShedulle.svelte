<script lang="ts">
  import { cancelOrder, getOrders } from '../api/orders'
  import { getAllServices } from '../api/services';
  import BackToMenu from '../components/BackToMenu.svelte';
  import Loader from '../components/Loader.svelte';
  import PrevNextButtons from '../components/PrevNextButtons.svelte';
  import ShedulleOrders from '../components/shedulle/ShedulleOrders.svelte';
  import type { Order } from '../models/Order'

  let date = new Date(new Date().getFullYear(), new Date().getMonth(), new Date().getDate())
  
  function nextDay() {
    date = new Date(date.getFullYear(), date.getMonth(), date.getDate() + 1)
  }
  
  function prevDay() {
    date = new Date(date.getFullYear(), date.getMonth(), date.getDate() - 1)
  }

  async function handleCancelOrder({ detail }: CustomEvent<Order>) {
    const { success } = await cancelOrder(detail.id)
    if (success) {
      ordersPromise = getOrders(date.getFullYear(), date.getMonth(), date.getDate())
    }
  }

  let servicesPromise = getAllServices()

  $: ordersPromise = getOrders(date.getFullYear(), date.getMonth(), date.getDate())

</script>

<div class="mb-12 fullw">
  <PrevNextButtons
    currentText={date.toLocaleString('ru', { day: 'numeric', month: 'long'})}
    on:next={nextDay}
    on:prev={prevDay}
  />
</div>
<div class="orders mb-20 fullw">
  {#await Promise.all([servicesPromise, ordersPromise])}
    <Loader />
  {:then [services, orders]}
    {#if orders.length > 0}
      <ShedulleOrders
        orders={orders}
        services={services}
        on:delete={handleCancelOrder}
      />
    {:else}
      <div class="no-orders fullw">
        Нет заказов
      </div>
    {/if}
  {/await}
</div>

<BackToMenu />

<style>
  .no-orders {
    display: flex;
    font-size: 24px;
    font-weight: bold;
    align-items: center;
    justify-content: center;
    min-height: 33vh;
  }
</style>

