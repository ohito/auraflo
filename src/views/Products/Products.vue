<template>
  <AdminLayout>
    <div class="p-6">
      <!-- HEADER -->
      <div class="mb-6">
        <div class="bg-white rounded-2xl border border-gray-200 p-4 shadow-sm flex flex-col md:flex-row items-center justify-between gap-4">
          <div class="flex items-center gap-4 min-w-0 w-full md:w-auto">
            <h2 class="text-2xl font-semibold whitespace-nowrap">Products</h2>

            <div class="relative flex-1 min-w-0 md:w-[560px]">
              <span class="absolute left-3 top-1/2 -translate-y-1/2 text-slate-400">
                <SearchIcon class="w-4 h-4" />
              </span>
              <input
                v-model="q"
                type="search"
                placeholder="Search products by name, sku, vendor..."
                class="w-full h-10 rounded-xl border border-gray-200 px-4 pl-10 text-sm placeholder:text-slate-400 focus:outline-none"
              />
            </div>
          </div>

          <div class="flex items-center gap-3 w-full md:w-auto">
            <select v-model="categoryFilter" class="h-9 rounded-xl border border-gray-200 bg-white px-3 text-sm">
              <option value="">All categories</option>
              <option v-for="c in uniqueCategories" :key="c" :value="c">{{ c }}</option>
            </select>

            <select v-model="sortBy" class="h-9 rounded-xl border border-gray-200 bg-white px-3 text-sm">
              <option value="name">Name</option>
              <option value="price_asc">Price ↑</option>
              <option value="price_desc">Price ↓</option>
              <option value="stock">Stock</option>
            </select>

            <button @click="openAdd" class="inline-flex items-center gap-2 rounded-lg bg-sky-500 px-4 py-2 text-white text-sm font-medium hover:bg-sky-600 shadow">+ Add Product</button>
          </div>
        </div>
      </div>

      <!-- BULK ACTIONS -->
      <div class="mb-4 flex items-center justify-between">
        <div class="flex items-center gap-3">
          <div class="text-sm text-slate-500">Bulk:</div>
          <button class="px-3 py-1 rounded border text-sm" :disabled="selected.size===0" @click="bulkExport">Export CSV</button>
          <button class="px-3 py-1 rounded border text-sm text-rose-600" :disabled="selected.size===0" @click="bulkDelete">Delete</button>
        </div>
        <div class="text-sm text-slate-500">Showing {{ filtered.length }} products</div>
      </div>

     <!-- PRODUCTS GRID -->
<div class="bg-white rounded-2xl border border-gray-200 p-4 shadow-sm">
  <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
    <div v-for="product in paged" :key="product.id" class="product-card bg-white border rounded-lg p-4 shadow-sm flex flex-col">
      <!-- top row: checkbox + image -->
      <div class="flex items-start gap-3">
        <div class="pt-1">
          <input type="checkbox" :checked="isSelected(product.id)" @click.stop="toggleSelect(product.id)" class="checkbox" />
        </div>

        <div class="w-20 h-20 rounded-md overflow-hidden flex-shrink-0 border bg-slate-50 flex items-center justify-center">
          <img :src="product.image" alt="" class="w-full h-full object-cover" />
        </div>

        <div class="flex-1 min-w-0">
          <div class="font-semibold truncate" :title="product.name">{{ product.name }}</div>
          <div class="text-xs text-slate-400 mt-1 truncate">{{ product.sku }}</div>
        </div>
      </div>

      <!-- middle: tags & spacer -->
      <div class="mt-3 flex-1">
        <div class="flex flex-wrap gap-2">
          <span v-for="t in product.tags" :key="t" class="text-xs px-2 py-1 rounded bg-slate-100 text-slate-600">{{ t }}</span>
        </div>

        <div class="mt-4 text-sm text-slate-700 font-semibold">
          {{ formatPrice(product.price) }}
        </div>
        <div class="text-xs text-slate-400">{{ product.stock }} in stock</div>
      </div>

      <!-- bottom: actions aligned to bottom -->
      <div class="mt-4 flex items-center gap-2 justify-between">
        <div class="flex gap-2">
          <button class="px-3 py-1 rounded border text-sm" @click.stop="openDetail(product)">View</button>
          <button class="px-3 py-1 rounded border text-sm" @click.stop="openEdit(product)">Edit</button>
        </div>
        <button class="px-3 py-1 rounded border text-sm text-rose-600" @click.stop="deleteProduct(product.id)">Delete</button>
      </div>
    </div>
  </div>

  <!-- pagination -->
  <div class="mt-4 flex items-center justify-between">
    <div class="text-sm text-slate-500">Page {{ page }} / {{ totalPages }}</div>
    <div class="flex items-center gap-2">
      <button @click="prevPage" :disabled="page===1" class="px-3 py-1 rounded border text-sm disabled:opacity-50">Prev</button>
      <button v-for="p in pages" :key="p" @click="page = p" :class="['px-3 py-1 rounded text-sm', page===p ? 'bg-slate-800 text-white' : 'border']">{{ p }}</button>
      <button @click="nextPage" :disabled="page===totalPages" class="px-3 py-1 rounded border text-sm disabled:opacity-50">Next</button>
    </div>
  </div>
</div>


      <!-- Add/Edit Modal -->
      <dialog ref="modal" class="rounded-lg p-0" @close="closeModal">
        <form method="dialog" class="w-[720px] max-w-full bg-white rounded-lg overflow-hidden shadow-lg">
          <div class="p-5 border-b flex items-center justify-between">
            <h3 class="text-lg font-semibold">{{ editMode ? 'Edit Product' : 'Add Product' }}</h3>
            <div class="text-sm text-slate-400">{{ form.id ? 'ID: ' + form.id : '' }}</div>
          </div>

          <div class="p-5 space-y-3">
            <div class="grid grid-cols-2 gap-3">
              <input v-model="form.name" placeholder="Product name" class="border rounded px-3 py-2" />
              <input v-model="form.sku" placeholder="SKU" class="border rounded px-3 py-2" />
              <input v-model.number="form.price" type="number" placeholder="Price (IDR)" class="border rounded px-3 py-2" />
              <input v-model.number="form.stock" type="number" placeholder="Stock" class="border rounded px-3 py-2" />
              <input v-model="form.vendor" placeholder="Vendor" class="border rounded px-3 py-2" />
              <input v-model="form.category" placeholder="Category" class="border rounded px-3 py-2" />
            </div>

            <div class="flex items-center gap-3">
              <input v-model="form.image" placeholder="Image URL" class="border rounded px-3 py-2 flex-1" />
              <input v-model="newTag" @keyup.enter.prevent="addTag" placeholder="Add tag & press Enter" class="border rounded px-3 py-2 w-48" />
            </div>

            <div class="flex flex-wrap gap-2">
              <span v-for="(t,i) in form.tags" :key="t" class="text-xs px-2 py-1 rounded bg-slate-100 flex items-center gap-2">
                {{ t }}
                <button class="text-rose-600 ml-1" @click.prevent="removeTag(i)">✕</button>
              </span>
            </div>

            <textarea v-model="form.description" placeholder="Description" class="w-full border rounded px-3 py-2"></textarea>
          </div>

          <div class="p-4 border-t flex justify-end gap-2">
            <button type="button" class="px-4 py-2 rounded border" @click="closeModal">Cancel</button>
            <button type="button" class="px-4 py-2 rounded bg-emerald-500 text-white" @click="saveProduct">{{ editMode ? 'Save' : 'Create' }}</button>
          </div>
        </form>
      </dialog>

      <!-- Product Detail slide-over -->
      <div v-if="showDetail" class="fixed inset-0 z-40">
        <div class="absolute inset-0 bg-black/30" @click="closeDetail"></div>
        <aside class="absolute right-6 top-20 w-[420px] bg-white rounded-xl shadow-2xl border p-5 overflow-auto max-h-[80vh]">
          <div class="flex items-start justify-between gap-3">
            <div class="flex items-start gap-3">
              <img :src="selectedProduct.image" class="w-16 h-16 object-cover rounded-md border" />
              <div>
                <div class="text-lg font-semibold">{{ selectedProduct.name }}</div>
                <div class="text-xs text-slate-500">{{ selectedProduct.sku }} • {{ selectedProduct.vendor }}</div>
                <div class="mt-2 text-sm font-semibold">{{ formatPrice(selectedProduct.price) }}</div>
              </div>
            </div>
            <button class="text-slate-500" @click="closeDetail">✕</button>
          </div>

          <div class="mt-4 text-sm text-slate-700">
            <div class="mb-3">
              <div class="font-semibold">Stock</div>
              <div class="mt-2">{{ selectedProduct.stock }} pcs</div>
            </div>

            <div class="mb-3">
              <div class="font-semibold">Category</div>
              <div class="mt-2 text-slate-500">{{ selectedProduct.category || '—' }}</div>
            </div>

            <div class="mb-3">
              <div class="font-semibold">Tags</div>
              <div class="mt-2 flex gap-2 flex-wrap">
                <span v-for="t in selectedProduct.tags" :key="t" class="text-xs px-2 py-1 rounded bg-slate-100">{{ t }}</span>
              </div>
            </div>

            <div class="mb-3">
              <div class="font-semibold">Description</div>
              <div class="mt-2 text-sm text-slate-600" v-html="selectedProduct.description || '<em>No description</em>'"></div>
            </div>

            <div class="mt-4 flex gap-3">
              <button class="px-3 py-2 rounded bg-sky-600 text-white" @click="openEdit(selectedProduct)">Edit</button>
              <button class="px-3 py-2 rounded border text-rose-600" @click="deleteProduct(selectedProduct.id)">Delete</button>
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

/* sample products */
const initialProducts = [
  { id:1, name:'Basic Tee - White', sku:'BTW-001', price:88000, stock:120, image:'https://i.pravatar.cc/200?img=10', tags:['Apparel','Top'], vendor:'Local Co', category:'Clothing', description:'Soft cotton tee.' },
  { id:2, name:'Running Shoes - Red', sku:'RS-RED-09', price:520000, stock:34, image:'https://i.pravatar.cc/200?img=11', tags:['Footwear'], vendor:'Sportify', category:'Shoes', description:'Lightweight running shoes.' },
  { id:3, name:'Wireless Earbuds', sku:'EB-202', price:349000, stock:54, image:'https://i.pravatar.cc/200?img=12', tags:['Electronics','Audio'], vendor:'Soundry', category:'Electronics', description:'Bluetooth earbuds with mic.' },
  { id:4, name:'Coffee Mug - Blue', sku:'CM-BL-01', price:45000, stock:200, image:'https://i.pravatar.cc/200?img=13', tags:['Home'], vendor:'HomeGoods', category:'Kitchen', description:'Ceramic mug 350ml.' },
  { id:5, name:'Backpack 20L', sku:'BP-20', price:210000, stock:76, image:'https://i.pravatar.cc/200?img=14', tags:['Bags'], vendor:'CarryCo', category:'Accessories', description:'Everyday 20L backpack.' },
  { id:6, name:'Denim Jacket', sku:'DJ-01', price:420000, stock:12, image:'https://i.pravatar.cc/200?img=15', tags:['Apparel'], vendor:'DenimLab', category:'Clothing', description:'Classic denim jacket.' },
]

const products = ref(JSON.parse(JSON.stringify(initialProducts)))

/* UI state */
const q = ref('')
const categoryFilter = ref('')
const sortBy = ref('name')
const page = ref(1)
const perPage = ref(8)
const selected = ref(new Set())

/* modal form */
const modal = ref(null)
const editMode = ref(false)
const form = reactive({
  id: null, name:'', sku:'', price:0, stock:0, image:'', tags:[], vendor:'', category:'', description:''
})
const newTag = ref('')

/* detail */
const showDetail = ref(false)
const selectedProduct = reactive({})

/* computed lists */
const filtered = computed(() => {
  const s = q.value.trim().toLowerCase()
  return products.value
    .filter(p => {
      if (categoryFilter.value && p.category !== categoryFilter.value) return false
      if (!s) return true
      return (p.name && p.name.toLowerCase().includes(s)) || (p.sku && p.sku.toLowerCase().includes(s)) || (p.vendor && p.vendor.toLowerCase().includes(s))
    })
    .sort((a,b) => {
      if (sortBy.value === 'name') return a.name.localeCompare(b.name)
      if (sortBy.value === 'price_asc') return a.price - b.price
      if (sortBy.value === 'price_desc') return b.price - a.price
      if (sortBy.value === 'stock') return b.stock - a.stock
      return 0
    })
})

const totalPages = computed(() => Math.max(1, Math.ceil(filtered.value.length / perPage.value)))
const pages = computed(() => {
  const arr = []
  for (let i=1;i<=totalPages.value;i++) arr.push(i)
  return arr
})
const paged = computed(() => {
  const start = (page.value - 1) * perPage.value
  return filtered.value.slice(start, start + perPage.value)
})

/* helpers */
function formatPrice(v=0) {
  if (!v) return 'Rp 0'
  if (v >= 1000) {
    const k = Math.round(v/100)/10
    return `Rp ${k}k`
  }
  return `Rp ${v}`
}

/* selection */
const allSelected = computed(() => selected.value.size > 0 && selected.value.size === filtered.value.length)
function toggleSelectAll(e) {
  if (e.target.checked) selected.value = new Set(filtered.value.map(p => p.id))
  else selected.value = new Set()
}
function toggleSelect(id) {
  if (selected.value.has(id)) selected.value.delete(id)
  else selected.value.add(id)
  selected.value = new Set(selected.value)
}
function isSelected(id) { return selected.value.has(id) }

/* pagination */
function prevPage() { if (page.value>1) page.value-- }
function nextPage() { if (page.value < totalPages.value) page.value++ }

/* unique categories */
const uniqueCategories = computed(() => {
  const set = new Set(products.value.map(p => p.category).filter(Boolean))
  return Array.from(set)
})

/* add/remove tags */
function addTag() {
  const t = newTag.value && newTag.value.trim()
  if (!t) return
  if (!form.tags.includes(t)) form.tags.push(t)
  newTag.value = ''
}
function removeTag(i) {
  form.tags.splice(i,1)
}

/* create / edit */
function openAdd() {
  editMode.value = false
  form.id = null
  form.name = ''
  form.sku = ''
  form.price = 0
  form.stock = 0
  form.image = `https://i.pravatar.cc/200?img=${Math.floor(Math.random()*40)+10}`
  form.tags = []
  form.vendor = ''
  form.category = ''
  form.description = ''
  modal.value?.showModal?.()
}
function openEdit(p) {
  editMode.value = true
  form.id = p.id
  form.name = p.name
  form.sku = p.sku
  form.price = p.price
  form.stock = p.stock
  form.image = p.image
  form.tags = Array.isArray(p.tags) ? [...p.tags] : []
  form.vendor = p.vendor
  form.category = p.category
  form.description = p.description
  modal.value?.showModal?.()
}
function closeModal() { modal.value?.close?.() }

function saveProduct() {
  if (!form.name || !form.sku) {
    alert('Name & SKU required')
    return
  }
  if (editMode.value && form.id) {
    const idx = products.value.findIndex(x => x.id === form.id)
    if (idx !== -1) {
      products.value[idx] = {
        id: form.id,
        name: form.name,
        sku: form.sku,
        price: Number(form.price)||0,
        stock: Number(form.stock)||0,
        image: form.image,
        tags: [...form.tags],
        vendor: form.vendor,
        category: form.category,
        description: form.description
      }
    }
  } else {
    const id = Math.max(0, ...products.value.map(p => p.id)) + 1
    products.value.unshift({
      id,
      name: form.name,
      sku: form.sku,
      price: Number(form.price)||0,
      stock: Number(form.stock)||0,
      image: form.image,
      tags: [...form.tags],
      vendor: form.vendor,
      category: form.category,
      description: form.description
    })
  }
  rebuildPagination()
  closeModal()
}

/* detail slide-over */
function openDetail(p) {
  Object.assign(selectedProduct, p)
  showDetail.value = true
}
function closeDetail() {
  showDetail.value = false
  Object.keys(selectedProduct).forEach(k => delete selectedProduct[k])
}

/* delete */
function deleteProduct(id) {
  if (!confirm('Delete this product?')) return
  products.value = products.value.filter(p => p.id !== id)
  selected.value.delete(id)
  rebuildPagination()
}

/* bulk */
function bulkDelete() {
  if (!confirm('Delete selected products?')) return
  const ids = new Set(Array.from(selected.value))
  products.value = products.value.filter(p => !ids.has(p.id))
  selected.value = new Set()
  rebuildPagination()
}
function bulkExport() {
  // simple CSV export of selected product rows
  const ids = new Set(Array.from(selected.value))
  const rows = products.value.filter(p => ids.has(p.id))
  if (rows.length === 0) {
    alert('No products selected')
    return
  }
  const header = ['id','name','sku','price','stock','vendor','category','tags']
  const csv = [header.join(',')]
  for (const r of rows) {
    csv.push([r.id, `"${r.name.replace(/"/g,'""')}"`, r.sku, r.price, r.stock, `"${(r.vendor||'').replace(/"/g,'""')}"`, `"${(r.category||'').replace(/"/g,'""')}"`, `"${(r.tags||[]).join(';')}"`].join(','))
  }
  const blob = new Blob([csv.join('\n')], { type: 'text/csv' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = 'products_export.csv'
  a.click()
  URL.revokeObjectURL(url)
}

/* misc */
function rebuildPagination() {
  if (page.value > totalPages.value) page.value = totalPages.value
}

/* lifecycle */
onMounted(() => {
  rebuildPagination()
})
</script>

<style scoped>
/* card hover & equal height helpers */
.product-card { transition: transform .12s ease, box-shadow .12s ease; min-height: 240px; } /* min-height menjaga konsistensi */
.product-card:hover { transform: translateY(-6px); box-shadow: 0 10px 30px rgba(2,6,23,0.08); }

/* image inside block */
.product-card img { display:block; }

/* checkbox small tweak */
.checkbox { width:18px; height:18px; }

/* header area: prevent wrapping overflow */
.header-card { display:flex; gap:1rem; align-items:center; justify-content:space-between; flex-wrap:wrap; }

/* ensure cards align bottom for actions */
.product-card > .mt-4.flex.items-center { margin-top: auto; }

/* responsive tweaks (if needed) */
@media (max-width: 900px) {
  .grid-cols-4 { grid-template-columns: repeat(1, minmax(0,1fr)); }
  .product-card { min-height: unset; }
}

</style>
