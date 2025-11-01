<template>
  <AdminLayout>
    <div class="p-6">
      <!-- header -->
      <div class="mb-6">
        <div class="bg-white rounded-2xl border border-gray-200 p-4 shadow-sm flex items-center justify-between gap-4">
          <div class="flex items-center gap-4 flex-1 min-w-0">
            <h2 class="text-2xl font-semibold whitespace-nowrap">Leads</h2>

            <div class="relative flex-1 min-w-0">
              <span class="absolute left-3 top-1/2 -translate-y-1/2 text-slate-400">
                <SearchIcon class="w-4 h-4" />
              </span>
              <input
                v-model="q"
                type="search"
                placeholder="Search leads..."
                class="w-full h-10 rounded-xl border border-gray-200 px-4 pl-10 text-sm placeholder:text-slate-400 focus:outline-none"
              />
            </div>

            <div class="hidden sm:flex items-center gap-2">
              <div class="text-sm text-slate-500">Status:</div>
              <select v-model="statusFilter" class="h-9 rounded-xl border border-gray-200 bg-white px-3 text-sm">
                <option value="">All</option>
                <option value="new">New</option>
                <option value="contacted">Contacted</option>
                <option value="qualified">Qualified</option>
                <option value="lost">Lost</option>
              </select>
            </div>
          </div>

          <div class="flex items-center gap-3">
            <button @click="openImport" class="inline-flex items-center gap-2 rounded-lg bg-emerald-50 text-emerald-700 px-4 py-2 text-sm border border-emerald-100 hover:bg-emerald-100">
              Import
            </button>

            <button @click="openAdd" class="inline-flex items-center gap-2 rounded-lg bg-sky-500 px-4 py-2 text-white text-sm font-medium hover:bg-sky-600 shadow">
              + Add Lead
            </button>
          </div>
        </div>
      </div>

      <!-- main content -->
      <div class="bg-white rounded-2xl border border-gray-200 p-4 shadow-sm">
        <div class="overflow-x-auto">
          <table class="min-w-full text-sm">
            <thead>
              <tr class="text-left text-xs text-slate-500">
                <th class="px-3 py-2">
                  <input type="checkbox" :checked="allSelected" @change="toggleSelectAll($event)" />
                </th>
                <th class="px-3 py-2">Name</th>
                <th class="px-3 py-2">Company</th>
                <th class="px-3 py-2">Email</th>
                <th class="px-3 py-2">Phone</th>
                <th class="px-3 py-2">Status</th>
                <th class="px-3 py-2">Actions</th>
              </tr>
            </thead>

            <tbody>
              <tr
                v-for="lead in paged"
                :key="lead.id"
                class="border-t border-gray-100 hover:bg-gray-50 cursor-pointer"
              >
                <td class="px-3 py-3 align-top">
                  <input
                    type="checkbox"
                    :checked="isSelected(lead.id)"
                    @click.stop="toggleSelect(lead.id)"
                  />
                </td>

                <td class="px-3 py-3 align-top" @click="openDetail(lead)">
                  <div class="flex items-center gap-3">
                    <img :src="lead.avatar" alt="" class="w-9 h-9 rounded-full object-cover border" />
                    <div>
                      <div class="font-medium text-slate-800">{{ lead.name }}</div>
                      <div class="text-xs text-slate-400">{{ lead.title || '—' }}</div>
                    </div>
                  </div>
                </td>

                <td class="px-3 py-3 align-top text-slate-700">{{ lead.company }}</td>
                <td class="px-3 py-3 align-top text-slate-600">{{ lead.email }}</td>
                <td class="px-3 py-3 align-top text-slate-600">{{ lead.phone }}</td>
                <td class="px-3 py-3 align-top">
                  <span
                    :class="['px-2 py-1 rounded text-xs font-semibold', statusClass(lead.status)]"
                  >
                    {{ statusLabel(lead.status) }}
                  </span>
                </td>
                <td class="px-3 py-3 align-top">
                  <div class="flex items-center gap-2">
                    <button class="px-3 py-1 rounded border text-sm" @click.stop="openEdit(lead)">Edit</button>
                    <button class="px-3 py-1 rounded border text-sm" @click.stop="deleteLead(lead.id)">Delete</button>
                  </div>
                </td>
              </tr>

              <tr v-if="filtered.length === 0">
                <td colspan="7" class="p-6 text-center text-slate-400">No leads found</td>
              </tr>
            </tbody>
          </table>
        </div>

        <!-- footer: showing + pagination -->
        <div class="mt-4 flex items-center justify-between">
          <div class="text-sm text-slate-500">Showing {{ filtered.length }} results</div>
          <div class="flex items-center gap-2">
            <button @click="prevPage" :disabled="page === 1" class="px-3 py-1 rounded border text-sm disabled:opacity-50">Prev</button>
            <button
              v-for="p in pages"
              :key="p"
              @click="page = p"
              :class="['px-3 py-1 rounded text-sm', page === p ? 'bg-slate-800 text-white' : 'border']"
            >
              {{ p }}
            </button>
            <button @click="nextPage" :disabled="page === totalPages" class="px-3 py-1 rounded border text-sm disabled:opacity-50">Next</button>
          </div>
        </div>
      </div>

      <!-- modal create/edit -->
      <dialog ref="modal" class="rounded-lg p-0" @close="closeModal">
        <form method="dialog" class="w-[700px] max-w-full bg-white rounded-lg overflow-hidden shadow-lg">
          <div class="p-5 border-b flex items-center justify-between">
            <h3 class="text-lg font-semibold">{{ editMode ? 'Edit Lead' : 'Add Lead' }}</h3>
            <div class="text-sm text-slate-400">{{ form.id ? ('ID: ' + form.id) : '' }}</div>
          </div>

          <div class="p-5 space-y-3">
            <div class="grid grid-cols-2 gap-3">
              <input v-model="form.name" placeholder="Full name" class="border rounded px-3 py-2" />
              <input v-model="form.company" placeholder="Company" class="border rounded px-3 py-2" />
              <input v-model="form.email" placeholder="Email" class="border rounded px-3 py-2" />
              <input v-model="form.phone" placeholder="Phone" class="border rounded px-3 py-2" />
              <input v-model="form.title" placeholder="Title" class="border rounded px-3 py-2" />
              <select v-model="form.status" class="border rounded px-3 py-2">
                <option value="new">New</option>
                <option value="contacted">Contacted</option>
                <option value="qualified">Qualified</option>
                <option value="lost">Lost</option>
              </select>
            </div>

            <textarea v-model="form.note" placeholder="Note" class="w-full border rounded px-3 py-2"></textarea>
          </div>

          <div class="p-4 border-t flex justify-end gap-2">
            <button type="button" class="px-4 py-2 rounded border" @click="closeModal">Cancel</button>
            <button type="button" class="px-4 py-2 rounded bg-emerald-500 text-white" @click="saveLead">{{ editMode ? 'Save' : 'Create' }}</button>
          </div>
        </form>
      </dialog>

      <!-- detail slide panel -->
      <div v-if="showDetail" class="fixed inset-0 z-40">
        <div class="absolute inset-0 bg-black/30" @click="closeDetail"></div>
        <aside class="absolute right-6 top-20 w-[360px] bg-white rounded-xl shadow-2xl border p-5 overflow-auto max-h-[70vh]">
          <div class="flex items-start justify-between gap-3">
            <div class="flex items-center gap-3">
              <img :src="activeLead.avatar" class="w-12 h-12 rounded-full border object-cover" />
              <div>
                <div class="font-semibold text-lg">{{ activeLead.name }}</div>
                <div class="text-xs text-slate-500">{{ activeLead.company }}</div>
              </div>
            </div>
            <button class="text-slate-500" @click="closeDetail">✕</button>
          </div>

          <div class="mt-4 text-sm text-slate-700">
            <div class="mb-2 font-semibold">{{ activeLead.title || '—' }}</div>
            <div class="mb-3 text-xs text-slate-500">{{ activeLead.email }} • {{ activeLead.phone }}</div>
            <div class="mb-4" v-html="activeLead.note || '<em>No notes</em>'"></div>

            <div class="mt-2 flex gap-2">
              <a :href="`tel:${activeLead.phone || ''}`" class="px-3 py-2 rounded bg-amber-400 text-white font-semibold">📞 Call</a>
              <button class="px-3 py-2 rounded bg-sky-500 text-white" @click="openEdit(activeLead)">✉️ Message / Edit</button>
            </div>
          </div>
        </aside>
      </div>
    </div>
  </AdminLayout>
</template>

<script setup>
import { ref, computed, reactive, onMounted } from 'vue'
import AdminLayout from '@/components/layout/AdminLayout.vue'
import SearchIcon from '@/icons/SearchIcon.vue'

/* sample leads */
const raw = ref([
  { id: 1, name: 'Ayu Rahma', title: 'Head Marketing', company: 'PT. Samudra', email: 'ayu@samudra.co', phone: '0812-1111-2222', status: 'new', avatar: 'https://i.pravatar.cc/80?img=10', note: 'Interested in product demo.' },
  { id: 2, name: 'Budi Santoso', title: 'CTO', company: 'InfraTech', email: 'budi@infra.id', phone: '0812-2222-3333', status: 'contacted', avatar: 'https://i.pravatar.cc/80?img=14', note: '' },
  { id: 3, name: 'Citra Dewi', title: 'Founder', company: 'Kopi Kita', email: 'citra@kopikita.com', phone: '0813-5555-6666', status: 'qualified', avatar: 'https://i.pravatar.cc/80?img=15', note: 'Budget approved.' },
  { id: 4, name: 'Dedi Hartono', title: 'Procurement', company: 'LogiCorp', email: 'dedi@logicorp.com', phone: '0812-9999-0000', status: 'lost', avatar: 'https://i.pravatar.cc/80?img=12', note: 'Not a fit right now.' },
  // add more for pagination demo
  { id: 5, name: 'Eka Putri', title: 'Marketing', company: 'MakanYuk', email: 'eka@makan.com', phone: '0812-4444-5555', status: 'new', avatar: 'https://i.pravatar.cc/80?img=8', note: '' },
  { id: 6, name: 'Fajar', title: 'Sales', company: 'TokoPlus', email: 'fajar@toko.com', phone: '0812-7777-8888', status: 'contacted', avatar: 'https://i.pravatar.cc/80?img=20', note: '' },
  { id: 7, name: 'Gita', title: 'Owner', company: 'Gita Studio', email: 'gita@studio.com', phone: '0813-1010-2020', status: 'qualified', avatar: 'https://i.pravatar.cc/80?img=30', note: '' },
  { id: 8, name: 'Hendra', title: 'COO', company: 'Hendra Corp', email: 'hendra@corp.com', phone: '0812-1313-1414', status: 'new', avatar: 'https://i.pravatar.cc/80?img=25', note: '' },
])

/* selection */
const selected = ref(new Set())

/* filters / search / pagination */
const q = ref('')
const statusFilter = ref('')
const page = ref(1)
const perPage = ref(6)

/* detail panel */
const showDetail = ref(false)
const activeLead = reactive({})

/* modal & form */
const modal = ref(null)
const form = reactive({
  id: null,
  name: '',
  title: '',
  company: '',
  email: '',
  phone: '',
  status: 'new',
  avatar: '',
  note: ''
})
const editMode = ref(false)

/* computed lists */
const filtered = computed(() => {
  const s = q.value.trim().toLowerCase()
  return raw.value
    .filter(l => {
      if (statusFilter.value && l.status !== statusFilter.value) return false
      if (!s) return true
      return l.name.toLowerCase().includes(s) || l.email.toLowerCase().includes(s) || l.company.toLowerCase().includes(s)
    })
    .sort((a,b) => a.name.localeCompare(b.name))
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
  if (e.target.checked) {
    selected.value = new Set(filtered.value.map(l => l.id))
  } else {
    selected.value = new Set()
  }
}
function toggleSelect(id) {
  if (selected.value.has(id)) selected.value.delete(id)
  else selected.value.add(id)
  // reassign to trigger reactivity
  selected.value = new Set(selected.value)
}
function isSelected(id) { return selected.value.has(id) }

/* pagination helpers */
function prevPage() { if (page.value > 1) page.value-- }
function nextPage() { if (page.value < totalPages.value) page.value++ }

/* detail panel */
function openDetail(lead) {
  Object.assign(activeLead, lead)
  showDetail.value = true
}
function closeDetail() {
  showDetail.value = false
  // clear activeLead
  Object.keys(activeLead).forEach(k => delete activeLead[k])
}

/* modal create/edit */
function openAdd() {
  editMode.value = false
  form.id = null
  form.name = ''
  form.title = ''
  form.company = ''
  form.email = ''
  form.phone = ''
  form.status = 'new'
  form.avatar = `https://i.pravatar.cc/80?img=${Math.floor(Math.random()*80)+1}`
  form.note = ''
  modal.value?.showModal?.()
}
function openEdit(lead) {
  editMode.value = true
  form.id = lead.id
  form.name = lead.name
  form.title = lead.title
  form.company = lead.company
  form.email = lead.email
  form.phone = lead.phone
  form.status = lead.status
  form.avatar = lead.avatar || `https://i.pravatar.cc/80?img=${Math.floor(Math.random()*80)+1}`
  form.note = lead.note || ''
  modal.value?.showModal?.()
}
function closeModal() { modal.value?.close?.() }

function saveLead() {
  if (!form.name || !form.company) {
    alert('Name & company are required')
    return
  }
  if (editMode.value && form.id) {
    const idx = raw.value.findIndex(r => r.id === form.id)
    if (idx !== -1) {
      raw.value[idx] = { id: form.id, name: form.name, title: form.title, company: form.company, email: form.email, phone: form.phone, status: form.status, avatar: form.avatar, note: form.note }
    }
  } else {
    const id = Math.max(0, ...raw.value.map(r => r.id)) + 1
    raw.value.unshift({ id, name: form.name, title: form.title, company: form.company, email: form.email, phone: form.phone, status: form.status, avatar: form.avatar, note: form.note })
  }
  rebuildPagination()
  closeModal()
}

/* delete */
function deleteLead(id) {
  if (!confirm('Delete this lead?')) return
  raw.value = raw.value.filter(r => r.id !== id)
  // remove from selection
  selected.value.delete(id)
  rebuildPagination()
}

/* helpers */
function rebuildPagination() {
  // ensure current page is not out of range
  if (page.value > totalPages.value) page.value = totalPages.value
}

/* small UI helpers */
function statusLabel(status) {
  if (!status) return '—'
  const map = { new: 'New', contacted: 'Contacted', qualified: 'Qualified', lost: 'Lost' }
  return map[status] || status
}
function statusClass(status) {
  if (status === 'new') return 'bg-emerald-100 text-emerald-700'
  if (status === 'contacted') return 'bg-amber-100 text-amber-700'
  if (status === 'qualified') return 'bg-sky-100 text-sky-700'
  if (status === 'lost') return 'bg-rose-100 text-rose-700'
  return 'bg-slate-100 text-slate-700'
}

/* import placeholder */
function openImport() {
  alert('Import feature placeholder — implement CSV upload here.')
}

onMounted(() => {
  rebuildPagination()
})
</script>

<style scoped>
/* small style overrides to match app */
.col-title { font-weight: 700; }
.dialog-backdrop {
  background: rgba(0,0,0,0.35);
}

/* ensure slide panel sits above other stuff */
.fixed.inset-0 { z-index: 60; }

/* responsive tweaks */
@media (max-width: 900px) {
  .header-card { flex-direction: column; gap: 8px; }
}
</style>
