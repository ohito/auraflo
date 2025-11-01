<template>
  <AdminLayout>
    <div class="p-6">
      <!-- header -->
      <div class="mb-6">
        <div class="bg-white rounded-2xl border border-gray-200 p-5 flex items-center justify-between">
          <div>
            <h1 class="text-2xl font-bold">Manage</h1>
            <div class="text-sm text-slate-500">Team, roles, API keys and audit logs</div>
          </div>

          <div class="flex items-center gap-3">
            <div class="relative">
              <input v-model="q" placeholder="Search members, roles..." class="h-10 w-64 rounded-lg border px-3 text-sm placeholder:text-slate-400" />
              <button v-if="q" @click="q=''" class="absolute right-2 top-1/2 -translate-y-1/2 text-slate-400 text-xs">✕</button>
            </div>
            <button class="px-4 py-2 rounded bg-emerald-500 text-white" @click="openAddMember">+ Add Member</button>
          </div>
        </div>
      </div>

      <!-- tabs -->
      <div class="mb-6 bg-white rounded-2xl border border-gray-200 p-4">
        <nav class="flex gap-4 border-b pb-3 mb-4">
          <button @click="activeTab='team'" :class="tabClass('team')">Team Members</button>
          <button @click="activeTab='roles'" :class="tabClass('roles')">Roles & Permissions</button>
          <button @click="activeTab='keys'" :class="tabClass('keys')">API Keys</button>
          <button @click="activeTab='audit'" :class="tabClass('audit')">Audit Log</button>
        </nav>

        <!-- TEAM -->
        <section v-if="activeTab === 'team'">
          <div class="mb-4 flex items-center justify-between">
            <div class="text-sm text-slate-500">Showing {{ filteredMembers.length }} members</div>
            <div class="flex items-center gap-2">
              <select v-model="filterRole" class="h-9 rounded-lg border px-3 text-sm">
                <option value="">All roles</option>
                <option v-for="r in roles" :key="r.key" :value="r.key">{{ r.name }}</option>
              </select>
            </div>
          </div>

          <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
            <div v-for="m in pagedMembers" :key="m.id" class="bg-white rounded-lg border p-4 flex flex-col">
              <div class="flex items-center gap-3">
                <img :src="m.avatar" class="w-12 h-12 rounded-full object-cover border" />
                <div class="flex-1 min-w-0">
                  <div class="font-semibold truncate">{{ m.name }}</div>
                  <div class="text-xs text-slate-400 truncate">{{ m.email }}</div>
                </div>
                <div class="text-xs text-slate-500">{{ findRoleName(m.role) }}</div>
              </div>

              <div class="mt-4 flex gap-2">
                <button class="px-3 py-1 rounded border text-sm" @click="openEditMember(m)">Edit</button>
                <button class="px-3 py-1 rounded border text-sm text-rose-600" @click="removeMember(m.id)">Remove</button>
                <button class="ml-auto px-3 py-1 rounded border text-sm" @click="impersonate(m)">Impersonate</button>
              </div>
            </div>
          </div>

          <!-- pagination -->
          <div class="mt-4 flex items-center justify-between">
            <div class="text-sm text-slate-500">Page {{ memberPage }} / {{ memberTotalPages }}</div>
            <div class="flex items-center gap-2">
              <button @click="memberPrev" :disabled="memberPage===1" class="px-3 py-1 rounded border">Prev</button>
              <button v-for="p in memberPages" :key="p" @click="memberPage=p" :class="['px-3 py-1 rounded', memberPage===p ? 'bg-slate-800 text-white' : 'border']">{{ p }}</button>
              <button @click="memberNext" :disabled="memberPage===memberTotalPages" class="px-3 py-1 rounded border">Next</button>
            </div>
          </div>
        </section>

        <!-- ROLES -->
        <section v-if="activeTab === 'roles'">
          <div class="mb-4 flex items-center justify-between">
            <div class="text-sm text-slate-500">Manage roles and their permissions</div>
            <div>
              <button class="px-3 py-1 rounded bg-sky-600 text-white" @click="openAddRole">+ Add Role</button>
            </div>
          </div>

          <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
            <div v-for="r in roles" :key="r.key" class="bg-white border rounded-lg p-4">
              <div class="flex items-start gap-3">
                <div class="flex-1">
                  <div class="font-semibold">{{ r.name }}</div>
                  <div class="text-xs text-slate-400 mt-1">{{ r.description }}</div>

                  <div class="mt-3 flex flex-wrap gap-2">
                    <span v-for="p in r.permissions" :key="p" class="text-xs px-2 py-1 rounded bg-slate-100">{{ p }}</span>
                  </div>
                </div>

                <div class="flex flex-col gap-2">
                  <button class="px-3 py-1 rounded border" @click="openEditRole(r)">Edit</button>
                  <button class="px-3 py-1 rounded border text-rose-600" @click="removeRole(r.key)">Delete</button>
                </div>
              </div>
            </div>
          </div>
        </section>

        <!-- KEYS -->
        <section v-if="activeTab === 'keys'">
          <div class="mb-4 flex items-center justify-between">
            <div class="text-sm text-slate-500">Manage API keys for integrations</div>
            <div>
              <button class="px-3 py-1 rounded bg-amber-500 text-white" @click="createApiKey">Create API Key</button>
            </div>
          </div>

          <div class="space-y-3">
            <div v-for="k in apiKeys" :key="k.id" class="bg-white rounded-lg border p-4 flex items-center gap-4">
              <div class="flex-1">
                <div class="font-semibold">{{ k.name }}</div>
                <div class="text-xs text-slate-400">{{ k.keyMasked }}</div>
                <div class="text-xs text-slate-400 mt-1">Created: {{ k.createdAt }}</div>
              </div>

              <div class="flex gap-2">
                <button class="px-3 py-1 rounded border" @click="revealKey(k)">{{ k.revealed ? 'Hide' : 'Reveal' }}</button>
                <button class="px-3 py-1 rounded border" @click="copyKey(k)">Copy</button>
                <button class="px-3 py-1 rounded border text-rose-600" @click="revokeKey(k.id)">Revoke</button>
              </div>
            </div>
          </div>
        </section>

        <!-- AUDIT -->
        <section v-if="activeTab === 'audit'">
          <div class="mb-4 text-sm text-slate-500">Recent activity (audit)</div>

          <div class="bg-white rounded-lg border p-3">
            <div v-for="a in recentAudit" :key="a.id" class="py-3 border-b last:border-b-0 flex items-start gap-3">
              <div class="text-xs text-slate-400 w-28">{{ a.time }}</div>
              <div>
                <div class="font-medium">{{ a.action }}</div>
                <div class="text-xs text-slate-500 mt-1">{{ a.meta }}</div>
              </div>
            </div>
          </div>
        </section>
      </div>
    </div>

    <!-- Add / Edit Member Modal -->
    <dialog ref="memberModal" class="rounded-lg p-0" @close="closeMemberModal">
      <form method="dialog" class="w-[540px] max-w-full bg-white rounded-lg overflow-hidden shadow-lg">
        <div class="p-5 border-b">
          <h3 class="text-lg font-semibold">{{ editingMember ? 'Edit Member' : 'Add Member' }}</h3>
        </div>

        <div class="p-5 space-y-3">
          <input v-model="memberForm.name" placeholder="Full name" class="w-full border rounded px-3 py-2" />
          <input v-model="memberForm.email" placeholder="Email" class="w-full border rounded px-3 py-2" />
          <select v-model="memberForm.role" class="w-full border rounded px-3 py-2">
            <option v-for="r in roles" :key="r.key" :value="r.key">{{ r.name }}</option>
          </select>
          <input v-model="memberForm.avatar" placeholder="Avatar URL" class="w-full border rounded px-3 py-2" />
        </div>

        <div class="p-4 border-t flex justify-end gap-2">
          <button type="button" class="px-4 py-2 rounded border" @click="closeMemberModal">Cancel</button>
          <button type="button" class="px-4 py-2 rounded bg-emerald-500 text-white" @click="saveMember">{{ editingMember ? 'Save' : 'Add' }}</button>
        </div>
      </form>
    </dialog>

    <!-- Add / Edit Role Modal -->
    <dialog ref="roleModal" class="rounded-lg p-0" @close="closeRoleModal">
      <form method="dialog" class="w-[520px] max-w-full bg-white rounded-lg overflow-hidden shadow-lg">
        <div class="p-5 border-b">
          <h3 class="text-lg font-semibold">{{ editingRole ? 'Edit Role' : 'Add Role' }}</h3>
        </div>

        <div class="p-5 space-y-3">
          <input v-model="roleForm.name" placeholder="Role name" class="w-full border rounded px-3 py-2" />
          <input v-model="roleForm.key" placeholder="Role key (identifier)" class="w-full border rounded px-3 py-2" />
          <textarea v-model="roleForm.description" placeholder="Description" class="w-full border rounded px-3 py-2"></textarea>

          <div>
            <div class="text-sm font-semibold mb-2">Permissions</div>
            <div class="flex flex-wrap gap-2">
              <label v-for="p in permissionOptions" :key="p" class="text-xs px-2 py-1 rounded border cursor-pointer">
                <input type="checkbox" :value="p" v-model="roleForm.permissions" class="mr-1" />{{ p }}
              </label>
            </div>
          </div>
        </div>

        <div class="p-4 border-t flex justify-end gap-2">
          <button type="button" class="px-4 py-2 rounded border" @click="closeRoleModal">Cancel</button>
          <button type="button" class="px-4 py-2 rounded bg-sky-600 text-white" @click="saveRole">{{ editingRole ? 'Save' : 'Create' }}</button>
        </div>
      </form>
    </dialog>
  </AdminLayout>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import AdminLayout from '@/components/layout/AdminLayout.vue'

/* STATE: members (demo data) */
const members = ref([
  { id: 1, name: 'Anna Adame', email: 'anna@example.com', role: 'admin', avatar: 'https://i.pravatar.cc/64?img=21' },
  { id: 2, name: 'Budi Santoso', email: 'budi@example.com', role: 'manager', avatar: 'https://i.pravatar.cc/64?img=32' },
  { id: 3, name: 'Siti Aminah', email: 'siti@example.com', role: 'member', avatar: 'https://i.pravatar.cc/64?img=15' },
  { id: 4, name: 'Rian Wirawan', email: 'rian@example.com', role: 'member', avatar: 'https://i.pravatar.cc/64?img=16' },
])

/* roles */
const roles = ref([
  { key: 'admin', name: 'Administrator', description: 'Full access to everything', permissions: ['manage_team','manage_roles','manage_billing'] },
  { key: 'manager', name: 'Manager', description: 'Manage projects & teams', permissions: ['manage_team','view_reports'] },
  { key: 'member', name: 'Member', description: 'Regular user', permissions: ['view_projects'] },
])

/* API keys */
const apiKeys = ref([
  { id: 'k_1', name: 'Default key', keyMasked: 'sk_live_*****abcd', rawKey: 'sk_live_1234abcd', createdAt: '2025-10-01', revealed: false },
])

/* audit log demo */
const recentAudit = ref([
  { id: 1, time: '2m ago', action: 'Member added: Budi Santoso', meta: 'by admin' },
  { id: 2, time: '1h ago', action: 'API key revoked', meta: 'by Anna' },
  { id: 3, time: '1d ago', action: 'Role updated: Manager', meta: 'by admin' },
])

/* UI state */
const q = ref('')
const activeTab = ref('team')
const filterRole = ref('')

/* MEMBER pagination */
const memberPage = ref(1)
const perPage = ref(6)
const memberPages = computed(() => {
  const pages = []
  for (let i = 1; i <= memberTotalPages.value; i++) pages.push(i)
  return pages
})
const memberTotalPages = computed(() => Math.max(1, Math.ceil(filteredMembers.value.length / perPage.value)))

/* filtered & paged */
const filteredMembers = computed(() => {
  const s = q.value.trim().toLowerCase()
  return members.value.filter(m => {
    if (filterRole.value && m.role !== filterRole.value) return false
    if (!s) return true
    return m.name.toLowerCase().includes(s) || m.email.toLowerCase().includes(s)
  })
})
const pagedMembers = computed(() => {
  const start = (memberPage.value - 1) * perPage.value
  return filteredMembers.value.slice(start, start + perPage.value)
})
function memberPrev() { if (memberPage.value > 1) memberPage.value-- }
function memberNext() { if (memberPage.value < memberTotalPages.value) memberPage.value++ }

/* helpers */
function findRoleName(key) {
  const r = roles.value.find(x => x.key === key)
  return r ? r.name : key
}

/* TAB CSS helper */
function tabClass(key) {
  return {
    'px-3 py-2 text-sm rounded-lg': true,
    'bg-slate-100': activeTab.value === key,
    'text-slate-700': activeTab.value === key,
    'text-slate-500': activeTab.value !== key
  }
}

/* MEMBER modal */
const memberModal = ref(null)
const editingMember = ref(null)
const memberForm = reactive({ id: null, name: '', email: '', role: 'member', avatar: '' })

function openAddMember() {
  editingMember.value = null
  memberForm.id = null
  memberForm.name = ''
  memberForm.email = ''
  memberForm.role = 'member'
  memberForm.avatar = `https://i.pravatar.cc/64?img=${Math.floor(Math.random()*60)+1}`
  memberModal.value?.showModal?.()
}
function openEditMember(m) {
  editingMember.value = m.id
  memberForm.id = m.id
  memberForm.name = m.name
  memberForm.email = m.email
  memberForm.role = m.role
  memberForm.avatar = m.avatar
  memberModal.value?.showModal?.()
}
function closeMemberModal() { memberModal.value?.close?.() }
function saveMember() {
  if (!memberForm.name || !memberForm.email) { alert('Name & email required'); return }
  if (editingMember.value) {
    const idx = members.value.findIndex(x => x.id === editingMember.value)
    if (idx !== -1) members.value[idx] = { id: editingMember.value, name: memberForm.name, email: memberForm.email, role: memberForm.role, avatar: memberForm.avatar }
  } else {
    const id = Math.floor(Math.random() * 100000)
    members.value.unshift({ id, name: memberForm.name, email: memberForm.email, role: memberForm.role, avatar: memberForm.avatar })
  }
  closeMemberModal()
}

/* remove member */
function removeMember(id) {
  if (!confirm('Remove member?')) return
  members.value = members.value.filter(m => m.id !== id)
}

/* impersonate (demo) */
function impersonate(m) {
  alert(`Start impersonating ${m.name} (demo)`)
}

/* ROLES modal */
const roleModal = ref(null)
const editingRole = ref(null)
const roleForm = reactive({ key: '', name: '', description: '', permissions: [] })
const permissionOptions = ['manage_team','manage_roles','manage_billing','view_reports','view_projects']

function openAddRole() {
  editingRole.value = null
  roleForm.key = ''
  roleForm.name = ''
  roleForm.description = ''
  roleForm.permissions = []
  roleModal.value?.showModal?.()
}
function openEditRole(r) {
  editingRole.value = r.key
  roleForm.key = r.key
  roleForm.name = r.name
  roleForm.description = r.description
  roleForm.permissions = [...r.permissions]
  roleModal.value?.showModal?.()
}
function closeRoleModal() { roleModal.value?.close?.() }
function saveRole() {
  if (!roleForm.key || !roleForm.name) { alert('Key & name required'); return }
  if (editingRole.value) {
    const idx = roles.value.findIndex(x => x.key === editingRole.value)
    if (idx !== -1) roles.value[idx] = { key: roleForm.key, name: roleForm.name, description: roleForm.description, permissions: [...roleForm.permissions] }
  } else {
    roles.value.push({ key: roleForm.key, name: roleForm.name, description: roleForm.description, permissions: [...roleForm.permissions] })
  }
  closeRoleModal()
}
function removeRole(key) {
  if (!confirm('Delete role?')) return
  roles.value = roles.value.filter(r => r.key !== key)
}

/* API KEYS */
function createApiKey() {
  const id = 'k_' + Math.random().toString(36).slice(2, 8)
  const raw = 'sk_' + Math.random().toString(36).slice(2, 12)
  apiKeys.value.unshift({ id, name: `Key ${apiKeys.value.length+1}`, keyMasked: mask(raw), rawKey: raw, createdAt: new Date().toISOString().slice(0,10), revealed: true })
  setTimeout(() => apiKeys.value[0].revealed = false, 4000) // hide after a few sec demo
}
function mask(raw) {
  if (!raw) return '*****'
  return raw.slice(0,8) + '*****' + raw.slice(-4)
}
function revokeKey(id) {
  if (!confirm('Revoke API key?')) return
  apiKeys.value = apiKeys.value.filter(k => k.id !== id)
}
function revealKey(k) { k.revealed = !k.revealed; if (k.revealed) k.keyMasked = k.rawKey }
function copyKey(k) {
  const s = k.revealed ? k.rawKey : k.keyMasked
  navigator.clipboard?.writeText(s).then(()=> alert('Copied to clipboard (demo)'))
}

/* audit filter */
const recentAuditFiltered = computed(()=> recentAudit.value) // could filter by q/time

onMounted(()=> {
  // ensure pages reset when filter changes
  watchFilter()
})

/* watch q or filterRole to reset pagination */
import { watch } from 'vue'
function watchFilter() {
  watch([q, filterRole], () => {
    memberPage.value = 1
  })
}
</script>

<style scoped>
/* small helpers */
.tab-active { background: #f1f5f9; }
/* ensure dialog backdrop */
dialog::backdrop { background: rgba(0,0,0,0.3); }

/* small card tweaks */
.bg-white { background: #fff; }
</style>
