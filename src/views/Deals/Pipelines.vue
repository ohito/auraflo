<template>
  <AdminLayout>
    <div class="p-6">
      <!-- Page header -->
      <div class="mb-6">
        <div class="bg-white rounded-2xl border border-gray-100 p-4 shadow-sm flex items-center justify-between">
          <div class="flex items-center gap-4">
            <h2 class="text-lg font-semibold">Deals Pipeline</h2>

            <div class="relative w-[420px]">
              <span class="absolute left-3 top-1/2 -translate-y-1/2 text-slate-400">
                <SearchIcon class="w-4 h-4" />
              </span>
              <input v-model="q" type="search" placeholder="Search for deals..." class="w-full h-10 rounded-lg border border-gray-200 px-4 pl-10 text-sm placeholder:text-slate-400 focus:outline-none" />
            </div>

            <div class="text-sm text-slate-500">Sort by:</div>
            <select v-model="sortBy" class="h-9 rounded-lg border border-gray-200 bg-white px-3 text-sm">
              <option value="owner">Owner</option>
              <option value="value_desc">Value (high → low)</option>
              <option value="value_asc">Value (low → high)</option>
              <option value="date">Date</option>
            </select>
          </div>

          <div class="flex items-center gap-3">
            <button @click="openAdd" class="inline-flex items-center gap-2 rounded-lg bg-emerald-500 px-4 py-2 text-white text-sm font-medium hover:bg-emerald-600 shadow">
              + Add Deal
            </button>
            <button @click="resetDemo" class="px-3 py-2 rounded-lg border text-sm">Reset Demo</button>
          </div>
        </div>
      </div>

      <!-- Kanban -->
      <div class="grid grid-cols-1 lg:grid-cols-5 gap-4">
        <div
          v-for="(col, idx) in columns"
          :key="col.key"
          class="bg-transparent"
        >
          <div class="rounded-xl p-4 mb-3"
               :class="['shadow-sm', 'border', 'border-gray-100', 'bg-white']">
            <div class="flex items-center justify-between gap-3">
              <div>
                <div class="text-sm font-semibold text-slate-700">{{ col.title }}</div>
                <div class="text-xs text-slate-400 mt-1">{{ formatCurrency(colTotal(col.key)) }} • {{ colCount(col.key) }} Deals</div>
              </div>
              <div class="text-xs text-slate-400">{{ col.badge }}</div>
            </div>
          </div>

          <!-- drop zone -->
          <div
            class="min-h-[200px] p-2 rounded-xl"
            :class="['bg-slate-50', isDragOver[col.key] ? 'ring-2 ring-sky-300' : '']"
            @dragover.prevent
            @dragenter.prevent="onDragEnter(col.key)"
            @dragleave.prevent="onDragLeave(col.key)"
            @drop.prevent="onDrop(col.key)"
          >
            <div class="space-y-3">
              <div
                v-for="deal in dealsByColumn(col.key)"
                :key="deal.id"
                class="bg-white border border-gray-100 rounded-lg p-3 shadow-sm cursor-grab"
                draggable="true"
                @dragstart="onDragStart(deal.id)"
                @dragend="onDragEnd"
              >
                <div class="flex items-start justify-between gap-3">
                  <div class="flex-1">
                    <div class="flex items-center gap-2">
                      <h4 class="font-medium text-slate-800 truncate">{{ deal.title }}</h4>
                      <span v-if="deal.flag" class="text-xs text-white bg-indigo-600 px-2 py-0.5 rounded">{{ deal.flag }}</span>
                    </div>
                    <div class="text-xs text-slate-500 mt-1">{{ formatCurrency(deal.value) }} • {{ deal.date }}</div>

                    <div class="mt-3 flex items-center gap-3">
                      <img :src="deal.ownerAvatar" class="w-7 h-7 rounded-full object-cover border" />
                      <div class="text-xs text-slate-500">
                        <div class="font-medium text-slate-700">{{ deal.owner }}</div>
                        <div>{{ deal.company }}</div>
                      </div>
                    </div>
                  </div>

                  <div class="text-right">
                    <div class="text-sm font-semibold text-slate-800">{{ shortValue(deal.value) }}</div>
                    <div class="text-xs text-slate-400 mt-2">
                      <button class="px-2 py-1 rounded border text-xs" @click.stop="openDeal(deal)">View</button>
                    </div>
                  </div>
                </div>
              </div>

              <div v-if="dealsByColumn(col.key).length === 0" class="py-6 text-center text-slate-400">No deals</div>
            </div>
          </div>
        </div>
      </div>

      <!-- Add Deal modal -->
      <dialog ref="modal" class="rounded-lg p-0" @close="closeModal">
        <form method="dialog" class="w-[640px] max-w-full bg-white rounded-lg overflow-hidden shadow-lg">
          <div class="p-5 border-b">
            <h3 class="text-lg font-semibold">Create Deal</h3>
          </div>
          <div class="p-5 space-y-3">
            <div class="grid grid-cols-2 gap-3">
              <input v-model="draft.title" placeholder="Deal title" class="border rounded px-3 py-2" />
              <input v-model="draft.company" placeholder="Company" class="border rounded px-3 py-2" />
              <input v-model.number="draft.value" type="number" placeholder="Value (IDR)" class="border rounded px-3 py-2" />
              <select v-model="draft.stage" class="border rounded px-3 py-2">
                <option v-for="c in columns" :key="c.key" :value="c.key">{{ c.title }}</option>
              </select>
            </div>
            <div class="flex items-center gap-3">
              <input v-model="draft.owner" placeholder="Owner name" class="border rounded px-3 py-2 flex-1" />
              <input v-model="draft.date" type="date" class="border rounded px-3 py-2 w-40" />
            </div>
            <textarea v-model="draft.note" placeholder="Note" class="w-full border rounded px-3 py-2"></textarea>
          </div>
          <div class="p-4 border-t flex justify-end gap-2">
            <button type="button" class="px-4 py-2 rounded border" @click="closeModal">Cancel</button>
            <button type="button" class="px-4 py-2 rounded bg-emerald-500 text-white" @click="createDeal">Create</button>
          </div>
        </form>
      </dialog>

      <!-- View Deal modal -->
      <dialog ref="viewModal" class="rounded-lg p-0" @close="closeView">
        <div class="w-[640px] max-w-full bg-white rounded-lg overflow-hidden shadow-lg">
          <div class="p-5 border-b flex items-center justify-between">
            <h3 class="text-lg font-semibold">{{ selectedDeal?.title }}</h3>
            <div class="text-sm text-slate-500">{{ selectedDeal?.stage }}</div>
          </div>

          <div class="p-5 space-y-3">
            <div class="flex items-center gap-4">
              <img :src="selectedDeal?.ownerAvatar" class="w-12 h-12 rounded-full object-cover border" />
              <div>
                <div class="font-medium">{{ selectedDeal?.owner }}</div>
                <div class="text-xs text-slate-500">{{ selectedDeal?.company }}</div>
              </div>
              <div class="ml-auto text-right">
                <div class="text-sm font-semibold">{{ formatCurrency(selectedDeal?.value || 0) }}</div>
                <div class="text-xs text-slate-400">{{ selectedDeal?.date }}</div>
              </div>
            </div>

            <div class="text-sm text-slate-700" v-html="selectedDeal?.note || '<em>No details</em>'"></div>
          </div>

          <div class="p-4 border-t flex justify-end gap-2">
            <button class="px-4 py-2 rounded border" @click="closeView">Close</button>
            <button class="px-4 py-2 rounded bg-sky-600 text-white" @click="markWon(selectedDeal)">Mark Won</button>
          </div>
        </div>
      </dialog>
    </div>
  </AdminLayout>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import AdminLayout from '@/components/layout/AdminLayout.vue'
import SearchIcon from '@/icons/SearchIcon.vue'

/* ---------- columns definition ---------- */
const columns = [
  { key: 'canvassing', title: 'CANVASSING', badge: '' },
  { key: 'qualification', title: 'QUALIFICATION', badge: '' },
  { key: 'quotation', title: 'QUOTATION', badge: '' },
  { key: 'negotiation', title: 'NEGOTIATION', badge: '' },
  { key: 'offer_accepted', title: 'OFFER ACCEPTED', badge: '' },
]

/* ---------- demo deals ---------- */
const initialDeals = [
  { id: 101, title: 'Managing sales team meeting', value: 87000, date: '2022-01-01', owner: 'Anna', ownerAvatar: 'https://i.pravatar.cc/64?img=21', company: 'Nesta Technologies', stage: 'canvassing', flag: '', note: '<p>Details meeting with team</p>' },
  { id: 102, title: 'Airbnb React Development', value: 20300, date: '2021-12-24', owner: 'Budi', ownerAvatar: 'https://i.pravatar.cc/64?img=32', company: 'Startup A', stage: 'canvassing', flag: 'Hot', note: '' },
  { id: 201, title: 'Custom Mobile Apps', value: 28700, date: '2021-12-13', owner: 'Kevin', ownerAvatar: 'https://i.pravatar.cc/64?img=5', company: 'Digital Co', stage: 'qualification', flag: '', note: '' },
  { id: 301, title: 'Art Studio Design', value: 147500, date: '2021-09-24', owner: 'Siti', ownerAvatar: 'https://i.pravatar.cc/64?img=7', company: 'DesignX', stage: 'quotation', flag: '', note: '' },
  { id: 401, title: 'SASS app workflow diagram', value: 17800, date: '2022-01-01', owner: 'James', ownerAvatar: 'https://i.pravatar.cc/64?img=12', company: 'DevWorks', stage: 'negotiation', flag: '', note: '' },
  { id: 501, title: 'Coupon Website', value: 27400, date: '2021-01-07', owner: 'Riana', ownerAvatar: 'https://i.pravatar.cc/64?img=9', company: 'eCom', stage: 'offer_accepted', flag: '', note: '' },
  // more items for demo
  { id: 103, title: 'Discovery Capital', value: 124300, date: '2021-12-29', owner: 'Alex', ownerAvatar: 'https://i.pravatar.cc/64?img=8', company: 'VC Fund', stage: 'canvassing', flag: 'VIP', note: 'Important investor' },
  { id: 302, title: 'Modern Design', value: 23000, date: '2021-10-03', owner: 'Dina', ownerAvatar: 'https://i.pravatar.cc/64?img=14', company: 'BrandCo', stage: 'quotation', flag: '', note: '' },
  { id: 502, title: 'Marketing Automation Demo', value: 94800, date: '2021-11-19', owner: 'Haryo', ownerAvatar: 'https://i.pravatar.cc/64?img=15', company: 'Marcom', stage: 'offer_accepted', flag: '', note: '' },
]

const deals = ref(JSON.parse(JSON.stringify(initialDeals))) // reactive array

/* ---------- UI state ---------- */
const q = ref('')
const sortBy = ref('owner')
const draggingId = ref(null)
const isDragOver = reactive({
  canvassing: false,
  qualification: false,
  quotation: false,
  negotiation: false,
  offer_accepted: false
})

/* modals + draft */
const modal = ref(null)
const viewModal = ref(null)
const draft = reactive({
  title: '',
  company: '',
  value: 0,
  stage: 'canvassing',
  owner: '',
  ownerAvatar: '',
  date: '',
  note: ''
})
const selectedDeal = ref(null)

/* ---------- helpers ---------- */
function dealsByColumn(stageKey) {
  const list = deals.value.filter(d => d.stage === stageKey && matchesQuery(d))
  // sorting
  if (sortBy.value === 'value_desc') return list.sort((a,b) => b.value - a.value)
  if (sortBy.value === 'value_asc') return list.sort((a,b) => a.value - b.value)
  if (sortBy.value === 'date') return list.sort((a,b) => new Date(b.date) - new Date(a.date))
  return list.sort((a,b) => a.owner.localeCompare(b.owner))
}

function matchesQuery(d) {
  const s = q.value.trim().toLowerCase()
  if (!s) return true
  return (d.title && d.title.toLowerCase().includes(s)) ||
         (d.company && d.company.toLowerCase().includes(s)) ||
         (d.owner && d.owner.toLowerCase().includes(s))
}

function colTotal(stageKey) {
  return deals.value.filter(d => d.stage === stageKey).reduce((s, d) => s + (Number(d.value)||0), 0)
}
function colCount(stageKey) {
  return deals.value.filter(d => d.stage === stageKey).length
}

function formatCurrency(val = 0) {
  // show in IDR style with suffix k for thousands (basic)
  if (val >= 1000) {
    // show as 147.5k etc (thousands)
    const v = Math.round(val / 100) / 10 // one decimal in thousands e.g., 147.5
    return `Rp ${v}k`
  }
  return `Rp ${val}`
}
function shortValue(val = 0) {
  if (val >= 1000000) return `${Math.round(val/1000000)}M`
  if (val >= 1000) return `${Math.round(val/1000)}k`
  return `Rp ${val}`
}

/* ---------- drag & drop ---------- */
function onDragStart(id) {
  draggingId.value = id
}
function onDragEnd() {
  draggingId.value = null
  // clear dragOver states
  Object.keys(isDragOver).forEach(k => isDragOver[k] = false)
}
function onDragEnter(colKey) {
  isDragOver[colKey] = true
}
function onDragLeave(colKey) {
  isDragOver[colKey] = false
}
function onDrop(colKey) {
  if (!draggingId.value) return
  const idx = deals.value.findIndex(d => d.id === draggingId.value)
  if (idx === -1) return
  deals.value[idx].stage = colKey
  // optional: change date to today
  // deals.value[idx].date = new Date().toISOString().slice(0,10)
  onDragEnd()
}

/* ---------- create / view ---------- */
function openAdd() {
  draft.title = ''
  draft.company = ''
  draft.value = 0
  draft.stage = 'canvassing'
  draft.owner = ''
  draft.ownerAvatar = `https://i.pravatar.cc/64?img=${Math.floor(Math.random()*40)+1}`
  draft.date = new Date().toISOString().slice(0,10)
  draft.note = ''
  modal.value?.showModal?.()
}
function closeModal() { modal.value?.close?.() }

function createDeal() {
  if (!draft.title || !draft.company) {
    alert('Title & company required')
    return
  }
  const id = Math.floor(Math.random() * 1000000)
  deals.value.unshift({
    id,
    title: draft.title,
    company: draft.company,
    value: Number(draft.value) || 0,
    date: draft.date,
    owner: draft.owner || 'Unassigned',
    ownerAvatar: draft.ownerAvatar,
    stage: draft.stage,
    flag: '',
    note: draft.note
  })
  closeModal()
}

function openDeal(deal) {
  selectedDeal.value = deal
  viewModal.value?.showModal?.()
}
function closeView() { viewModal.value?.close?.() }

function markWon(deal) {
  if (!deal) return
  deal.stage = 'offer_accepted'
  alert(`Deal "${deal.title}" marked as Won.`)
  closeView()
}

/* demo helpers */
function resetDemo() {
  deals.value = JSON.parse(JSON.stringify(initialDeals))
}

/* mount */
onMounted(() => {
  // nothing special; refs bound in template
})
</script>

<style scoped>
/* tweak scrollbar inside columns */
.max-h-96::-webkit-scrollbar { height: 6px; width: 6px; }
</style>
