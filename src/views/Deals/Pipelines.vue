<template>
  <AdminLayout>
    <div class="p-6">
      <!-- HEADER: title | search | sort | actions -->
      <div class="mb-6">
        <div class="header-card grid grid-cols-3 items-center gap-4 px-5 py-4">
          <!-- title -->
          <div class="col-span-1">
            <h2 class="text-2xl font-semibold">Deals Pipeline</h2>
          </div>

          <!-- search + sort (center) -->
          <div class="col-span-1 flex items-center justify-center">
            <div class="relative w-full max-w-[720px]">
              <span class="absolute left-4 top-1/2 -translate-y-1/2 text-slate-400">
                <SearchIcon class="w-4 h-4" />
              </span>
              <input
                v-model="q"
                type="search"
                placeholder="Search for deals..."
                class="w-full h-11 rounded-lg border border-gray-200 px-4 pl-11 text-sm placeholder:text-slate-400 focus:outline-none"
              />
            </div>

            <div class="ml-4 hidden md:flex items-center gap-2">
              <div class="text-sm text-slate-500">Sort by:</div>
              <select v-model="sortBy" class="h-9 rounded-lg border border-gray-200 bg-white px-3 text-sm">
                <option value="owner">Owner</option>
                <option value="value_desc">Value (high → low)</option>
                <option value="value_asc">Value (low → high)</option>
                <option value="date">Date</option>
              </select>
            </div>
          </div>

          <!-- actions -->
          <div class="col-span-1 flex items-center justify-end gap-3">
            <button @click="openAdd" class="btn-primary">+ Add Deal</button>
            <button @click="resetDemo" class="btn-ghost">Reset Demo</button>
          </div>
        </div>
      </div>

      <!-- KANBAN GRID (no horizontal scroll by default) -->
      <div class="kanban-wrap-grid">
        <div class="kanban-grid" :style="`--cols: ${columns.length}`">
          <div v-for="col in columns" :key="col.key" class="column">
            <!-- column header -->
            <div class="col-header" :style="{ background: col.bg, color: col.text }">
              <div>
                <div class="col-title" :style="{ color: col.text }">{{ col.title }}</div>
                <div class="col-sub" :style="{ color: col.text }">{{ formatCurrency(colTotal(col.key)) }} • {{ colCount(col.key) }} Deals</div>
              </div>
            </div>

            <!-- draggable list for each column -->
            <draggable
              v-model="columnsMap[col.key]"
              group="deals"
              item-key="id"
              :animation="200"
              class="col-list"
              @change="onDragChange"
            >
              <template #item="{ element: deal }">
                <div class="card" :key="deal.id">
                  <!-- header area (click to toggle expand, dblclick to open modal) -->
                  <div
                    class="card-body"
                    @click="openDeal(deal)"
                    @dblclick.stop="openViewModal(deal)"
                    role="button"
                    :aria-expanded="expandedId === deal.id"
                  >
                    <div class="card-left">
                      <h4 class="card-title" :title="deal.title">{{ deal.title }}</h4>
                      <div class="card-meta">
                        <div class="meta-left">{{ formatCurrency(deal.value) }} • {{ deal.date }}</div>
                        <div class="meta-right">{{ shortValue(deal.value) }}</div>
                      </div>

                      <div class="card-owner mt-3">
                        <img :src="deal.ownerAvatar" class="owner-avatar" />
                        <div class="owner-info">
                          <div class="owner-name">{{ deal.owner }}</div>
                          <div class="owner-company">{{ deal.company }}</div>
                        </div>
                      </div>
                    </div>

                    <div class="card-action">
                      <button class="view-btn" @click.stop="openViewModal(deal)">View</button>
                    </div>
                  </div>

                  <!-- expanded area (visible when expandedId matches) -->
                  <div v-if="expandedId === deal.id" class="card-expanded">
                    <div class="expanded-company">
                      <div class="company-name">{{ deal.company }}</div>
                      <span class="badge-days">4 Days</span>
                    </div>

                    <div class="expanded-desc" v-html="getDescription(deal)"></div>

                    <ul class="expanded-activities">
                      <li class="activity-item">
                        <div class="activity-title">Meeting with Thomas</div>
                        <div class="activity-meta">Yesterday at 9:12AM</div>
                      </li>
                      <li class="activity-item">
                        <div class="activity-title">Product Demo</div>
                        <div class="activity-meta">Monday at 04:41PM</div>
                      </li>
                      <li class="activity-item">
                        <div class="activity-title">Marketing Team Meeting</div>
                        <div class="activity-meta">Monday at 04:41PM</div>
                      </li>
                    </ul>

                    <div class="expanded-actions">
                      <a :href="`tel:${deal.phone || ''}`" class="btn-call">📞 Call</a>
                      <button class="btn-message" @click.stop="onMessage(deal)">✉️ Message</button>
                    </div>
                  </div>
                </div>
              </template>

              <template #footer>
                <div v-if="columnsMap[col.key].length === 0" class="col-empty">No deals</div>
              </template>
            </draggable>
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
import { ref, reactive, onMounted } from 'vue'
import draggable from 'vuedraggable'
import AdminLayout from '@/components/layout/AdminLayout.vue'
import SearchIcon from '@/icons/SearchIcon.vue'


function getDescription(deal) {
  // jika deal.note ada, pakai itu (anggap sudah HTML-safe)
  if (deal && deal.note) return deal.note
  // default string — tulis di sini bebas pakai apostrof
  return `As a company grows however, you find it's not as easy to shout across`
}


/* ---------- columns definition ---------- */
const columns = [
  { key: 'canvassing', title: 'CANVASSING', bg: '#fff1f0', text: '#1f2937' },         // pink pastel
  { key: 'qualification', title: 'QUALIFICATION', bg: '#f0fdf4', text: '#1f2937' },   // mint pastel
  { key: 'quotation', title: 'QUOTATION', bg: '#fffbf0', text: '#1f2937' },           // cream pastel
  { key: 'negotiation', title: 'NEGOTIATION', bg: '#f3f8ff', text: '#1f2937' },       // light-blue pastel
  { key: 'offer_accepted', title: 'OFFER ACCEPTED', bg: '#f5f3ff', text: '#1f2937' }, // lilac pastel
]

/* ---------- demo deals ---------- */
const initialDeals = [
  { id: 101, title: 'Managing sales team meeting', value: 87000, date: '2022-01-01', owner: 'Anna', ownerAvatar: 'https://i.pravatar.cc/64?img=21', company: 'Nesta Technologies', stage: 'canvassing', flag: '', note: '<p>Details meeting with team</p>' },
  { id: 102, title: 'Airbnb React Development', value: 20300, date: '2021-12-24', owner: 'Budi', ownerAvatar: 'https://i.pravatar.cc/64?img=32', company: 'Startup A', stage: 'canvassing', flag: 'Hot', note: '' },
  { id: 103, title: 'Discovery Capital', value: 124300, date: '2021-12-29', owner: 'Alex', ownerAvatar: 'https://i.pravatar.cc/64?img=8', company: 'VC Fund', stage: 'canvassing', flag: 'VIP', note: 'Important investor' },

  { id: 201, title: 'Custom Mobile Apps', value: 28700, date: '2021-12-13', owner: 'Kevin', ownerAvatar: 'https://i.pravatar.cc/64?img=5', company: 'Digital Co', stage: 'qualification', flag: '', note: '' },
  { id: 202, title: 'Managing sales team meeting', value: 87000, date: '2022-01-01', owner: 'Anna', ownerAvatar: 'https://i.pravatar.cc/64?img=21', company: 'Nesta Technologies', stage: 'qualification', flag: '', note: '' },

  { id: 301, title: 'Art Studio Design', value: 147500, date: '2021-09-24', owner: 'Siti', ownerAvatar: 'https://i.pravatar.cc/64?img=7', company: 'DesignX', stage: 'quotation', flag: '', note: '' },
  { id: 302, title: 'Modern Design', value: 23000, date: '2021-10-03', owner: 'Dina', ownerAvatar: 'https://i.pravatar.cc/64?img=14', company: 'BrandCo', stage: 'quotation', flag: '', note: '' },

  { id: 401, title: 'SASS app workflow diagram', value: 17800, date: '2022-01-01', owner: 'James', ownerAvatar: 'https://i.pravatar.cc/64?img=12', company: 'DevWorks', stage: 'negotiation', flag: '', note: '' },

  { id: 501, title: 'Coupon Website', value: 27400, date: '2021-01-07', owner: 'Riana', ownerAvatar: 'https://i.pravatar.cc/64?img=9', company: 'eCom', stage: 'offer_accepted', flag: '', note: '' },
  { id: 502, title: 'Marketing Automation Demo', value: 94800, date: '2021-11-19', owner: 'Haryo', ownerAvatar: 'https://i.pravatar.cc/64?img=15', company: 'Marcom', stage: 'offer_accepted', flag: '', note: '' },
]

/* reactive deals store */
const deals = ref(JSON.parse(JSON.stringify(initialDeals)))

/* columnsMap: stageKey -> array (vuedraggable v-model) */
const columnsMap = reactive({})
function rebuildColumnsMap() {
  for (const c of columns) {
    columnsMap[c.key] = deals.value.filter(d => d.stage === c.key)
  }
}
rebuildColumnsMap()

/* UI state */
const q = ref('')
const sortBy = ref('owner')

/* modals & draft */
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

/* expanded inline card state */
const expandedId = ref(null)

/* helpers */
function colTotal(stageKey) {
  return deals.value.filter(d => d.stage === stageKey).reduce((s,d) => s + (Number(d.value)||0), 0)
}
function colCount(stageKey) {
  return deals.value.filter(d => d.stage === stageKey).length
}
function formatCurrency(val = 0) {
  if (!val) return 'Rp 0'
  if (val >= 1000) {
    const v = Math.round(val / 100) / 10
    return `Rp ${v}k`
  }
  return `Rp ${val}`
}
function shortValue(val = 0) {
  if (val >= 1000000) return `${Math.round(val/1000000)}M`
  if (val >= 1000) return `${Math.round(val/1000)}k`
  return `Rp ${val}`
}

/* draggable change: rebuild master list and update stages */
function onDragChange(evt) {
  const updated = []
  for (const c of columns) {
    const arr = columnsMap[c.key] || []
    for (const item of arr) {
      item.stage = c.key
      updated.push(item)
    }
  }
  // append any missing items
  const others = deals.value.filter(d => !updated.find(u => u.id === d.id))
  deals.value = updated.concat(others)
  rebuildColumnsMap()
  // TODO: persist to API here (PATCH /deals/:id)
}

/* create / view / misc */
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
  const newDeal = {
    id, title: draft.title, company: draft.company,
    value: Number(draft.value)||0, date: draft.date,
    owner: draft.owner || 'Unassigned', ownerAvatar: draft.ownerAvatar,
    stage: draft.stage, flag: '', note: draft.note
  }
  deals.value.unshift(newDeal)
  rebuildColumnsMap()
  closeModal()
}

/* expand inline (single-click) */
function openDeal(deal) {
  if (!deal) { expandedId.value = null; return }
  expandedId.value = expandedId.value === deal.id ? null : deal.id
}

/* modal view (double-click or View button) */
function openViewModal(deal) {
  selectedDeal.value = deal
  viewModal.value?.showModal?.()
}
function closeView() { viewModal.value?.close?.() }

function markWon(deal) {
  if (!deal) return
  deal.stage = 'offer_accepted'
  rebuildColumnsMap()
  closeView()
}

function resetDemo() {
  deals.value = JSON.parse(JSON.stringify(initialDeals))
  rebuildColumnsMap()
}

// example message action
function onMessage(deal) {
  alert(`Open message composer to ${deal.owner || 'Owner'}`)
}

onMounted(() => {
  rebuildColumnsMap()
})
</script>

<style scoped>
/* theme buttons */
.btn-primary { background:#10b981; color:white; padding:.6rem .9rem; border-radius:.6rem; font-weight:600; box-shadow:0 1px 0 rgba(0,0,0,0.04); }
.btn-ghost { background:white; padding:.5rem .8rem; border-radius:.6rem; border:1px solid #e6e9ee; }

/* header card */
.header-card {
  background: #ffffff;
  border-radius: 14px;
  border: 1px solid #eef1f6;
  box-shadow: 0 6px 18px rgba(28, 42, 66, 0.04);
}

/* ----- GRID LAYOUT (no horizontal scroll) ----- */
.kanban-wrap-grid {
  width: 100%;
  padding-bottom: 12px;
  overflow: visible;
}

/* dynamic columns based on --cols */
.kanban-grid {
  display: grid;
  gap: 20px;
  grid-template-columns: repeat(var(--cols), minmax(0, 1fr));
  align-items: start;
  width: 100%;
}

/* column */
.column {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

/* column header - pill */
.col-header {
  border-radius: 10px;
  padding: 14px;
  border: 1px solid rgba(15,23,42,0.04); /* very subtle border */
  box-shadow: 0 6px 18px rgba(28,42,66,0.03);
  /* remove fixed white bg so inline bg works */
}

/* make title bigger & uppercase like screenshot */
.col-title {
  font-size: 14px;
  font-weight: 800;
  letter-spacing: 0.6px;
  text-transform: uppercase;
}

/* subtitle lighter but slightly darker than pastel */
.col-sub {
  font-size: 13px;
  margin-top: 8px;
  opacity: 0.75;
}


/* list area inside column */
.col-list {
  min-height: 120px;
  background: transparent;
  padding: 6px;
  display:flex;
  flex-direction:column;
  gap:12px;
}

/* card */
.card {
  width: 100%;
  background:white;
  border-radius:12px;
  padding:12px;
  border: 1px solid #eef1f6;
  box-shadow: 0 6px 18px rgba(28, 42, 66, 0.03);
  cursor: grab;
}
.card-body { display:flex; gap:8px; align-items:flex-start; }
.card-left { flex:1; min-width:0; }
.card-title { font-weight:700; color:#0f172a; font-size:15px; line-height:1.1; max-width:100%; overflow:hidden; text-overflow:ellipsis; white-space:nowrap; }
.card-meta { display:flex; justify-content:space-between; align-items:center; gap:8px; margin-top:6px; color:#6b7280; font-size:12px; }
.meta-right { font-weight:700; color:#0f172a; font-size:12px; }

/* owner row */
.card-owner { display:flex; align-items:center; gap:8px; margin-top:8px; }
.owner-avatar { width:34px; height:34px; border-radius:9999px; object-fit:cover; border:1px solid #f1f5f9; }
.owner-info { font-size:12px; color:#6b7280; }
.owner-name { font-weight:600; color:#0f172a; font-size:13px; }
.owner-company { font-size:12px; color:#94a3b8; }

/* action area */
.card-action { display:flex; align-items:center; margin-left:8px; }
.view-btn { border: 1px solid #e6e9ee; background:white; color:#374151; padding:6px 8px; border-radius:8px; font-size:12px; }

/* empty area message */
.col-empty { color:#9aa4b2; text-align:center; padding:14px 0; }

/* expanded inline area */
.card-expanded {
  border-top: 1px dashed rgba(15,23,42,0.06);
  padding: 14px;
  background: #fff;
  margin-top: 10px;
  border-radius: 8px;
}

.expanded-company {
  display:flex;
  align-items:center;
  gap:12px;
  margin-bottom:8px;
}
.company-name { font-weight:700; color:#0f172a; }
.badge-days {
  background:#fde68a;
  color:#92400e;
  padding:4px 8px;
  border-radius:8px;
  font-size:12px;
  font-weight:700;
}

.expanded-desc {
  color:#6b7280;
  margin:10px 0 12px 0;
  font-size:13px;
  line-height:1.45;
}

.expanded-activities { list-style:none; padding:0; margin:0; border-top:1px dashed rgba(15,23,42,0.04); }
.expanded-activities .activity-item { padding:12px 0; border-bottom:1px dashed rgba(15,23,42,0.04); }
.activity-title { font-weight:700; color:#0f172a; }
.activity-meta { font-size:12px; color:#6b7280; margin-top:6px; }

.expanded-actions { display:flex; gap:10px; margin-top:12px; }
.btn-call { background:#f59e0b; color:#fff; padding:8px 12px; border-radius:8px; font-weight:700; text-decoration:none; }
.btn-message { background:#3b82f6; color:#fff; padding:8px 12px; border-radius:8px; border:none; cursor:pointer; font-weight:700; }

/* responsive: on small screens, stack columns vertically */
@media (max-width: 900px) {
  .kanban-grid {
    grid-template-columns: repeat(1, minmax(0, 1fr));
  }
  .header-card { grid-template-columns: 1fr; gap:8px; }
}
</style>
