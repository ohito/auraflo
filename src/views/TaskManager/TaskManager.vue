<template>
  <AdminLayout>
    <div class="p-6">
      <!-- HEADER -->
      <div class="mb-6">
        <div class="bg-white rounded-2xl border border-gray-200 p-4 shadow-sm flex flex-col md:flex-row items-center justify-between gap-4">
          <div class="flex items-center gap-4 min-w-0 w-full md:w-auto">
            <h2 class="text-2xl font-semibold whitespace-nowrap">Task Manager</h2>

            <div class="relative flex-1 min-w-0 md:w-[520px]">
              <span class="absolute left-3 top-1/2 -translate-y-1/2 text-slate-400">
                <SearchIcon class="w-4 h-4" />
              </span>
              <input
                v-model="q"
                type="search"
                placeholder="Search tasks by title, assignee or project..."
                class="w-full h-10 rounded-xl border border-gray-200 px-4 pl-10 text-sm placeholder:text-slate-400 focus:outline-none"
              />
            </div>
          </div>

          <div class="flex items-center gap-3 w-full md:w-auto">
            <select v-model="statusFilter" class="h-9 rounded-xl border border-gray-200 bg-white px-3 text-sm">
              <option value="">All status</option>
              <option value="todo">To Do</option>
              <option value="in_progress">In Progress</option>
              <option value="done">Done</option>
            </select>

            <select v-model="assigneeFilter" class="h-9 rounded-xl border border-gray-200 bg-white px-3 text-sm">
              <option value="">All assignees</option>
              <option v-for="u in uniqueAssignees" :key="u" :value="u">{{ u }}</option>
            </select>

            <button @click="openAdd" class="inline-flex items-center gap-2 rounded-lg bg-sky-500 px-4 py-2 text-white text-sm font-medium hover:bg-sky-600 shadow">+ Add Task</button>
          </div>
        </div>
      </div>

      <!-- BULK ACTIONS + LIST -->
      <div class="bg-white rounded-2xl border border-gray-200 p-4 shadow-sm">
        <!-- bulk actions -->
        <div class="mb-3 flex items-center justify-between">
          <div class="flex items-center gap-3">
            <div class="text-sm text-slate-500">Bulk:</div>
            <button :disabled="selected.size === 0" class="px-3 py-1 rounded border text-sm" @click="bulkChangeStatus('todo')">Mark To Do</button>
            <button :disabled="selected.size === 0" class="px-3 py-1 rounded border text-sm" @click="bulkChangeStatus('in_progress')">Mark In Progress</button>
            <button :disabled="selected.size === 0" class="px-3 py-1 rounded border text-sm" @click="bulkChangeStatus('done')">Mark Done</button>
            <button :disabled="selected.size === 0" class="px-3 py-1 rounded border text-sm text-rose-600" @click="bulkDelete">Delete</button>
          </div>

          <div class="text-sm text-slate-500">Showing {{ filtered.length }} tasks</div>
        </div>

        <!-- table -->
        <div class="overflow-x-auto">
          <table class="min-w-full text-sm">
            <thead>
              <tr class="text-left text-xs text-slate-500">
                <th class="px-3 py-2 w-8">
                  <input type="checkbox" :checked="allSelected" @change="toggleSelectAll($event)" />
                </th>
                <th class="px-3 py-2">Task</th>
                <th class="px-3 py-2">Project</th>
                <th class="px-3 py-2">Assignee</th>
                <th class="px-3 py-2">Priority</th>
                <th class="px-3 py-2">Due</th>
                <th class="px-3 py-2">Status</th>
                <th class="px-3 py-2">Actions</th>
              </tr>
            </thead>

            <tbody>
              <tr v-for="task in paged" :key="task.id" class="border-t border-gray-100 hover:bg-gray-50">
                <td class="px-3 py-3 align-top">
                  <input type="checkbox" :checked="isSelected(task.id)" @click.stop="toggleSelect(task.id)" />
                </td>

                <td class="px-3 py-3 align-top" @click="openDetail(task)">
                  <div class="font-medium text-slate-800 truncate" :title="task.title">{{ task.title }}</div>
                  <div class="text-xs text-slate-400">{{ task.description_short }}</div>
                </td>

                <td class="px-3 py-3 align-top text-slate-700">{{ task.project }}</td>

                <td class="px-3 py-3 align-top">
                  <div class="flex items-center gap-3">
                    <img :src="task.assigneeAvatar" alt="" class="w-8 h-8 rounded-full object-cover border" />
                    <div class="text-sm">
                      <div class="font-medium">{{ task.assignee }}</div>
                      <div class="text-xs text-slate-400">{{ task.assignee_role || '' }}</div>
                    </div>
                  </div>
                </td>

                <td class="px-3 py-3 align-top">
                  <span :class="priorityClass(task.priority)">{{ task.priority }}</span>
                </td>

                <td class="px-3 py-3 align-top">{{ task.due || '—' }}</td>

                <td class="px-3 py-3 align-top">
                  <span :class="statusBadge(task.status)">{{ statusLabel(task.status) }}</span>
                </td>

                <td class="px-3 py-3 align-top">
                  <div class="flex items-center gap-2">
                    <button class="px-3 py-1 rounded border text-sm" @click.stop="openEdit(task)">Edit</button>
                    <button class="px-3 py-1 rounded border text-sm" @click.stop="deleteTask(task.id)">Delete</button>
                  </div>
                </td>
              </tr>

              <tr v-if="filtered.length === 0">
                <td colspan="8" class="p-6 text-center text-slate-400">No tasks found</td>
              </tr>
            </tbody>
          </table>
        </div>

        <!-- pagination -->
        <div class="mt-4 flex items-center justify-between">
          <div class="text-sm text-slate-500">Page {{ page }} / {{ totalPages }}</div>
          <div class="flex items-center gap-2">
            <button @click="prevPage" :disabled="page === 1" class="px-3 py-1 rounded border text-sm disabled:opacity-50">Prev</button>
            <button v-for="p in pages" :key="p" @click="page = p" :class="['px-3 py-1 rounded text-sm', page === p ? 'bg-slate-800 text-white' : 'border']">{{ p }}</button>
            <button @click="nextPage" :disabled="page === totalPages" class="px-3 py-1 rounded border text-sm disabled:opacity-50">Next</button>
          </div>
        </div>
      </div>

      <!-- create / edit modal -->
      <dialog ref="modal" class="rounded-lg p-0" @close="closeModal">
        <form method="dialog" class="w-[720px] max-w-full bg-white rounded-lg overflow-hidden shadow-lg">
          <div class="p-5 border-b flex items-center justify-between">
            <h3 class="text-lg font-semibold">{{ editMode ? 'Edit Task' : 'Create Task' }}</h3>
            <div class="text-sm text-slate-400">{{ form.id ? 'ID: ' + form.id : '' }}</div>
          </div>

          <div class="p-5 space-y-3">
            <div class="grid md:grid-cols-2 gap-3">
              <input v-model="form.title" placeholder="Title" class="border rounded px-3 py-2" />
              <input v-model="form.project" placeholder="Project" class="border rounded px-3 py-2" />
              <select v-model="form.priority" class="border rounded px-3 py-2">
                <option value="Low">Low</option>
                <option value="Normal">Normal</option>
                <option value="High">High</option>
                <option value="Critical">Critical</option>
              </select>
              <input v-model="form.due" type="date" class="border rounded px-3 py-2" />
              <select v-model="form.assignee" class="border rounded px-3 py-2">
                <option value="">Unassigned</option>
                <option v-for="u in uniqueAssignees" :key="u" :value="u">{{ u }}</option>
              </select>
              <select v-model="form.status" class="border rounded px-3 py-2">
                <option value="todo">To Do</option>
                <option value="in_progress">In Progress</option>
                <option value="done">Done</option>
              </select>
            </div>

            <textarea v-model="form.description" placeholder="Description" class="w-full border rounded px-3 py-2"></textarea>
          </div>

          <div class="p-4 border-t flex justify-end gap-2">
            <button type="button" class="px-4 py-2 rounded border" @click="closeModal">Cancel</button>
            <button type="button" class="px-4 py-2 rounded bg-emerald-500 text-white" @click="saveTask">{{ editMode ? 'Save' : 'Create' }}</button>
          </div>
        </form>
      </dialog>

      <!-- detail slide-over -->
      <div v-if="showDetail" class="fixed inset-0 z-40">
        <div class="absolute inset-0 bg-black/30" @click="closeDetail"></div>
        <aside class="absolute right-6 top-20 w-[420px] bg-white rounded-xl shadow-2xl border p-5 overflow-auto max-h-[80vh]">
          <div class="flex items-start justify-between gap-3">
            <div>
              <div class="text-lg font-semibold">{{ activeTask.title }}</div>
              <div class="text-xs text-slate-500">{{ activeTask.project }} • Due: {{ activeTask.due || '—' }}</div>
            </div>
            <button class="text-slate-500" @click="closeDetail">✕</button>
          </div>

          <div class="mt-4 text-sm text-slate-700">
            <div class="mb-3">
              <div class="font-semibold">Assignee</div>
              <div class="mt-2 flex items-center gap-3">
                <img :src="activeTask.assigneeAvatar" class="w-10 h-10 rounded-full border object-cover" />
                <div>
                  <div class="font-medium">{{ activeTask.assignee || 'Unassigned' }}</div>
                  <div class="text-xs text-slate-400">{{ activeTask.assignee_role || '' }}</div>
                </div>
              </div>
            </div>

            <div class="mb-3">
              <div class="font-semibold">Priority</div>
              <div class="mt-2">{{ activeTask.priority }}</div>
            </div>

            <div class="mb-3">
              <div class="font-semibold">Description</div>
              <div class="mt-2 text-sm text-slate-600" v-html="activeTask.description || '<em>No description</em>'"></div>
            </div>

            <div class="mt-4 flex gap-3">
              <button class="px-3 py-2 rounded bg-sky-600 text-white" @click="openEdit(activeTask)">Edit</button>
              <button class="px-3 py-2 rounded border" @click="deleteTask(activeTask.id)">Delete</button>
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

/* sample tasks */
const initialTasks = [
  { id: 1, title: 'Design landing hero', description: 'Create hero section for campaign', description_short: 'Create hero section', project: 'Website Redesign', assignee: 'Anna', assigneeAvatar: 'https://i.pravatar.cc/64?img=21', assignee_role: 'Designer', priority: 'High', due: '2025-11-05', status: 'in_progress' },
  { id: 2, title: 'Integrate payment gateway', description: 'Add Xendit integration and test flows', description_short: 'Xendit integration', project: 'Checkout v2', assignee: 'Budi', assigneeAvatar: 'https://i.pravatar.cc/64?img=14', assignee_role: 'Backend', priority: 'Critical', due: '2025-10-28', status: 'todo' },
  { id: 3, title: 'Prepare monthly report', description: 'Collect metrics and compile report', description_short: 'Monthly metrics', project: 'Operations', assignee: 'Citra', assigneeAvatar: 'https://i.pravatar.cc/64?img=15', assignee_role: 'Ops', priority: 'Normal', due: '2025-11-01', status: 'todo' },
  { id: 4, title: 'Fix checkout bug', description: 'Orders failing with 422 in prod', description_short: 'Orders 422 bug', project: 'Checkout v2', assignee: 'Dedi', assigneeAvatar: 'https://i.pravatar.cc/64?img=12', assignee_role: 'Backend', priority: 'High', due: '', status: 'in_progress' },
  { id: 5, title: 'QA release 1.4', description: 'Run regression suite', description_short: 'Regression QA', project: 'Release', assignee: 'Eka', assigneeAvatar: 'https://i.pravatar.cc/64?img=8', assignee_role: 'QA', priority: 'Normal', due: '2025-11-03', status: 'done' },
]

const tasks = ref(JSON.parse(JSON.stringify(initialTasks)))

/* filters / search / pagination */
const q = ref('')
const statusFilter = ref('')
const assigneeFilter = ref('')
const page = ref(1)
const perPage = ref(8)

/* selection */
const selected = ref(new Set())

/* modal/form/detail */
const modal = ref(null)
const editMode = ref(false)
const form = reactive({
  id: null, title: '', project: '', assignee: '', assigneeAvatar: '', assignee_role: '', priority: 'Normal', due: '', status: 'todo', description: ''
})

const showDetail = ref(false)
const activeTask = reactive({})

/* computed lists */
const filtered = computed(() => {
  const s = q.value.trim().toLowerCase()
  return tasks.value
    .filter(t => {
      if (statusFilter.value && t.status !== statusFilter.value) return false
      if (assigneeFilter.value && t.assignee !== assigneeFilter.value) return false
      if (!s) return true
      return t.title.toLowerCase().includes(s) || (t.assignee && t.assignee.toLowerCase().includes(s)) || (t.project && t.project.toLowerCase().includes(s))
    })
    .sort((a,b) => {
      // sort: critical/high first, then due date asc
      const prio = { 'Critical': 0, 'High': 1, 'Normal': 2, 'Low': 3 }
      if (prio[a.priority] !== prio[b.priority]) return prio[a.priority] - prio[b.priority]
      if (a.due && b.due) return new Date(a.due) - new Date(b.due)
      if (a.due) return -1
      if (b.due) return 1
      return a.title.localeCompare(b.title)
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
  if (e.target.checked) selected.value = new Set(filtered.value.map(t => t.id))
  else selected.value = new Set()
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

/* unique assignees */
const uniqueAssignees = computed(() => {
  const set = new Set(tasks.value.map(t => t.assignee).filter(Boolean))
  return Array.from(set)
})

/* modal create/edit */
function openAdd() {
  editMode.value = false
  form.id = null
  form.title = ''
  form.project = ''
  form.assignee = ''
  form.assigneeAvatar = ''
  form.assignee_role = ''
  form.priority = 'Normal'
  form.due = ''
  form.status = 'todo'
  form.description = ''
  modal.value?.showModal?.()
}
function openEdit(task) {
  editMode.value = true
  form.id = task.id
  form.title = task.title
  form.project = task.project
  form.assignee = task.assignee
  form.assigneeAvatar = task.assigneeAvatar || `https://i.pravatar.cc/64?img=${Math.floor(Math.random()*80)+1}`
  form.assignee_role = task.assignee_role || ''
  form.priority = task.priority
  form.due = task.due
  form.status = task.status
  form.description = task.description || ''
  modal.value?.showModal?.()
}
function closeModal() { modal.value?.close?.() }

function saveTask() {
  if (!form.title || !form.project) {
    alert('Title & project required')
    return
  }
  if (editMode.value && form.id) {
    const idx = tasks.value.findIndex(t => t.id === form.id)
    if (idx !== -1) {
      tasks.value[idx] = {
        id: form.id,
        title: form.title,
        project: form.project,
        assignee: form.assignee,
        assigneeAvatar: form.assigneeAvatar,
        assignee_role: form.assignee_role,
        priority: form.priority,
        due: form.due,
        status: form.status,
        description: form.description,
        description_short: form.description ? form.description.slice(0,60) : ''
      }
    }
  } else {
    const id = Math.max(0, ...tasks.value.map(t => t.id)) + 1
    tasks.value.unshift({
      id,
      title: form.title,
      project: form.project,
      assignee: form.assignee,
      assigneeAvatar: form.assigneeAvatar || `https://i.pravatar.cc/64?img=${Math.floor(Math.random()*80)+1}`,
      assignee_role: form.assignee_role || '',
      priority: form.priority,
      due: form.due,
      status: form.status,
      description: form.description,
      description_short: form.description ? form.description.slice(0,60) : ''
    })
  }
  rebuildPagination()
  closeModal()
}

/* delete */
function deleteTask(id) {
  if (!confirm('Delete this task?')) return
  tasks.value = tasks.value.filter(t => t.id !== id)
  selected.value.delete(id)
  rebuildPagination()
}

/* bulk operations */
function bulkChangeStatus(status) {
  if (selected.value.size === 0) return
  const ids = Array.from(selected.value)
  tasks.value.forEach(t => {
    if (ids.includes(t.id)) t.status = status
  })
  // clear selection
  selected.value = new Set()
}
function bulkDelete() {
  if (!confirm('Delete selected tasks?')) return
  const ids = new Set(Array.from(selected.value))
  tasks.value = tasks.value.filter(t => !ids.has(t.id))
  selected.value = new Set()
  rebuildPagination()
}

/* detail panel */
function openDetail(task) {
  Object.assign(activeTask, task)
  showDetail.value = true
}
function closeDetail() {
  showDetail.value = false
  // clear activeTask
  Object.keys(activeTask).forEach(k => delete activeTask[k])
}

/* misc */
function rebuildPagination() {
  if (page.value > totalPages.value) page.value = totalPages.value
}

/* UI helpers */
function statusLabel(s) { if (!s) return '—'; return s === 'todo' ? 'To Do' : s === 'in_progress' ? 'In Progress' : 'Done' }
function statusBadge(s) {
  if (s === 'todo') return 'px-2 py-1 rounded text-xs bg-slate-100 text-slate-700'
  if (s === 'in_progress') return 'px-2 py-1 rounded text-xs bg-amber-100 text-amber-700'
  if (s === 'done') return 'px-2 py-1 rounded text-xs bg-emerald-100 text-emerald-700'
  return 'px-2 py-1 rounded text-xs bg-slate-100 text-slate-700'
}
function priorityClass(p) {
  if (!p) return ''
  if (p === 'Critical') return 'px-2 py-1 rounded text-xs bg-rose-100 text-rose-700 font-semibold'
  if (p === 'High') return 'px-2 py-1 rounded text-xs bg-amber-100 text-amber-700 font-semibold'
  if (p === 'Normal') return 'px-2 py-1 rounded text-xs bg-sky-100 text-sky-700'
  return 'px-2 py-1 rounded text-xs bg-slate-100 text-slate-700'
}

/* lifecycle */
onMounted(() => {
  rebuildPagination()
})
</script>

<style scoped>
/* small UI helpers */
.col-title { font-weight: 700; }

/* ensure slide panel sits above other elements */
.fixed.inset-0 { z-index: 60; }

/* responsive tweaks */
@media (max-width: 900px) {
  .header-card { flex-direction: column; gap: 8px; }
}
</style>
