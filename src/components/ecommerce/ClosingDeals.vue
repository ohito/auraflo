<template>
  <div class="overflow-hidden rounded-2xl border border-gray-200 bg-white p-6">
    <!-- Header -->
    <div class="flex items-center justify-between mb-4">
      <h3 class="text-lg font-semibold text-slate-800">Closing Deals</h3>
      <div>
        <select
          v-model="filter"
          class="border border-gray-200 rounded-md px-3 py-1.5 text-sm focus:outline-none focus:ring-1 focus:ring-emerald-400"
        >
          <option value="closed">Closed Deals</option>
          <option value="won">Won Deals</option>
          <option value="lost">Lost Deals</option>
        </select>
      </div>
    </div>

    <!-- Table -->
    <div class="max-w-full overflow-x-auto custom-scrollbar">
      <table class="min-w-full text-sm">
        <thead>
          <tr class="border-t border-gray-100">
            <th class="py-3 px-4 text-left font-medium text-gray-500">Deal Name</th>
            <th class="py-3 px-4 text-left font-medium text-gray-500">Sales Rep</th>
            <th class="py-3 px-4 text-left font-medium text-gray-500">Amount</th>
            <th class="py-3 px-4 text-left font-medium text-gray-500">Close Date</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="deal in filteredDeals"
            :key="deal.id"
            class="border-t border-gray-100 hover:bg-gray-50 transition-colors"
          >
            <td class="py-3 px-4 text-slate-700 font-medium whitespace-nowrap">
              {{ deal.name }}
            </td>

            <td class="py-3 px-4 whitespace-nowrap">
              <div class="flex items-center gap-3">
                <img
                  :src="deal.avatar"
                  :alt="deal.rep"
                  class="h-8 w-8 rounded-full object-cover"
                />
                <div class="text-slate-800 font-medium">{{ deal.rep }}</div>
              </div>
            </td>

            <td class="py-3 px-4 text-slate-700 whitespace-nowrap">{{ deal.amount }}</td>
            <td class="py-3 px-4 text-slate-600 whitespace-nowrap">{{ deal.date }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const filter = ref('closed')

const deals = ref([
  {
    id: 1,
    name: 'Acme Inc Install',
    rep: 'Donald Risher',
    avatar: '/images/avatar/avatar-01.jpeg',
    amount: '$96k',
    date: 'Today',
    status: 'closed',
  },
  {
    id: 2,
    name: 'Save lots Stores',
    rep: 'Jansh Brown',
    avatar: '/images/avatar/avatar-02.jpeg',
    amount: '$55.7k',
    date: '30 Dec 2021',
    status: 'closed',
  },
  {
    id: 3,
    name: 'William PVT',
    rep: 'Ayaan Hudda',
    avatar: '/images/avatar/avatar-03.jpeg',
    amount: '$102k',
    date: '25 Nov 2021',
    status: 'won',
  },
  {
    id: 4,
    name: 'Raitech Soft',
    rep: 'Julia William',
    avatar: '/images/avatar/avatar-04.jpeg',
    amount: '$89.5k',
    date: '20 Sep 2021',
    status: 'closed',
  },
  {
    id: 5,
    name: 'Absternet LLC',
    rep: 'Vitoria Rodrigues',
    avatar: '/images/avatar/avatar-05.jpeg',
    amount: '$89.5k',
    date: '20 Sep 2021',
    status: 'closed',
  },
])

const filteredDeals = computed(() => {
  if (filter.value === 'closed') {
    return deals.value.filter((d) =>
      ['closed', 'won', 'lost'].includes(d.status)
    )
  }
  return deals.value.filter((d) => d.status === filter.value)
})
</script>

<style scoped>
.custom-scrollbar::-webkit-scrollbar {
  height: 6px;
}
.custom-scrollbar::-webkit-scrollbar-thumb {
  background-color: rgba(0, 0, 0, 0.1);
  border-radius: 4px;
}
</style>
