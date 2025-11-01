<template>
  <AdminLayout>
    <div class="p-6">
      <!-- header -->
      <div class="mb-6">
        <div class="bg-white rounded-2xl border border-gray-100 p-4 shadow-sm flex items-center justify-between">
          <div class="flex items-center gap-3">
        
            <button
              @click="openEmailBlast"
              class="inline-flex items-center gap-2 rounded-lg bg-indigo-600 px-4 py-2 text-white text-sm font-medium hover:bg-indigo-700 shadow"
              title="Create Email Blast"
            >
              Email Blast
            </button>
          </div>

          <div class="flex items-center gap-3">
            <button class="inline-flex items-center gap-2 rounded-lg bg-emerald-50 text-emerald-700 px-4 py-2 text-sm border border-emerald-100 hover:bg-emerald-100">
              Import
            </button>

            <button
              class="inline-flex items-center justify-center rounded-lg bg-white border border-gray-100 px-3 py-2 text-slate-500 hover:bg-gray-50"
              title="More"
            >
              <svg class="w-5 h-5" viewBox="0 0 24 24" fill="none" aria-hidden="true">
                <circle cx="12" cy="6" r="1.5" fill="currentColor"></circle>
                <circle cx="12" cy="12" r="1.5" fill="currentColor"></circle>
                <circle cx="12" cy="18" r="1.5" fill="currentColor"></circle>
              </svg>
            </button>
          </div>
        </div>
      </div>

      <!-- Main grid: left = history list, right = contact preview -->
      <div class="mb-6">
        <!-- history list -->
        <div class="col-span-12 lg:col-span-9">
          <div class="bg-white rounded-2xl border border-gray-200 p-4">
            <div class="flex items-center justify-between mb-4">
              <h3 class="text-lg font-semibold">Email Blast History</h3>
              <div class="text-sm text-slate-500">Total: {{ blastsHistory.length }}</div>
            </div>

            <div class="overflow-x-auto">
              <table class="min-w-full text-sm">
                <thead>
                  <tr class="text-left text-xs text-slate-500">
                    <th class="px-3 py-2">Subject</th>
                    <th class="px-3 py-2">From</th>
                    <th class="px-3 py-2">Recipients</th>
                    <th class="px-3 py-2">Status</th>
                    <th class="px-3 py-2">Scheduled At</th>
                    <th class="px-3 py-2">Actions</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="b in blastsHistory" :key="b.id" class="border-t border-gray-100 hover:bg-gray-50">
                    <td class="px-3 py-3 align-top">
                      <div class="font-medium text-slate-800 truncate max-w-[320px]" :title="b.subject">{{ b.subject || '(no subject)' }}</div>
                      <div class="text-xs text-slate-400 mt-1 truncate max-w-[320px]" :title="b.previewText">{{ b.previewText }}</div>
                    </td>
                    <td class="px-3 py-3 align-top text-slate-700">{{ b.from }}</td>
                    <td class="px-3 py-3 align-top text-slate-600">{{ b.recipients.length }}</td>
                    <td class="px-3 py-3 align-top">
                      <span v-if="b.status === 'sent'" class="text-xs px-2 py-1 rounded bg-emerald-100 text-emerald-700">Sent</span>
                      <span v-else-if="b.status === 'scheduled'" class="text-xs px-2 py-1 rounded bg-yellow-100 text-yellow-700">Scheduled</span>
                      <span v-else class="text-xs px-2 py-1 rounded bg-slate-100 text-slate-700">{{ b.status }}</span>
                    </td>
                    <td class="px-3 py-3 align-top text-slate-500">{{ b.scheduledAt || '-' }}</td>
                    <td class="px-3 py-3 align-top">
                      <div class="flex gap-2">
                        <button class="px-2 py-1 rounded border text-xs" @click="viewBlast(b)">View</button>
                        <button class="px-2 py-1 rounded border text-xs" @click="resendBlast(b)">Resend</button>
                        <button class="px-2 py-1 rounded border text-xs text-red-600" @click="deleteBlast(b.id)">Delete</button>
                      </div>
                    </td>
                  </tr>

                  <tr v-if="blastsHistory.length === 0">
                    <td colspan="6" class="px-3 py-6 text-center text-slate-400">No email blasts yet</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>

       
      </div>

      <!-- Email Blast modal -->
      <dialog ref="emailModal" class="rounded-lg p-0" @close="closeEmailModal">
        <form method="dialog" class="w-[900px] max-w-full bg-white rounded-lg overflow-hidden shadow-lg">
          <div class="p-5 border-b flex items-center justify-between">
            <div>
              <h3 class="text-lg font-semibold">Create Email Blast</h3>
              <div class="text-sm text-slate-500">Compose and select recipients</div>
            </div>
            <div class="text-sm text-slate-400">Recipients: <strong>{{ selectedRecipientsCount }}</strong></div>
          </div>

          <div class="p-5 space-y-4">
            <div class="grid grid-cols-2 gap-3">
              <input v-model="blast.from" placeholder="From (name <email>)" class="border rounded px-3 py-2" />
              <input v-model="blast.subject" placeholder="Subject" class="border rounded px-3 py-2" />
            </div>

            <div class="flex items-center gap-3">
              <label class="flex items-center gap-2">
                <input type="checkbox" v-model="recipientsAllChecked" />
                Select all visible
              </label>
              <div class="ml-auto text-xs text-slate-500">Or pick individual recipients below</div>
            </div>

            <div class="border rounded p-3">
              <div class="flex items-center gap-3 mb-3">
                <input v-model="recipientsSearch" placeholder="Search contacts..." class="border rounded px-3 py-1 w-full" />
                <div class="text-sm text-slate-500">Showing {{ filteredRecipients.length }}</div>
              </div>

              <div class="max-h-48 overflow-auto">
                <div v-for="c in filteredRecipients" :key="c.id" class="flex items-center justify-between py-2 border-b last:border-b-0">
                  <div class="flex items-center gap-3">
                    <input type="checkbox" :checked="recipientSet.has(c.id)" @change="toggleRecipient(c.id)" />
                    <img :src="c.avatar" class="w-8 h-8 rounded-full object-cover" />
                    <div>
                      <div class="font-medium text-slate-800">{{ c.name }}</div>
                      <div class="text-xs text-slate-400">{{ c.email }}</div>
                    </div>
                  </div>
                  <div class="text-xs text-slate-500">{{ c.company }}</div>
                </div>

                <div v-if="filteredRecipients.length === 0" class="py-6 text-center text-slate-400">No contacts</div>
              </div>
            </div>

            <div>
              <div class="text-sm text-slate-500 mb-2">Message</div>
              <div
                ref="editor"
                contenteditable="true"
                class="min-h-[160px] border rounded p-3 prose max-w-full focus:outline-none"
                @input="onEditorInput"
              >{{ blast.body }}</div>
              <div class="text-xs text-slate-400 mt-1">Simple editor. You can paste HTML.</div>
            </div>

            <div v-if="error" class="text-sm text-red-600">{{ error }}</div>
          </div>

          <div class="p-4 border-t flex justify-between items-center gap-2">
            <div class="text-sm text-slate-500">Scheduled: <input type="datetime-local" v-model="blast.scheduledAt" class="ml-2 border rounded px-2 py-1 text-sm" /></div>
            <div class="flex items-center gap-2">
              <button type="button" class="px-4 py-2 rounded border" @click="closeEmailModal">Cancel</button>
              <button type="button" class="px-4 py-2 rounded bg-indigo-600 text-white" @click="sendEmailBlast">Send</button>
            </div>
          </div>
        </form>
      </dialog>

      <!-- preview modal -->
      <dialog ref="previewModal" class="rounded-lg p-0" @close="closePreviewModal">
        <div class="w-[900px] max-w-full bg-white rounded-lg overflow-hidden shadow-lg">
          <div class="p-4 border-b flex items-center justify-between">
            <div><h3 class="text-lg font-semibold">Preview</h3></div>
            <div class="text-sm text-slate-500">Subject: {{ previewBlast.subject }}</div>
          </div>
          <div class="p-4">
            <div class="border rounded p-4 prose max-w-none" v-html="previewBlast.body"></div>
            <div class="mt-4 text-sm text-slate-500">Recipients ({{ previewBlast.recipients.length }}):</div>
            <div class="mt-2 space-y-1">
              <div v-for="r in previewBlast.recipients" :key="r.id" class="text-sm">{{ r.name }} — {{ r.email }}</div>
            </div>
          </div>
          <div class="p-4 border-t text-right">
            <button class="px-4 py-2 rounded border" @click="closePreviewModal">Close</button>
          </div>
        </div>
      </dialog>
    </div>
  </AdminLayout>
</template>

<script setup>
import { ref, computed, onMounted, nextTick } from 'vue'
import AdminLayout from '@/components/layout/AdminLayout.vue'

/* ------------------------
   Sample contacts dataset
   ------------------------ */
const raw = ref([
  { id: 1, name: 'Tonya Noble', role: 'Designer', company: 'Nesta Technologies', email: 'tonyanoble@velzon.com', phone: '414-453-5725', score: 225, tags: ['Lead','Partner'], avatar: 'https://i.pravatar.cc/80?img=10', note: 'Personal note about Tonya.', last_updated:'15 Dec,2021 08:58AM' },
  { id: 2, name: 'Kevin Dawson', role: 'Developer', company: 'Meta4Systems', email: 'kevindawson@velzon.com', phone: '745-321-9874', score: 500, tags: ['Lead'], avatar: 'https://i.pravatar.cc/80?img=5', note: 'Personal note about Kevin' },
  { id: 3, name: 'Timothy Smith', role: 'Manager', company: 'Digitech Galaxy', email: 'timothysmith@velzon.com', phone: '231-480-8536', score: 154, tags: ['Partner'], avatar: 'https://i.pravatar.cc/80?img=7', note: 'Personal note about Timothy' },
  { id: 4, name: 'Michael Morris', role: 'Designer', company: 'Syntyce Solutions', email: 'michaelmorris@velzon.com', phone: '231-480-8536', score: 732, tags: ['Lead','Partner'], avatar: 'https://i.pravatar.cc/80?img=6', note: 'Personal note about Michael' },
  { id: 5, name: 'Herbert Stokes', role: 'Sales', company: 'Themesbrand', email: 'herbertstokes@velzon.com', phone: '949-791-0614', score: 784, tags: ['Exiting','Lead'], avatar: 'https://i.pravatar.cc/80?img=11', note: 'Personal note about Herbert' },
  { id: 6, name: 'Mary Cousar', role: 'CMO', company: 'Micro Design', email: 'marycousar@velzon.com', phone: '540-575-0991', score: 643, tags: ['Exiting'], avatar: 'https://i.pravatar.cc/80?img=12', note: 'Personal note about Mary' },
  { id: 7, name: 'James Price', role: 'Designer', company: 'Themesbrand', email: 'jamesprice@velzon.com', phone: '646-276-2274', score: 81, tags: ['Lead'], avatar: 'https://i.pravatar.cc/80?img=8', note: 'Personal note about James' },
  { id: 8, name: 'Alexis Clarke', role: 'Product', company: 'Android Galaxy', email: 'alexisclarke@velzon.com', phone: '515-395-1069', score: 369, tags: ['Lead','Partner'], avatar: 'https://i.pravatar.cc/80?img=9', note: 'Personal note about Alexis.', last_updated:'15 Dec,2021 08:58AM' },
  { id: 9, name: 'Lina Hart', role: 'HR', company: 'Acme', email: 'lina@acme.com', phone: '111-222-3333', score: 150, tags: ['Partner'], avatar: 'https://i.pravatar.cc/80?img=13', note: '' },
  { id: 10, name: 'Budi Santoso', role: 'Sales', company: 'LocalCo', email: 'budi@local.co', phone: '0812-345-678', score: 210, tags: ['Lead'], avatar: 'https://i.pravatar.cc/80?img=14', note: '' },
  { id: 11, name: 'Siti Aminah', role: 'Support', company: 'Biz', email: 'siti@biz.com', phone: '0813-222-333', score: 80, tags: [], avatar: 'https://i.pravatar.cc/80?img=15', note: '' },
  { id: 12, name: 'Rian Wirawan', role: 'DevOps', company: 'Infra', email: 'rian@infra.com', phone: '0812-999-888', score: 420, tags: ['Lead'], avatar: 'https://i.pravatar.cc/80?img=16', note: '' },
])

/* ------------------------
   History & UI state
   ------------------------ */
const blastsHistory = ref([
  {
    id: 1,
    from: 'noreply@company.com',
    subject: 'Welcome to our service',
    previewText: 'Hello — thanks for joining us...',
    body: '<p>Hello — thanks for joining us!</p>',
    recipients: [{ id: 2, email: 'kevindawson@velzon.com', name: 'Kevin Dawson' }],
    status: 'sent',
    scheduledAt: null,
    createdAt: new Date().toISOString()
  }
])

const active = ref(null)
const form = ref({ name: '', company: '', email: '', phone: '', note: '' })

/* dialog/template refs */
const modal = ref(null)
const emailModal = ref(null)
const previewModal = ref(null)
const editor = ref(null)

/* blast composition */
const blast = ref({
  from: '',
  subject: '',
  body: '',
  scheduledAt: ''
})

const previewBlast = ref({
  subject: '',
  body: '',
  recipients: []
})

const error = ref('')

/* recipient selection (fixed implementation) */
const recipientSet = ref(new Set())
const recipientsSearch = ref('')

const filteredRecipients = computed(() => {
  const s = recipientsSearch.value.trim().toLowerCase()
  return raw.value.filter(c => {
    if (!s) return true
    return (c.name && c.name.toLowerCase().includes(s)) ||
           (c.email && c.email.toLowerCase().includes(s)) ||
           (c.company && c.company.toLowerCase().includes(s))
  })
})

/* computed checkbox: reflect whether ALL visible filtered recipients are selected */
const recipientsAllChecked = computed({
  get() {
    const visible = filteredRecipients.value
    if (visible.length === 0) return false
    return visible.every(c => recipientSet.value.has(c.id))
  },
  set(val) {
    if (val) {
      // select all visible
      recipientSet.value = new Set(filteredRecipients.value.map(c => c.id))
    } else {
      // unselect visible ones (keep any selection outside current filter)
      const newSet = new Set(recipientSet.value)
      for (const c of filteredRecipients.value) newSet.delete(c.id)
      recipientSet.value = newSet
    }
    // reassign already done to ensure reactivity
  }
})

const selectedRecipientsCount = computed(() => recipientSet.value.size)

/* ------------------------
   Helpers / actions
   ------------------------ */
function toggleRecipient(id) {
  if (recipientSet.value.has(id)) recipientSet.value.delete(id)
  else recipientSet.value.add(id)
  // force reactivity
  recipientSet.value = new Set(recipientSet.value)
}

/* Email Blast modal open/close */
function openEmailBlast() {
  blast.value = { from: '', subject: '', body: '', scheduledAt: '' }
  recipientSet.value = new Set()
  recipientsSearch.value = ''
  error.value = ''
  nextTick(() => {
    if (editor.value) editor.value.innerHTML = blast.value.body || ''
    emailModal.value?.showModal()
  })
}
function closeEmailModal() { emailModal.value?.close() }

/* Editor sync */
function onEditorInput(e) { blast.value.body = e.target.innerHTML }

/* Preview */
function viewBlast(b) {
  previewBlast.value = { subject: b.subject, body: b.body, recipients: b.recipients }
  nextTick(() => previewModal.value?.showModal())
}
function closePreviewModal() { previewModal.value?.close() }

/* Send blast: validate and push to history (demo) */
function sendEmailBlast() {
  // sync editor
  blast.value.body = editor.value?.innerHTML || blast.value.body || ''

  // validation
  if (!blast.value.subject || !blast.value.body || blast.value.body.trim() === '') {
    error.value = 'Subject and message body are required.'
    return
  }

  // build recipients list from recipientSet
  const recipients = Array.from(recipientSet.value).map(id => {
    const c = raw.value.find(x => x.id === id)
    return { id: c.id, email: c.email, name: c.name }
  })

  if (recipients.length === 0) {
    error.value = 'Please select at least one recipient (or Select all visible).'
    return
  }

  const payload = {
    id: Math.floor(Math.random() * 1000000),
    from: blast.value.from || 'noreply@yourdomain.com',
    subject: blast.value.subject,
    previewText: (blast.value.body.replace(/<[^>]*>/g, '')).slice(0, 120),
    body: blast.value.body,
    recipients,
    status: blast.value.scheduledAt ? 'scheduled' : 'sent',
    scheduledAt: blast.value.scheduledAt || null,
    createdAt: new Date().toISOString()
  }

  // push to history (demo)
  blastsHistory.value.unshift(payload)

  // close modal
  closeEmailModal()

  // feedback
  alert(`Email Blast queued: ${payload.recipients.length} recipients — status: ${payload.status}`)
}

/* Resend / delete */
function resendBlast(b) {
  const clone = {
    ...b,
    id: Math.floor(Math.random() * 1000000),
    status: 'sent',
    createdAt: new Date().toISOString()
  }
  blastsHistory.value.unshift(clone)
  alert(`Resent blast to ${clone.recipients.length} recipients`)
}
function deleteBlast(id) {
  if (!confirm('Delete this blast?')) return
  blastsHistory.value = blastsHistory.value.filter(b => b.id !== id)
}

/* lifecycle */
onMounted(() => {
  active.value = raw.value[0] || null
  // template refs are bound automatically; editor/emailModal/previewModal/modal available
})
</script>

<style scoped>
dialog::backdrop {
  background: rgba(0,0,0,0.3);
}
.prose a { text-decoration: underline; }
</style>
