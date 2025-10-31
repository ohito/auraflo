<template>
  <admin-layout>
    <div class="grid grid-cols-12 gap-4 md:gap-6">
       <div class="col-span-12">
        <ecommerce-metrics />
       </div>

      <div class="col-span-12 xl:col-span-3">
        <sales-forecast />
      </div>

      <div class="col-span-12 xl:col-span-3">
        <deal-type />
      </div>

       <div class="col-span-12 xl:col-span-6">
        <balance-overview />
      </div>

      <div class="col-span-12 xl:col-span-7">
        <deals-status />
      </div>

      <div class="col-span-12 xl:col-span-5">
        <my-tasks />
      </div>

      <div class="col-span-5">
        <upcoming-activities />
      </div>
       
      <div class="col-span-7">
        <closing-deals />
      </div>
    </div>
  </admin-layout>
</template>

<script>
import AdminLayout from '../components/layout/AdminLayout.vue'
import EcommerceMetrics from '../components/ecommerce/EcommerceMetrics.vue'
import SalesForecast from '../components/ecommerce/SalesForecast.vue'
import DealType from '../components/ecommerce/DealType.vue'
import BalanceOverview from '../components/ecommerce/BalanceOverview.vue'
import DealsStatus from '../components/ecommerce/DealsStatus.vue'
import MyTasks from '../components/ecommerce/MyTasks.vue'
import UpcomingActivities from '../components/ecommerce/UpcomingActivities.vue'
import ClosingDeals from '../components/ecommerce/ClosingDeals.vue'

export default {
  components: {
    AdminLayout,
    EcommerceMetrics,
    SalesForecast,
    DealType,
    BalanceOverview,
    DealsStatus,
    MyTasks,
    UpcomingActivities,
    ClosingDeals,
  },
  name: 'Ecommerce',
}
</script>
