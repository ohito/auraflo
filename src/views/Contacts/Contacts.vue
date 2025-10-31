<template>
<AdminLayout>
  <div class="p-6">
    <!-- page header -->
    <!-- START: top controls (replace existing top control block) -->
<div class="mb-6">
  <div class="bg-white rounded-2xl border border-gray-100 p-4 shadow-sm flex items-center justify-between">
    <!-- left: Add button -->
    <div>
      <button
        @click="openAdd"
        class="inline-flex items-center gap-2 rounded-lg bg-sky-500 px-4 py-2 text-white text-sm font-medium hover:bg-sky-600 shadow"
      >
        <svg class="w-4 h-4" viewBox="0 0 24 24" fill="none"><path d="M12 4v16M4 12h16" stroke="currentColor" stroke-width="2" stroke-linecap="round"/></svg>
        + Add Contacts
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
<!-- END -->


    <div class="grid grid-cols-12 gap-6">
      <!-- main table -->
      <div class="col-span-12 lg:col-span-9">
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
                    placeholder="Search for contact..."
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
                    <option value="company">Company</option>
                    <option value="score">Lead Score</option>
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
                  <th class="px-3 py-2">
                    <input type="checkbox" :checked="allSelected" @change="toggleSelectAll($event)" />
                  </th>
                  <th class="px-3 py-2">Name</th>
                  <th class="px-3 py-2">Company</th>
                  <th class="px-3 py-2">Email ID</th>
                  <th class="px-3 py-2">Phone No</th>
                  <th class="px-3 py-2">Lead Score</th>
                  <th class="px-3 py-2">Tags</th>
                </tr>
              </thead>
              <tbody>
                <tr
                  v-for="contact in paged"
                  :key="contact.id"
                  class="border-t border-gray-100 hover:bg-gray-50 cursor-pointer"
                  @click="selectRow(contact)"
                >
                  <td class="px-3 py-3 align-top">
                    <input type="checkbox" :checked="isSelected(contact.id)" @click.stop="toggleSelect(contact.id)" />
                  </td>

                  <td class="px-3 py-3 align-top">
                    <div class="flex items-center gap-3">
                      <img :src="contact.avatar" alt="" class="w-9 h-9 rounded-full object-cover border" />
                      <div>
                        <div class="font-medium text-slate-800">{{ contact.name }}</div>
                        <div class="text-xs text-slate-400">{{ contact.role }}</div>
                      </div>
                    </div>
                  </td>

                  <td class="px-3 py-3 align-top text-slate-700">{{ contact.company }}</td>
                  <td class="px-3 py-3 align-top text-slate-600">{{ contact.email }}</td>
                  <td class="px-3 py-3 align-top text-slate-600">{{ contact.phone }}</td>
                  <td class="px-3 py-3 align-top text-slate-700">{{ contact.score }}</td>
                  <td class="px-3 py-3 align-top">
                    <div class="flex gap-2">
                      <span v-for="t in contact.tags" :key="t" class="text-xs px-2 py-1 rounded bg-slate-100 text-slate-600">{{ t }}</span>
                    </div>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>

          <!-- footer: showing + pagination -->
          <div class="mt-4 flex items-center justify-between">
            <div class="text-sm text-slate-500">Showing {{ filtered.length }} results</div>
            <div class="flex items-center gap-2">
              <button @click="prevPage" :disabled="page === 1" class="px-3 py-1 rounded border text-sm disabled:opacity-50">Prev</button>
              <button v-for="p in pages" :key="p" @click="page = p" :class="['px-3 py-1 rounded text-sm', page === p ? 'bg-slate-800 text-white' : 'border']">{{ p }}</button>
              <button @click="nextPage" :disabled="page === totalPages" class="px-3 py-1 rounded border text-sm disabled:opacity-50">Next</button>
            </div>
          </div>
        </div>
      </div>

      <!-- detail panel -->
      <div class="col-span-12 lg:col-span-3">
        <div class="bg-white rounded-2xl border border-gray-200 p-5">
          <div v-if="active" class="text-center">
            <img :src="active.avatar" alt="avatar" class="w-24 h-24 rounded-full mx-auto mb-3 object-cover border" />
            <h3 class="text-lg font-semibold text-slate-800">{{ active.name }}</h3>
            <div class="text-sm text-slate-400">{{ active.company }}</div>

            <div class="mt-4 flex items-center justify-center gap-3">
              <a :href="`tel:${active.phone}`" class="p-2 rounded bg-emerald-50 text-emerald-600 text-sm">📞</a>
              <a :href="`mailto:${active.email}`" class="p-2 rounded bg-emerald-50 text-emerald-600 text-sm">✉️</a>
              <button class="p-2 rounded bg-slate-100 text-slate-600 text-sm" @click="openEdit(active)">✏️</button>
            </div>

            <div class="mt-6 text-left text-sm text-slate-700">
              <div class="mt-3 text-xs text-slate-500">{{ active.note }}</div>
              <p class="mb-2"><span class="text-slate-400">Phone:</span> {{ active.phone }}</p>
              <p class="mb-2"><span class="text-slate-400">Email:</span> {{ active.email }}</p>
              <p class="mb-2"><span class="text-slate-400">Lead Score:</span> {{ active.score }}</p>
              <p class="mb-2"><span class="text-slate-400">Tags:</span>
                <span v-for="t in active.tags" :key="t" class="ml-2 text-xs px-2 py-1 rounded bg-slate-100 text-slate-600">{{ t }}</span>
              </p>
              <p class="mb-2"><span class="text-slate-400">Last Contacted:</span> {{ active.last_updated }}</p>
            </div>
          </div>

          <div v-else class="text-center text-slate-400">
            <div class="py-10">Select a contact to view details</div>
          </div>
        </div>
      </div>
    </div>

    <!-- modal: add/edit (simple) -->
    <dialog ref="modal" class="rounded-lg p-0" @close="closeModal">
      <form method="dialog" class="w-[700px] max-w-full bg-white rounded-lg overflow-hidden shadow-lg">
        <div class="p-5 border-b">
          <h3 class="text-lg font-semibold">Add Contact</h3>
        </div>
        <div class="p-5 space-y-3">
          <div class="grid grid-cols-2 gap-3">
            <input v-model="form.name" placeholder="Full name" class="border rounded px-3 py-2" />
            <input v-model="form.company" placeholder="Company" class="border rounded px-3 py-2" />
            <input v-model="form.email" placeholder="Email" class="border rounded px-3 py-2" />
            <input v-model="form.phone" placeholder="Phone" class="border rounded px-3 py-2" />
          </div>
          <textarea v-model="form.note" placeholder="Note" class="w-full border rounded px-3 py-2"></textarea>
        </div>
        <div class="p-4 border-t flex justify-end gap-2">
          <button type="button" class="px-4 py-2 rounded border" @click="closeModal">Cancel</button>
          <button type="button" class="px-4 py-2 rounded bg-emerald-500 text-white" @click="saveContact">Save</button>
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

const q = ref('')
const sortBy = ref('name')
const page = ref(1)
const perPage = ref(8)
const selected = ref(new Set())
const active = ref(null)
const modal = ref(null)

const raw = ref([
  { id: 1, name: 'Tonya Noble', role: 'Designer', company: 'Nesta Technologies', email: 'tonyanoble@velzon.com', phone: '414-453-5725', score: 225, tags: ['Lead','Partner'], avatar: 'https://i.pravatar.cc/80?img=10', note: 'Personal note about Tonya. Ryan Joseph Giggs (lahir 29 November 1973) adalah pelatih sepak bola Wales, mantan pemain profesional dan salah satu pemilik Salford City. Dianggap sebagai salah satu pemain terhebat di generasinya, Giggs menghabiskan seluruh karier profesionalnya di Manchester United, di mana ia juga menjabat sebagai pemain-manajer dan asisten manajer sementara klub. Dia adalah salah satu pesepakbola paling berprestasi sepanjang masa, dan merupakan salah satu dari hanya 44 pemain yang telah membuat lebih dari 1.000 penampilan dalam kariernya.', last_updated:'15 Dec,2021 08:58AM' },
  { id: 2, name: 'Kevin Dawson', role: 'Developer', company: 'Meta4Systems', email: 'kevindawson@velzon.com', phone: '745-321-9874', score: 500, tags: ['Lead'], avatar: 'https://i.pravatar.cc/80?img=5', note: 'Personal note about Kevin' },
  { id: 3, name: 'Timothy Smith', role: 'Manager', company: 'Digitech Galaxy', email: 'timothysmith@velzon.com', phone: '231-480-8536', score: 154, tags: ['Partner'], avatar: 'https://i.pravatar.cc/80?img=7', note: 'Personal note about Timothy' },
  { id: 4, name: 'Michael Morris', role: 'Designer', company: 'Syntyce Solutions', email: 'michaelmorris@velzon.com', phone: '231-480-8536', score: 732, tags: ['Lead','Partner'], avatar: 'https://i.pravatar.cc/80?img=6', note: 'Personal note about Michael' },
  { id: 5, name: 'Herbert Stokes', role: 'Sales', company: 'Themesbrand', email: 'herbertstokes@velzon.com', phone: '949-791-0614', score: 784, tags: ['Exiting','Lead'], avatar: 'https://i.pravatar.cc/80?img=11', note: 'Personal note about Herbert' },
  { id: 6, name: 'Mary Cousar', role: 'CMO', company: 'Micro Design', email: 'marycousar@velzon.com', phone: '540-575-0991', score: 643, tags: ['Exiting'], avatar: 'https://i.pravatar.cc/80?img=12', note: 'Personal note about Mary' },
  { id: 7, name: 'James Price', role: 'Designer', company: 'Themesbrand', email: 'jamesprice@velzon.com', phone: '646-276-2274', score: 81, tags: ['Lead'], avatar: 'https://i.pravatar.cc/80?img=8', note: 'Personal note about James' },
  { id: 8, name: 'Alexis Clarke', role: 'Product', company: 'Android Galaxy', email: 'alexisclarke@velzon.com', phone: '515-395-1069', score: 369, tags: ['Lead','Partner'], avatar: 'https://i.pravatar.cc/80?img=9', note: 'Personal note about Alexis. Ryan Joseph Giggs (lahir 29 November 1973) adalah pelatih sepak bola Wales, mantan pemain profesional dan salah satu pemilik Salford City. Dianggap sebagai salah satu pemain terhebat di generasinya, Giggs menghabiskan seluruh karier profesionalnya di Manchester United, di mana ia juga menjabat sebagai pemain-manajer dan asisten manajer sementara klub. Dia adalah salah satu pesepakbola paling berprestasi sepanjang masa, dan merupakan salah satu dari hanya 44 pemain yang telah membuat lebih dari 1.000 penampilan dalam kariernya.',last_updated:'15 Dec,2021 08:58AM' },
  // additional sample contacts for pagination demo
  { id: 9, name: 'Lina Hart', role: 'HR', company: 'Acme', email: 'lina@acme.com', phone: '111-222-3333', score: 150, tags: ['Partner'], avatar: 'https://i.pravatar.cc/80?img=13', note: '' },
  { id: 10, name: 'Budi Santoso', role: 'Sales', company: 'LocalCo', email: 'budi@local.co', phone: '0812-345-678', score: 210, tags: ['Lead'], avatar: 'https://i.pravatar.cc/80?img=14', note: '' },
  { id: 11, name: 'Siti Aminah', role: 'Support', company: 'Biz', email: 'siti@biz.com', phone: '0813-222-333', score: 80, tags: [], avatar: 'https://i.pravatar.cc/80?img=15', note: '' },
  { id: 12, name: 'Rian Wirawan', role: 'DevOps', company: 'Infra', email: 'rian@infra.com', phone: '0812-999-888', score: 420, tags: ['Lead'], avatar: 'https://i.pravatar.cc/80?img=16', note: '' },
])

const filtered = computed(() => {
  const s = q.value.trim().toLowerCase()
  return raw.value
    .filter(c => {
      if (!s) return true
      return c.name.toLowerCase().includes(s) || c.email.toLowerCase().includes(s) || c.company.toLowerCase().includes(s)
    })
    .sort((a,b) => {
      if (sortBy.value === 'name') return a.name.localeCompare(b.name)
      if (sortBy.value === 'company') return a.company.localeCompare(b.company)
      if (sortBy.value === 'score') return b.score - a.score
      return 0
    })
})

const totalPages = computed(() => Math.max(1, Math.ceil(filtered.value.length / perPage.value)))
const totalPagesVal = totalPages
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

const allSelected = computed(() => selected.value.size > 0 && selected.value.size === filtered.value.length)

function toggleSelectAll(e) {
  if (e.target.checked) {
    selected.value = new Set(filtered.value.map(c => c.id))
  } else {
    selected.value = new Set()
  }
}
function toggleSelect(id) {
  if (selected.value.has(id)) selected.value.delete(id)
  else selected.value.add(id)
  // make reactive
  selected.value = new Set(selected.value)
}

function isSelected(id) {
  return selected.value.has(id)
}

function selectRow(contact) {
  active.value = contact
}

function prevPage() {
  if (page.value > 1) page.value--
}
function nextPage() {
  if (page.value < totalPages.value) page.value++
}

function openAdd() {
  form.name = ''
  form.company = ''
  form.email = ''
  form.phone = ''
  form.note = ''
  modal.value.showModal()
}

function closeModal() {
  modal.value?.close()
}

const form = {
  name: '',
  company: '',
  email: '',
  phone: '',
  note: ''
}

function saveContact() {
  // demo: push to raw
  const id = Math.max(...raw.value.map(r => r.id)) + 1
  raw.value.unshift({
    id,
    name: form.name || 'No name',
    role: '',
    company: form.company || '',
    email: form.email || '',
    phone: form.phone || '',
    score: 0,
    tags: [],
    avatar: `https://i.pravatar.cc/80?img=${Math.floor(Math.random()*60)+1}`,
    note: form.note || ''
  })
  closeModal()
}

onMounted(() => {
  // set initial active
  active.value = raw.value[0]
})
</script>

<style scoped>
dialog::backdrop {
  background: rgba(0,0,0,0.3);
}
</style>
