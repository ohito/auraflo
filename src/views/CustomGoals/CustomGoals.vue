<template>
  <AdminLayout>
    <div class="p-6">
      <!-- HEADER -->
      <div class="mb-6">
        <div class="bg-white rounded-2xl border border-gray-200 p-4 shadow-sm flex flex-col md:flex-row items-center justify-between gap-4">
          <div class="flex items-center gap-4 w-full md:w-auto min-w-0">
            <h2 class="text-2xl font-semibold whitespace-nowrap">Custom Goals</h2>

            <div class="relative flex-1 min-w-0 md:w-[540px]">
              <span class="absolute left-3 top-1/2 -translate-y-1/2 text-slate-400">
                <SearchIcon class="w-4 h-4" />
              </span>
              <input
                v-model="q"
                type="search"
                placeholder="Search goals by title or owner..."
                class="w-full h-10 rounded-xl border border-gray-200 px-4 pl-10 text-sm placeholder:text-slate-400 focus:outline-none"
              />
            </div>
          </div>

          <div class="flex items-center gap-3 w-full md:w-auto">
            <select v-model="statusFilter" class="h-9 rounded-xl border border-gray-200 bg-white px-3 text-sm">
              <option value="">All status</option>
              <option value="active">Active</option>
              <option value="paused">Paused</option>
              <option value="completed">Completed</option>
            </select>

            <button @click="openAdd" class="inline-flex items-center gap-2 rounded-lg bg-sky-500 px-4 py-2 text-white text-sm font-medium hover:bg-sky-600 shadow">+ Create Goal</button>
          </div>
        </div>
      </div>

      <!-- GOALS GRID -->
      <div class="grid gap-6">
        <div class="bg-white rounded-2xl border border-gray-200 p-4 shadow-sm">
          <div class="flex items-center justify-between mb-4">
            <div class="text-sm text-slate-600">Showing {{ filtered.length }} results</div>
            <div class="flex items-center gap-2">
              <button class="px-3 py-1 rounded border text-sm" @click="bulkMarkComplete" :disabled="selected.size===0">Mark Complete</button>
              <button class="px-3 py-1 rounded border text-sm text-rose-600" @click="bulkDelete" :disabled="selected.size===0">Delete</button>
            </div>
          </div>

          <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
            <div v-for="goal in paged" :key="goal.id" class="goal-card relative bg-white border rounded-lg p-4 shadow-sm">
              <div class="flex items-start gap-3">
                <input type="checkbox" :checked="isSelected(goal.id)" @click.stop="toggleSelect(goal.id)" class="mt-1" />
                <div class="flex-1 min-w-0">
                  <div class="flex items-start justify-between gap-3">
                    <div>
                      <div class="font-semibold text-lg truncate" :title="goal.title">{{ goal.title }}</div>
                      <div class="text-xs text-slate-500 mt-1 truncate">{{ goal.owner }} • {{ goal.type }}</div>
                    </div>
                    <div class="text-sm text-slate-700 font-semibold">{{ goalProgressPercent(goal) }}%</div>
                  </div>

                  <div class="mt-3">
                    <div class="w-full bg-slate-100 rounded-full h-2 overflow-hidden">
                      <div class="h-2 rounded-full" :style="{ width: goalProgressPercent(goal) + '%', background: progressColor(goal) }"></div>
                    </div>
                    <div class="mt-2 text-xs text-slate-500 flex items-center justify-between">
                      <span>{{ formatCurrency(goal.target) }} target</span>
                      <span>Due {{ goal.due || '—' }}</span>
                    </div>
                  </div>

                  <div class="mt-3 flex items-center gap-2">
                    <button class="px-3 py-1 rounded border text-sm" @click="openDetail(goal)">View</button>
                    <button class="px-3 py-1 rounded border text-sm" @click="openEdit(goal)">Edit</button>
                  </div>
                </div>
              </div>

              <!-- ribbon / flag -->
              <div v-if="goal.flag" class="absolute left-0 -top-3">
                <span class="bg-rose-500 text-white text-xs px-2 py-1 rounded-bl-md">{{ goal.flag }}</span>
              </div>
            </div>
          </div>

          <!-- pagination -->
          <div class="mt-4 flex items-center justify-between">
            <div class="text-sm text-slate-500">Page {{ page }} / {{ totalPages }}</div>
            <div class="flex items-center gap-2">
              <button @click="prevPage" :disabled="page===1" class="px-3 py-1 rounded border text-sm disabled:opacity-50">Prev</button>
              <button v-for="p in pages" :key="p" @click="page = p" :class="['px-3 py-1 rounded text-sm', page === p ? 'bg-slate-800 text-white' : 'border']">{{ p }}</button>
              <button @click="nextPage" :disabled="page===totalPages" class="px-3 py-1 rounded border text-sm disabled:opacity-50">Next</button>
            </div>
          </div>
        </div>
      </div>

      <!-- Create / Edit Modal -->
      <dialog ref="modal" class="rounded-lg p-0" @close="closeModal">
        <form method="dialog" class="w-[680px] max-w-full bg-white rounded-lg overflow-hidden shadow-lg">
          <div class="p-5 border-b flex items-center justify-between">
            <h3 class="text-lg font-semibold">{{ editMode ? 'Edit Goal' : 'Create Goal' }}</h3>
            <div class="text-sm text-slate-400">{{ form.id ? 'ID: ' + form.id : '' }}</div>
          </div>

          <div class="p-5 space-y-3">
            <input v-model="form.title" placeholder="Goal title" class="w-full border rounded px-3 py-2" />

            <div class="grid grid-cols-2 gap-3">
              <select v-model="form.type" class="border rounded px-3 py-2">
                <option value="Revenue">Revenue</option>
                <option value="Leads">Leads</option>
                <option value="Retention">Retention</option>
                <option value="Custom">Custom</option>
              </select>

              <select v-model="form.status" class="border rounded px-3 py-2">
                <option value="active">Active</option>
                <option value="paused">Paused</option>
                <option value="completed">Completed</option>
              </select>

              <input v-model="form.owner" placeholder="Owner" class="border rounded px-3 py-2" />
              <input v-model.number="form.target" type="number" placeholder="Target amount (IDR)" class="border rounded px-3 py-2" />
            </div>

            <div class="flex items-center gap-3">
              <input v-model="form.due" type="date" class="border rounded px-3 py-2 w-48" />
              <input v-model.number="form.progress" type="number" placeholder="Current progress (IDR)" class="border rounded px-3 py-2 w-48" />
              <input v-model="form.flag" placeholder="Flag (VIP/Hot)" class="border rounded px-3 py-2 w-48" />
            </div>

            <textarea v-model="form.note" placeholder="Note / description" class="w-full border rounded px-3 py-2"></textarea>
          </div>

          <div class="p-4 border-t flex justify-end gap-2">
            <button type="button" class="px-4 py-2 rounded border" @click="closeModal">Cancel</button>
            <button type="button" class="px-4 py-2 rounded bg-emerald-500 text-white" @click="saveGoal">{{ editMode ? 'Save' : 'Create' }}</button>
          </div>
        </form>
      </dialog>

      <!-- DETAIL SLIDE-OVER -->
      <div v-if="showDetail" class="fixed inset-0 z-40">
        <div class="absolute inset-0 bg-black/30" @click="closeDetail"></div>
        <aside class="absolute right-6 top-20 w-[380px] bg-white rounded-xl shadow-2xl border p-5 overflow-auto max-h-[80vh]">
          <div class="flex items-start justify-between gap-3">
            <div>
              <div class="text-lg font-semibold">{{ selectedGoal.title }}</div>
              <div class="text-xs text-slate-500">{{ selectedGoal.type }} • {{ selectedGoal.owner }}</div>
            </div>
            <button class="text-slate-500" @click="closeDetail">✕</button>
          </div>

          <div class="mt-4 text-sm text-slate-700">
            <div class="mb-3">
              <div class="font-semibold">Progress</div>
              <div class="mt-2">
                <div class="w-full bg-slate-100 rounded-full h-3 overflow-hidden">
                  <div class="h-3 rounded-full" :style="{ width: goalProgressPercent(selectedGoal) + '%', background: progressColor(selectedGoal) }"></div>
                </div>
                <div class="mt-2 text-xs text-slate-500">{{ formatCurrency(selectedGoal.progress) }} / {{ formatCurrency(selectedGoal.target) }} • {{ goalProgressPercent(selectedGoal) }}%</div>
              </div>
            </div>

            <div class="mb-3">
              <div class="font-semibold">Timeline</div>
              <div class="mt-2 text-xs text-slate-500">Due: {{ selectedGoal.due || '—' }}</div>
            </div>

            <div class="mb-3">
              <div class="font-semibold">Notes</div>
              <div class="mt-2 text-sm text-slate-600" v-html="selectedGoal.note || '<em>No notes</em>'"></div>
            </div>

            <div class="mt-4 flex gap-3">
              <button class="px-3 py-2 rounded bg-sky-600 text-white" @click="openEdit(selectedGoal)">Edit</button>
              <button class="px-3 py-2 rounded border text-rose-600" @click="deleteGoal(selectedGoal.id)">Delete</button>
            </div>
          </div>
        </aside>
      </div>
    </div>
  </AdminLayout>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import AdminLayout from '@/components/layout/AdminLayout.vue'
import SearchIcon from '@/icons/SearchIcon.vue'

/* demo goals */
const initialGoals = [
  { id: 1, title: 'Q4 Revenue Target', type: 'Revenue', owner: 'Anna', target: 500000000, progress: 231600000, due: '2025-12-31', status: 'active', flag: 'Top', note: 'Focus on enterprise sales and partnerships.' },
  { id: 2, title: 'New Leads 2025', type: 'Leads', owner: 'Budi', target: 2000, progress: 1150, due: '2025-11-30', status: 'active', flag: '', note: '' },
  { id: 3, title: 'Retention Rate 90%', type: 'Retention', owner: 'Citra', target: 90, progress: 72, due: '2025-12-01', status: 'paused', flag: 'Watch', note: 'Investigate churn reasons.' },
  { id: 4, title: 'Upsell Program', type: 'Revenue', owner: 'Dedi', target: 120000000, progress: 94800000, due: '', status: 'active', flag: '', note: 'Pilot with top 50 customers.' },
  { id: 5, title: 'Improve NPS', type: 'Custom', owner: 'Eka', target: 70, progress: 58, due: '2025-10-30', status: 'completed', flag: '', note: 'NPS improved after onboarding changes.' }
]

const goals = ref(JSON.parse(JSON.stringify(initialGoals)))

/* UI state */
const q = ref('')
const statusFilter = ref('')
const page = ref(1)
const perPage = ref(6)
const selected = ref(new Set())

/* modal form */
const modal = ref(null)
const editMode = ref(false)
const form = reactive({
  id: null, title: '', type: 'Revenue', owner: '', target: 0, progress: 0, due: '', status: 'active', flag: '', note: ''
})

/* detail */
const showDetail = ref(false)
const selectedGoal = reactive({})

/* computed lists */
const filtered = computed(() => {
  const s = q.value.trim().toLowerCase()
  return goals.value
    .filter(g => {
      if (statusFilter.value && g.status !== statusFilter.value) return false
      if (!s) return true
      return (g.title && g.title.toLowerCase().includes(s)) || (g.owner && g.owner.toLowerCase().includes(s)) || (g.type && g.type.toLowerCase().includes(s))
    })
    .sort((a,b) => {
      // prefer flagged, then active, then by progress descending
      if ((a.flag ? 1 : 0) !== (b.flag ? 1 : 0)) return (b.flag ? 1 : 0) - (a.flag ? 1 : 0)
      if (a.status !== b.status) {
        const order = { active: 0, paused: 1, completed: 2 }
        return (order[a.status]||9) - (order[b.status]||9)
      }
      return goalProgressPercent(b) - goalProgressPercent(a)
    })
})

const totalPages = computed(() => Math.max(1, Math.ceil(filtered.value.length / perPage.value)))
const pages = computed(() => {
  const arr = []
  for (let i = 1; i <= totalPages.value; i++) arr.push(i)
  return arr
})
const paged = computed(() => {
  const start = (page.value - 1) * perPage.value
  return filtered.value.slice(start, start + perPage.value)
})

/* selection helpers */
const allSelected = computed(() => selected.value.size > 0 && selected.value.size === filtered.value.length)
function toggleSelectAll(e) {
  if (e.target.checked) selected.value = new Set(filtered.value.map(g => g.id))
  else selected.value = new Set()
}
function toggleSelect(id) {
  if (selected.value.has(id)) selected.value.delete(id)
  else selected.value.add(id)
  selected.value = new Set(selected.value)
}
function isSelected(id) { return selected.value.has(id) }

/* modal create/edit */
function openAdd() {
  editMode.value = false
  form.id = null
  form.title = ''
  form.type = 'Revenue'
  form.owner = ''
  form.target = 0
  form.progress = 0
  form.due = ''
  form.status = 'active'
  form.flag = ''
  form.note = ''
  modal.value?.showModal?.()
}
function openEdit(goal) {
  editMode.value = true
  form.id = goal.id
  form.title = goal.title
  form.type = goal.type
  form.owner = goal.owner
  form.target = goal.target
  form.progress = goal.progress
  form.due = goal.due
  form.status = goal.status
  form.flag = goal.flag
  form.note = goal.note
  modal.value?.showModal?.()
}
function closeModal() { modal.value?.close?.() }

function saveGoal() {
  if (!form.title || !form.owner) {
    alert('Title & Owner required')
    return
  }
  if (editMode.value && form.id) {
    const idx = goals.value.findIndex(g => g.id === form.id)
    if (idx !== -1) {
      goals.value[idx] = { id: form.id, title: form.title, type: form.type, owner: form.owner, target: Number(form.target)||0, progress: Number(form.progress)||0, due: form.due, status: form.status, flag: form.flag, note: form.note }
    }
  } else {
    const id = Math.max(0, ...goals.value.map(g => g.id)) + 1
    goals.value.unshift({ id, title: form.title, type: form.type, owner: form.owner, target: Number(form.target)||0, progress: Number(form.progress)||0, due: form.due, status: form.status, flag: form.flag, note: form.note })
  }
  rebuildPagination()
  closeModal()
}

/* detail panel */
function openDetail(goal) {
  Object.assign(selectedGoal, goal)
  showDetail.value = true
}
function closeDetail() {
  showDetail.value = false
  Object.keys(selectedGoal).forEach(k => delete selectedGoal[k])
}

/* delete */
function deleteGoal(id) {
  if (!confirm('Delete this goal?')) return
  goals.value = goals.value.filter(g => g.id !== id)
  selected.value.delete(id)
  rebuildPagination()
}

/* bulk */
function bulkMarkComplete() {
  if (selected.value.size === 0) return
  const ids = new Set(Array.from(selected.value))
  goals.value.forEach(g => { if (ids.has(g.id)) g.status = 'completed' })
  selected.value = new Set()
}
function bulkDelete() {
  if (selected.value.size === 0) return
  if (!confirm('Delete selected goals?')) return
  const ids = new Set(Array.from(selected.value))
  goals.value = goals.value.filter(g => !ids.has(g.id))
  selected.value = new Set()
  rebuildPagination()
}

/* helpers */
function goalProgressPercent(g) {
  if (!g) return 0
  const t = Number(g.target) || 0
  const p = Number(g.progress) || 0
  if (t === 0) return 0
  return Math.min(100, Math.round((p / t) * 100))
}
function progressColor(g) {
  const pct = goalProgressPercent(g)
  if (pct >= 80) return '#10b981' // green
  if (pct >= 50) return '#f59e0b' // amber
  return '#3b82f6' // blue
}
function formatCurrency(val = 0) {
  if (!val) return 'Rp 0'
  if (val >= 1000) {
    const v = Math.round(val / 100) / 10
    return `Rp ${v}k`
  }
  return `Rp ${val}`
}

/* pagination */
function prevPage() { if (page.value > 1) page.value-- }
function nextPage() { if (page.value < totalPages.value) page.value++ }
function rebuildPagination() { if (page.value > totalPages.value) page.value = totalPages.value }

/* lifecycle */
onMounted(() => {
  rebuildPagination()
})
</script>

<style scoped>
/* small styles */
.goal-card { transition: transform .12s ease; }
.goal-card:hover { transform: translateY(-4px); }

/* modal styles fallback */
dialog::backdrop { background: rgba(0,0,0,0.3); }

/* responsive */
@media (max-width: 900px) {
  .grid-cols-3 { grid-template-columns: repeat(1, minmax(0,1fr)); }
}
</style>
