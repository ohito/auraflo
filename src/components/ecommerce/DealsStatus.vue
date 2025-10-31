<template>
  <div class="rounded-2xl border border-gray-200 bg-white p-5 w-full h-full flex flex-col dark:border-gray-800 dark:bg-white/[0.03]">
    <!-- Header -->
    <div class="flex items-center justify-between mb-4">
      <h3 class="text-lg font-semibold text-gray-800 dark:text-white/90">Deals Status</h3>

      <div class="flex items-center gap-3">
        <label class="sr-only" for="range">Range</label>
        <select
          id="range"
          v-model="range"
          class="text-sm rounded-md border border-gray-100 bg-white py-2 px-3 shadow-sm hover:border-gray-200 focus:outline-none focus:ring-1 focus:ring-brand-400"
        >
          <option value="nov2021">02 Nov 2021 to 31 Dec 2021</option>
          <option value="2021">2021</option>
          <option value="last30">Last 30 days</option>
        </select>
      </div>
    </div>

    <!-- Table (growable) -->
    <div class="overflow-x-auto flex-1">
      <table class="min-w-full divide-y divide-gray-100 text-left">
        <thead class="bg-gray-50">
          <tr>
            <th class="px-4 py-3 text-xs font-semibold text-gray-500 uppercase">Name</th>
            <th class="px-4 py-3 text-xs font-semibold text-gray-500 uppercase">Last Contacted</th>
            <th class="px-4 py-3 text-xs font-semibold text-gray-500 uppercase">Sales Representative</th>
            <th class="px-4 py-3 text-xs font-semibold text-gray-500 uppercase">Status</th>
            <th class="px-4 py-3 text-xs font-semibold text-gray-500 uppercase text-right">Deal Value</th>
          </tr>
        </thead>

        <tbody class="divide-y divide-gray-100">
          <tr v-for="(d, idx) in deals" :key="d.id" class="hover:bg-gray-50">
            <!-- Name -->
            <td class="px-4 py-4 whitespace-nowrap align-top">
              <div class="text-sm font-medium text-gray-900">{{ d.name }}</div>
              <div class="text-xs text-gray-400 mt-1">{{ d.company ?? '' }}</div>
            </td>

            <!-- Last Contacted -->
            <td class="px-4 py-4 whitespace-nowrap align-top">
              <div class="text-sm text-gray-600">{{ d.last_contacted }}</div>
            </td>

            <!-- Sales Representative -->
            <td class="px-4 py-4 whitespace-nowrap align-top">
              <div class="flex items-center gap-3">
                <img
                  :src="d.rep.avatar"
                  :alt="d.rep.name"
                  class="w-8 h-8 rounded-full object-cover border border-gray-100"
                />
                <div>
                  <div class="text-sm font-medium text-gray-800">{{ d.rep.name }}</div>
                  <div class="text-xs text-gray-400">{{ d.rep.role }}</div>
                </div>
              </div>
            </td>

            <!-- Status -->
            <td class="px-4 py-4 whitespace-nowrap align-top">
              <span
                :class="statusClass(d.status)"
                class="inline-flex items-center rounded-full px-3 py-1 text-xs font-semibold"
              >
                {{ d.status }}
              </span>
            </td>

            <!-- Deal Value -->
            <td class="px-4 py-4 whitespace-nowrap align-top text-right">
              <div class="text-sm font-semibold text-gray-800">{{ formatMoney(d.value) }}</div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- footer: optional pagination -->
    <div class="mt-4 flex items-center justify-between">
      <div class="text-sm text-gray-500">Showing {{ deals.length }} deals</div>
      <div class="flex items-center gap-2">
        <button class="px-3 py-1 rounded-md border border-gray-100 text-sm text-gray-600 hover:bg-gray-50">Prev</button>
        <button class="px-3 py-1 rounded-md border border-gray-100 text-sm text-gray-600 hover:bg-gray-50">Next</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const range = ref('nov2021')

const deals = ref([
  {
    id: 1,
    name: 'Absternet LLC',
    company: '',
    last_contacted: 'Sep 20, 2021',
    rep: { name: 'Donald Risher', role: '', avatar: 'https://i.pravatar.cc/80?img=15' },
    status: 'Deal Won',
    value: 100000,
  },
  {
    id: 2,
    name: 'Raitech Soft',
    company: '',
    last_contacted: 'Sep 23, 2021',
    rep: { name: 'Sofia Cunha', role: '', avatar: 'https://i.pravatar.cc/80?img=32' },
    status: 'Intro Call',
    value: 150000,
  },
  {
    id: 3,
    name: 'William PVT',
    company: '',
    last_contacted: 'Sep 27, 2021',
    rep: { name: 'Luis Rocha', role: '', avatar: 'https://i.pravatar.cc/80?img=12' },
    status: 'Stuck',
    value: 78180,
  },
  {
    id: 4,
    name: 'Loiusee LLP',
    company: '',
    last_contacted: 'Sep 30, 2021',
    rep: { name: 'Vitoria Rodrigues', role: '', avatar: 'https://i.pravatar.cc/80?img=47' },
    status: 'Deal Won',
    value: 180000,
  },
  {
    id: 5,
    name: 'Apple Inc.',
    company: '',
    last_contacted: 'Sep 30, 2021',
    rep: { name: 'Vitoria Rodrigues', role: '', avatar: 'https://i.pravatar.cc/80?img=11' },
    status: 'New Lead',
    value: 78900,
  },
])

function formatMoney(n) {
  if (n >= 1000) {
    const k = Math.round(n / 1000)
    return `$${k.toLocaleString()}k`
  }
  return `$${n}`
}

function statusClass(status) {
  switch (status) {
    case 'Deal Won':
      return 'bg-emerald-100 text-emerald-700'
    case 'Intro Call':
      return 'bg-amber-100 text-amber-700'
    case 'Stuck':
      return 'bg-red-100 text-red-700'
    case 'New Lead':
      return 'bg-sky-100 text-sky-700'
    default:
      return 'bg-gray-100 text-gray-700'
  }
}
</script>

<style scoped>
select:focus {
  outline: none;
  box-shadow: 0 0 0 3px rgba(70, 95, 255, 0.08);
}
</style>
