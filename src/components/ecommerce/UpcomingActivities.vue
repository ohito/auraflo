<template>
  <div class="overflow-hidden rounded-2xl border border-gray-200 bg-white p-5 sm:p-6">
    <div class="flex items-start justify-between mb-4">
      <h3 class="text-lg font-semibold text-slate-800">Upcoming Activities</h3>
      <button class="p-1 rounded-md hover:bg-slate-50" title="More">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 text-slate-400" viewBox="0 0 20 20" fill="currentColor">
          <path d="M6 10a2 2 0 11-4 0 2 2 0 014 0zm6 0a2 2 0 11-4 0 2 2 0 014 0zm6 0a2 2 0 11-4 0 2 2 0 014 0z" />
        </svg>
      </button>
    </div>

    <ul class="divide-y divide-dashed divide-slate-200">
      <li v-for="(act, i) in pageItems" :key="act.id" class="py-4 flex items-start justify-between">
        <div class="flex items-start gap-4">
          <!-- date box -->
          <div class="flex flex-col items-center justify-center w-12 h-12 rounded-md bg-slate-50 text-center text-sm text-slate-600">
            <div class="font-semibold text-slate-800">{{ day(act.date) }}</div>
            <div class="text-xs">{{ monthShort(act.date) }}</div>
          </div>

          <div>
            <div class="text-xs text-slate-400">{{ formatTimeRange(act.start, act.end) }}</div>
            <div class="text-sm font-medium text-slate-800">{{ act.title }}</div>
          </div>
        </div>

        <div class="flex -space-x-2">
          <img
            v-for="(a, idx) in visibleAvatars(act.participants)"
            :key="a.id"
            :src="a.avatar"
            :alt="a.name"
            class="h-7 w-7 rounded-full border-2 border-white shadow-xs"
          />
        </div>
      </li>
    </ul>

    <div class="mt-4 flex items-center justify-between">
      <div class="text-sm text-slate-400">Showing {{ pageStart }} of {{ total }} Results</div>

      <div class="flex items-center gap-2">
        <button @click="prevPage" :disabled="page===1" class="px-3 py-1 rounded border border-gray-200 text-sm disabled:opacity-50">&larr;</button>
        <button v-for="p in pages" :key="p" @click="page = p" :class="['px-3 py-1 rounded text-sm', page===p ? 'bg-slate-800 text-white' : 'border border-gray-200']">{{ p }}</button>
        <button @click="nextPage" :disabled="page===totalPages" class="px-3 py-1 rounded border border-gray-200 text-sm disabled:opacity-50">&rarr;</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const itemsPerPage = 4
const maxAvatars = 4

const activities = ref([
  { id: 1, date: '2021-10-25', start: '00:00', end: '15:30', title: 'Meeting for campaign with sales team', participants: sampleParticipants(5) },
  { id: 2, date: '2021-10-20', start: '14:00', end: '15:45', title: 'Adding a new event with attachments', participants: sampleParticipants(3) },
  { id: 3, date: '2021-10-17', start: '16:30', end: '19:15', title: 'Create new project Bundling Product', participants: sampleParticipants(4) },
  { id: 4, date: '2021-10-12', start: '10:30', end: '13:15', title: 'Weekly closed sales won checking with sales team', participants: sampleParticipants(9) },
  { id: 5, date: '2021-10-05', start: '09:00', end: '10:00', title: 'Client onboarding call', participants: sampleParticipants(2) },
  { id: 6, date: '2021-10-02', start: '13:00', end: '14:00', title: 'Design handoff review', participants: sampleParticipants(3) },
])

const page = ref(1)

const total = computed(() => activities.value.length)
const totalPages = computed(() => Math.max(1, Math.ceil(total.value / itemsPerPage)))

const pageItems = computed(() => {
  const start = (page.value - 1) * itemsPerPage
  return activities.value.slice(start, start + itemsPerPage)
})

const pageStart = computed(() => Math.min(page.value * itemsPerPage, total.value))
const pages = computed(() => {
  const arr = []
  for (let i=1;i<=totalPages.value;i++) arr.push(i)
  return arr
})

function prevPage(){ if(page.value>1) page.value-- }
function nextPage(){ if(page.value<totalPages.value) page.value++ }

function formatTimeRange(s,e){
  // assume HH:mm input -> convert to friendly like 12:00am
  return `${to12(s)} - ${to12(e)}`
}
function to12(hm){
  const [hh,mm] = hm.split(':').map(Number)
  const am = hh < 12
  const hh12 = ((hh + 11) % 12) + 1
  const suffix = am ? 'am' : 'pm'
  return `${String(hh12).padStart(2,'0')}:${String(mm).padStart(2,'0')}${suffix}`
}

function day(d){ return new Date(d).getDate() }
function monthShort(d){ return new Date(d).toLocaleString(undefined,{month:'short'}) }

function visibleAvatars(list){ return list.slice(0, maxAvatars) }

function extraCountBg(index){
  // pick a background color class for the extra badge
  const colors = ['bg-rose-500','bg-emerald-500','bg-indigo-600','bg-yellow-500']
  return colors[index % colors.length] + ' text-white'
}

function sampleParticipants(n){
  const avatars = [
    '/images/avatar/avatar-01.jpeg','/images/avatar/avatar-02.jpeg','/images/avatar/avatar-03.jpeg','/images/avatar/avatar-04.jpeg','/images/avatar/avatar-05.jpeg'
  ]
  const res = []
  for(let i=0;i<n;i++) res.push({ id: i+1, name: 'User '+(i+1), avatar: avatars[i % avatars.length] })
  return res
}
</script>

<style scoped>
/* keep avatars overlapping */
</style>
