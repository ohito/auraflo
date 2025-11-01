<template>
<AdminLayout>
  <div class="p-6">
    <!-- header -->
    <div class="mb-6">
  <div class="bg-white rounded-2xl border border-gray-100 p-4 shadow-sm flex items-center justify-between">
    <!-- left: Add button -->
    <div>
      <button
        @click="openAdd"
        class="inline-flex items-center gap-2 rounded-lg bg-sky-500 px-4 py-2 text-white text-sm font-medium hover:bg-sky-600 shadow"
      >
        <svg class="w-4 h-4" viewBox="0 0 24 24" fill="none"><path d="M12 4v16M4 12h16" stroke="currentColor" stroke-width="2" stroke-linecap="round"/></svg>
        + Add Companies
      </button>
    </div>

    <!-- right: Filters / Import / More (three vertical dots) -->
    <div class="flex items-center gap-3">
      <button class="inline-flex items-center gap-2 rounded-lg bg-rose-500 px-4 py-2 text-white text-sm font-medium hover:bg-rose-600">
        <FilterIcon class="w-4 h-4" />
        Filters
      </button>

      <button class="inline-flex items-center gap-2 rounded-lg bg-emerald-50 text-emerald-700 px-4 py-2 text-sm border border-emerald-100 hover:bg-emerald-100">
        Import
      </button>

      <!-- small square button with vertical dots (more) -->
      <button
        class="inline-flex items-center justify-center rounded-lg bg-white border border-gray-100 px-3 py-2 text-slate-500 hover:bg-gray-50"
        title="More"
      >
        <!-- vertical ellipsis icon -->
        <svg class="w-5 h-5" viewBox="0 0 24 24" fill="none" aria-hidden="true">
          <circle cx="12" cy="6" r="1.5" fill="currentColor"></circle>
          <circle cx="12" cy="12" r="1.5" fill="currentColor"></circle>
          <circle cx="12" cy="18" r="1.5" fill="currentColor"></circle>
        </svg>
      </button>
    </div>
  </div>
</div>

    <!-- content -->
    <div class="grid grid-cols-12 gap-6">
      <!-- table -->
      <div class="col-span-12 lg:col-span-8">
        <div class="bg-white rounded-2xl border border-gray-200 p-4">
             <!-- START: replace search & sort card with this -->
            <div class="bg-white rounded-2xl border border-gray-100 p-4 shadow-sm">
            <div class="flex items-center justify-between gap-4">
                <!-- search (left) -->
                <div class="flex-1">
                <div class="relative">
                    <span class="absolute left-3 top-1/2 -translate-y-1/2 text-slate-400">
                    <SearchIcon class="w-5 h-5" />  
                    </span>
                    <input
                    v-model="q"
                    type="search"
                    placeholder="Search for company..."
                    class="w-full h-12 rounded-xl border border-gray-200 px-4 pl-11 text-sm placeholder:text-slate-400 focus:outline-none focus:ring-1 focus:ring-emerald-300"
                    />
                </div>
                </div>

                <!-- sort (right) -->
                <div class="flex items-center gap-3">
                <div class="text-sm text-slate-500 hidden sm:block">Sort by:</div>
                <div>
                    <select
                    v-model="sortBy"
                    class="h-10 rounded-xl border border-gray-200 bg-white px-4 text-sm focus:outline-none"
                    >
                    <option value="name">Name</option>
                    <option value="industry">Industry</option>
                    <option value="revenue">Revenue</option>
                    </select>
                </div>
                </div>
            </div>
            </div>
            <!-- END -->

          <div class="overflow-x-auto">
            <table class="min-w-full text-sm">
              <thead>
                <tr class="text-left text-xs text-slate-500">
                  <th class="px-3 py-2">#</th>
                  <th class="px-3 py-2">Company Name</th>
                  <th class="px-3 py-2">Industry</th>
                  <th class="px-3 py-2">Location</th>
                  <th class="px-3 py-2">Employees</th>
                  <th class="px-3 py-2">Annual Revenue</th>
                </tr>
              </thead>

              <tbody>
                <tr
                  v-for="(c, idx) in paged"
                  :key="c.id"
                  class="border-t border-gray-100 hover:bg-gray-50 cursor-pointer"
                  @click="selectRow(c)"
                >
                  <td class="px-3 py-3 align-top text-slate-600">{{ (page-1)*perPage + idx + 1 }}</td>

                  <td class="px-3 py-3 align-top">
                    <div class="flex items-center gap-3">
                      <div class="w-10 h-10 rounded-md bg-slate-100 flex items-center justify-center text-slate-700 font-semibold">
                        {{ companyInitials(c.name) }}
                      </div>
                      <div>
                        <div class="font-medium text-slate-800">{{ c.name }}</div>
                        <div class="text-xs text-slate-400">{{ c.contact_person }}</div>
                      </div>
                    </div>
                  </td>

                  <td class="px-3 py-3 align-top text-slate-700">{{ c.industry }}</td>
                  <td class="px-3 py-3 align-top text-slate-600">{{ c.location }}</td>
                  <td class="px-3 py-3 align-top text-slate-700">{{ c.employees }}</td>
                  <td class="px-3 py-3 align-top text-slate-700">{{ formatMoney(c.revenue) }}</td>
                </tr>

                <tr v-if="filtered.length === 0">
                  <td colspan="6" class="px-3 py-6 text-center text-sm text-slate-400">No companies found</td>
                </tr>
              </tbody>
            </table>
          </div>

          <!-- footer -->
          <div class="mt-4 flex items-center justify-between">
            <div class="text-sm text-slate-500">Showing {{ filtered.length }} companies</div>
            <div class="flex items-center gap-2">
              <button @click="prevPage" :disabled="page === 1" class="px-3 py-1 rounded border text-sm disabled:opacity-50">Prev</button>
              <button v-for="p in pages" :key="p" @click="page = p" :class="['px-3 py-1 rounded text-sm', page === p ? 'bg-slate-800 text-white' : 'border']">{{ p }}</button>
              <button @click="nextPage" :disabled="page === totalPages" class="px-3 py-1 rounded border text-sm disabled:opacity-50">Next</button>
            </div>
          </div>
        </div>
      </div>

      <!-- detail panel -->
      <div class="col-span-12 lg:col-span-4">
        <div class="bg-white rounded-2xl border border-gray-200 p-5">
          <div v-if="active" class="text-left">
            <div class="flex items-center gap-4">
              <div class="w-20 h-20 rounded-md bg-slate-100 flex items-center justify-center text-slate-700 font-bold text-xl">
                {{ companyInitials(active.name) }}
              </div>
              <div>
                <h3 class="text-lg font-semibold text-slate-800">{{ active.name }}</h3>
                <div class="text-sm text-slate-400">{{ active.industry }}</div>
              </div>
            </div>

            <div class="mt-4 text-sm text-slate-700">
              <p class="mb-2"><span class="text-slate-400">Contact:</span> {{ active.contact_person }}</p>
              <p class="mb-2"><span class="text-slate-400">Email:</span> {{ active.email }}</p>
              <p class="mb-2"><span class="text-slate-400">Phone:</span> {{ active.phone }}</p>
              <p class="mb-2"><span class="text-slate-400">Employees:</span> {{ active.employees }}</p>
              <p class="mb-2"><span class="text-slate-400">Revenue:</span> {{ formatMoney(active.revenue) }}</p>
              <p class="mt-3 text-xs text-slate-500">{{ active.note }}</p>
            </div>

            <div class="mt-4 flex gap-2">
              <button class="px-3 py-2 rounded border text-sm">Edit</button>
              <button @click="removeCompany(active.id)" class="px-3 py-2 rounded bg-red-50 text-red-600 text-sm border border-red-100">Remove</button>
            </div>
          </div>

          <div v-else class="text-center text-slate-400">
            <div class="py-10">Select a company to view details</div>
          </div>
        </div>
      </div>
    </div>

    <!-- modal add/edit -->
    <dialog ref="modal" class="rounded-lg p-0" @close="closeModal">
      <form method="dialog" class="w-[700px] max-w-full bg-white rounded-lg overflow-hidden shadow-lg">
        <div class="p-5 border-b">
          <h3 class="text-lg font-semibold">{{ editing ? 'Edit Company' : 'Add Company' }}</h3>
        </div>
        <div class="p-5 space-y-3">
          <div class="grid grid-cols-2 gap-3">
            <input v-model="form.name" placeholder="Company name" class="border rounded px-3 py-2" />
            <input v-model="form.industry" placeholder="Industry" class="border rounded px-3 py-2" />
            <input v-model="form.contact_person" placeholder="Contact person" class="border rounded px-3 py-2" />
            <input v-model="form.phone" placeholder="Phone" class="border rounded px-3 py-2" />
            <input v-model="form.email" placeholder="Email" class="border rounded px-3 py-2" />
            <input v-model.number="form.employees" placeholder="Employees" class="border rounded px-3 py-2" />
            <input v-model.number="form.revenue" placeholder="Annual revenue (USD)" class="border rounded px-3 py-2" />
          </div>
          <textarea v-model="form.note" placeholder="Notes" class="w-full border rounded px-3 py-2"></textarea>
        </div>
        <div class="p-4 border-t flex justify-end gap-2">
          <button type="button" class="px-4 py-2 rounded border" @click="closeModal">Cancel</button>
          <button type="button" class="px-4 py-2 rounded bg-emerald-500 text-white" @click="saveCompany">{{ editing ? 'Update' : 'Save' }}</button>
        </div>
      </form>
    </dialog>
  </div>
  </AdminLayout>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import AdminLayout from '@/components/layout/AdminLayout.vue'
import SearchIcon from '@/icons/SearchIcon.vue'
import FilterIcon from '@/icons/FilterIcon.vue'

// state
const q = ref('')
const sortBy = ref('name')
const page = ref(1)
const perPage = ref(8)
const active = ref(null)
const modal = ref(null)
const editing = ref(false)

// sample data
const raw = ref([
  { id: 1, name: 'Velzon Inc', industry: 'Software', location: 'San Francisco, CA', contact_person: 'Tonya Noble', phone: '414-453-5725', email: 'tonya@velzon.com', employees: 320, revenue: 1250000, note: 'Top client.' },
  { id: 2, name: 'MetaSoft', industry: 'Technology', location: 'New York, NY', contact_person: 'Kevin Dawson', phone: '745-321-9874', email: 'kevin@metasoft.com', employees: 1200, revenue: 4200000, note: '' },
  { id: 3, name: 'Digitech Galaxy', industry: 'Services', location: 'Austin, TX', contact_person: 'Timothy Smith', phone: '231-480-8536', email: 'tim@digitech.com', employees: 240, revenue: 780000, note: '' },
  { id: 4, name: 'Syntyce Solutions', industry: 'Consulting', location: 'Chicago, IL', contact_person: 'Michael Morris', phone: '231-480-8536', email: 'michael@syntyce.com', employees: 85, revenue: 290000, note: '' },
  { id: 5, name: 'Themesbrand', industry: 'Design', location: 'Seattle, WA', contact_person: 'Herbert Stokes', phone: '949-791-0614', email: 'herbert@themes.com', employees: 46, revenue: 98000, note: '' },
  { id: 6, name: 'Micro Design', industry: 'Design', location: 'Boston, MA', contact_person: 'Mary Cousar', phone: '540-575-0991', email: 'mary@micro.co', employees: 56, revenue: 140000, note: '' },
  { id: 7, name: 'Themesbrand 2', industry: 'Design', location: 'Remote', contact_person: 'James Price', phone: '646-276-2274', email: 'james@themes.com', employees: 12, revenue: 36000, note: '' },
  { id: 8, name: 'Android Galaxy', industry: 'Hardware', location: 'San Jose, CA', contact_person: 'Alexis Clarke', phone: '515-395-1069', email: 'alexis@android.com', employees: 560, revenue: 2200000, note: '' },
])

// computed / helpers
const filtered = computed(() => {
  const s = q.value.trim().toLowerCase()
  return raw.value
    .filter(c => {
      if (!s) return true
      return c.name.toLowerCase().includes(s) || c.industry.toLowerCase().includes(s) || c.location.toLowerCase().includes(s)
    })
    .sort((a,b) => {
      if (sortBy.value === 'name') return a.name.localeCompare(b.name)
      if (sortBy.value === 'industry') return a.industry.localeCompare(b.industry)
      if (sortBy.value === 'revenue') return b.revenue - a.revenue
      return 0
    })
})

const totalPages = computed(() => Math.max(1, Math.ceil(filtered.value.length / perPage.value)))
const totalPagesArray = computed(() => {
  const arr = []
  for (let i = 1; i <= totalPages.value; i++) arr.push(i)
  return arr
})

const paged = computed(() => {
  const start = (page.value - 1) * perPage.value
  return filtered.value.slice(start, start + perPage.value)
})

const pages = totalPagesArray
const totalPagesVal = totalPages

function prevPage() { if (page.value > 1) page.value-- }
function nextPage() { if (page.value < totalPages.value) page.value++ }

function selectRow(c) {
  active.value = c
}

const form = ref({
  id: null, name: '', industry: '', location: '', contact_person: '', phone: '', email: '', employees: 0, revenue: 0, note: ''
})

function openAdd() {
  editing.value = false
  form.value = { id: null, name: '', industry: '', location: '', contact_person: '', phone: '', email: '', employees: 0, revenue: 0, note: '' }
  modal.value.showModal()
}

function closeModal() {
  modal.value?.close()
}

function saveCompany() {
  if (!form.value.name) return alert('Name required')
  if (editing.value) {
    const i = raw.value.findIndex(r => r.id === form.value.id)
    if (i !== -1) raw.value[i] = { ...form.value }
  } else {
    const id = Math.max(...raw.value.map(r => r.id)) + 1
    raw.value.unshift({ ...form.value, id })
  }
  closeModal()
}

function removeCompany(id) {
  if (!confirm('Remove company?')) return
  raw.value = raw.value.filter(r => r.id !== id)
  if (active.value?.id === id) active.value = null
}

function companyInitials(name = '') {
  return (name.split(' ').map(s => s[0] || '').slice(0,2).join('') || 'C').toUpperCase()
}

function formatMoney(n = 0) {
  if (!n) return '$0'
  if (n >= 1000) {
    const k = Math.round(n / 1000)
    return `$${k.toLocaleString()}k`
  }
  return `$${n.toLocaleString()}`
}

onMounted(() => {
  active.value = raw.value[0] || null
})
</script>

<style scoped>
dialog::backdrop {
  background: rgba(0,0,0,0.3);
}
</style>
