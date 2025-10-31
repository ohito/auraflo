<template>
  <div class="w-full h-full">
    <div class="overflow-hidden rounded-2xl border border-gray-100 bg-white shadow-sm h-full flex flex-col">
      <!-- header -->
      <div class="flex items-center justify-between px-5 py-4">
        <div>
          <h3 class="text-lg font-semibold text-slate-800">My Tasks</h3>
          <div class="text-sm text-slate-400 mt-1">{{ remaining }} of {{ maxTasks }} remaining</div>
        </div>

        <div class="flex items-center gap-3">
          <button
            @click="toggleAdd"
            class="inline-flex items-center gap-2 rounded-md bg-emerald-500 px-3.5 py-1.5 text-sm font-medium text-white hover:bg-emerald-600 shadow-sm"
          >
            <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4" />
            </svg>
            Add Task
          </button>

          <!-- settings icon: adjust import path if necessary -->
          <button class="p-2 rounded-md hover:bg-slate-50" title="Settings">
            <SettingsIcon class="h-5 w-5 text-slate-400" />
          </button>
        </div>
      </div>

      <div class="border-t border-dashed border-slate-100"></div>

      <!-- add task area (not growing) -->
      <transition name="fade">
        <div v-if="showAdd" class="px-5 py-4">
          <div class="flex gap-2">
            <input v-model="newTaskTitle" type="text" placeholder="New task title"
                   class="flex-1 rounded-md border border-gray-200 px-3 py-2 text-sm focus:outline-none focus:ring-1 focus:ring-emerald-300" />
            <input v-model="newTaskDate" type="date"
                   class="rounded-md border border-gray-200 px-3 py-2 text-sm focus:outline-none" />
            <button @click="addTask" class="rounded-md bg-emerald-500 px-4 py-2 text-white text-sm hover:bg-emerald-600">Save</button>
          </div>
        </div>
      </transition>

      <!-- list: grow so footer stays bottom -->
      <ul class="divide-y divide-dashed divide-slate-100 px-5 flex-1 overflow-auto">
        <li v-for="(task, idx) in visibleTasks" :key="task.id" class="py-4 flex items-start justify-between">
          <div class="flex items-start gap-3">
            <input type="checkbox" v-model="task.done" class="mt-1 h-4 w-4 text-emerald-600 rounded" />
            <div>
              <div :class="task.done ? 'line-through text-slate-400' : 'text-slate-800'" class="text-sm font-medium">
                {{ task.title }}
              </div>
              <div class="text-xs text-slate-400 mt-1">{{ task.note }}</div>
            </div>
          </div>
          <div class="text-xs text-slate-500 mt-0.5">{{ formatDate(task.date) }}</div>
        </li>
      </ul>

      <div class="px-5 py-4">
        <a href="#" @click.prevent="toggleShowMore" class="text-sm text-emerald-600 hover:underline">{{ showMore ? 'Show less' : 'Show more...' }}</a>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import SettingsIcon from '../../icons/SettingsIcon.vue' // correct path from src/components/ecommerce/

const tasks = ref([
  { id: 1, title: 'Review and make sure nothing slips through cracks', note: '', date: '2021-09-15', done: false },
  { id: 2, title: 'Send meeting invites for sales upcampaign', note: '', date: '2021-09-20', done: false },
  { id: 3, title: 'Weekly closed sales won checking with sales team', note: '', date: '2021-09-24', done: false },
  { id: 4, title: 'Add notes that can be viewed from the individual view', note: '', date: '2021-09-27', done: false },
  { id: 5, title: 'Follow up on onboarding emails', note: '', date: '2021-09-28', done: false },
  { id: 6, title: 'Prepare monthly report', note: '', date: '2021-09-30', done: true },
])

const showAdd = ref(false)
const showMore = ref(false)
const maxTasks = ref(10)
const newTaskTitle = ref('')
const newTaskDate = ref('')

const remaining = computed(() => tasks.value.filter(t => !t.done).length)

const visibleTasks = computed(() => {
  if (showMore.value) return tasks.value
  return tasks.value.slice(0, 4)
})

function toggleAdd() { showAdd.value = !showAdd.value }
function toggleShowMore() { showMore.value = !showMore.value }

function addTask() {
  if (!newTaskTitle.value.trim()) return
  if (tasks.value.length >= maxTasks.value) {
    alert('Maximum tasks reached')
    return
  }
  tasks.value.unshift({
    id: Date.now(),
    title: newTaskTitle.value.trim(),
    note: '',
    date: newTaskDate.value || new Date().toISOString().slice(0,10),
    done: false
  })
  newTaskTitle.value = ''
  newTaskDate.value = ''
  showAdd.value = false
}

function formatDate(d) {
  if (!d) return ''
  return d
}
</script>

<style scoped>
.fade-enter-active, .fade-leave-active { transition: opacity .15s ease }
.fade-enter-from, .fade-leave-to { opacity: 0 }
</style>
