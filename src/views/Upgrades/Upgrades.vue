<template>
  <AdminLayout>
    <div class="p-6">
      <!-- HEADER -->
      <div class="mb-6">
        <div class="bg-white rounded-2xl border border-gray-200 p-6 shadow-sm flex flex-col md:flex-row items-start md:items-center justify-between gap-4">
          <div>
            <h1 class="text-2xl font-bold">Upgrade Plan</h1>
            <div class="text-sm text-slate-500 mt-1">Choose a plan that fits your team and scale — billing handled securely.</div>
          </div>

          <div class="flex items-center gap-3">
            <div class="text-sm text-slate-500">Billing:</div>
            <div class="inline-flex items-center bg-slate-50 rounded-full p-1">
              <button :class="['px-3 py-1 rounded-full text-sm', billingCycle==='monthly' ? 'bg-white shadow' : 'text-slate-500']" @click="billingCycle='monthly'">Monthly</button>
              <button :class="['px-3 py-1 rounded-full text-sm', billingCycle==='yearly' ? 'bg-white shadow' : 'text-slate-500']" @click="billingCycle='yearly'">Yearly (save 20%)</button>
            </div>
          </div>
        </div>
      </div>

      <!-- Current Plan -->
      <div class="mb-6">
        <div class="bg-white rounded-2xl border border-emerald-100 p-4 shadow-sm flex items-center justify-between">
          <div>
            <div class="text-xs uppercase text-emerald-600 font-semibold">Current Plan</div>
            <div class="text-lg font-semibold">Pro — Team</div>
            <div class="text-sm text-slate-500 mt-1">Next billing date: {{ nextBillingDate }}</div>
          </div>

          <div class="text-right">
            <div class="text-xl font-bold">{{ displayPrice(currentPlanKey) }}</div>
            <div class="text-sm text-slate-400">per {{ billingCycle }}</div>
          </div>
        </div>
      </div>

      <!-- Pricing Tiers -->
      <div class="mb-6">
        <div class="grid grid-cols-1 sm:grid-cols-3 gap-6">
          <div v-for="p in plans" :key="p.key" :class="['rounded-2xl border p-5 relative', p.key === currentPlanKey ? 'border-emerald-200 shadow-lg' : 'border-gray-200']">
            <div class="flex items-center justify-between">
              <div>
                <div class="text-sm font-semibold">{{ p.name }}</div>
                <div class="text-xs text-slate-400 mt-1">{{ p.subtitle }}</div>
              </div>
              <div class="text-right">
                <div class="text-2xl font-extrabold">{{ displayPrice(p.key) }}</div>
                <div class="text-xs text-slate-400">per {{ billingCycle }}</div>
              </div>
            </div>

            <ul class="mt-4 space-y-2 text-sm text-slate-600">
              <li v-for="f in p.features" :key="f" class="flex items-center gap-2">
                <svg class="w-4 h-4 text-emerald-500" viewBox="0 0 24 24" fill="none"><path d="M5 13l4 4L19 7" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>
                <span>{{ f }}</span>
              </li>
            </ul>

            <div class="mt-6 flex items-center justify-between">
              <button
                v-if="p.key === currentPlanKey"
                class="px-4 py-2 rounded-lg bg-slate-100 text-slate-700 text-sm font-semibold"
                disabled
              >Current</button>

              <button
                v-else
                class="px-4 py-2 rounded-lg bg-emerald-500 text-white text-sm font-semibold hover:bg-emerald-600"
                @click="openUpgrade(p.key)"
              >Upgrade</button>

              <div class="text-xs text-slate-400">{{ p.userLimit }}</div>
            </div>

            <div v-if="p.badge" class="absolute -top-3 left-4 bg-indigo-600 text-white px-3 py-1 rounded-full text-xs font-semibold shadow">
              {{ p.badge }}
            </div>
          </div>
        </div>
      </div>

      <!-- Plan Comparison -->
      <div class="mb-6 bg-white rounded-2xl border border-gray-200 p-5 shadow-sm">
        <h3 class="font-semibold mb-3">Plan Comparison</h3>
        <div class="overflow-auto">
          <table class="w-full table-auto text-sm">
            <thead>
              <tr class="text-left text-xs text-slate-500">
                <th class="p-2">Feature</th>
                <th v-for="p in plans" :key="p.key" class="p-2 text-center">{{ p.name }}</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="row in comparison" :key="row.feature" class="border-t">
                <td class="p-3 font-medium">{{ row.feature }}</td>
                <td v-for="p in plans" :key="p.key" class="p-3 text-center">
                  <span v-if="row.available.includes(p.key)" class="text-emerald-600 font-semibold">Yes</span>
                  <span v-else class="text-slate-400">—</span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <!-- Billing / Payment method -->
      <div class="mb-6 grid grid-cols-1 md:grid-cols-2 gap-6">
        <div class="bg-white rounded-2xl border border-gray-200 p-4 shadow-sm">
          <h4 class="font-semibold mb-2">Billing Method</h4>
          <div class="text-sm text-slate-500">Default payment method</div>

          <div class="mt-4 flex items-center gap-3">
            <div class="flex-1">
              <div class="text-sm font-semibold">Visa •••• 4242</div>
              <div class="text-xs text-slate-400">Expires 08/26</div>
            </div>
            <div class="flex gap-2">
              <button class="px-3 py-1 rounded border text-sm" @click="openPayment">Change</button>
              <button class="px-3 py-1 rounded bg-sky-600 text-white text-sm" @click="openPayment">Add payment</button>
            </div>
          </div>
        </div>

        <div class="bg-white rounded-2xl border border-gray-200 p-4 shadow-sm">
          <h4 class="font-semibold mb-2">Subscription</h4>
          <div class="text-sm text-slate-500">Manage team seats & billing</div>

          <div class="mt-4 flex items-center gap-3">
            <div class="flex-1">
              <div class="text-sm">Seats used: <span class="font-semibold">{{ usedSeats }}/{{ plans.find(p=>p.key===currentPlanKey)?.userLimit || '—' }}</span></div>
              <div class="text-xs text-slate-400 mt-1">Upgrade to add more seats</div>
            </div>
            <button class="px-3 py-1 rounded bg-emerald-500 text-white text-sm" @click="openUpgrade()">Manage seats</button>
          </div>
        </div>
      </div>

      <!-- FAQ -->
      <div class="bg-white rounded-2xl border border-gray-200 p-5 shadow-sm">
        <h4 class="font-semibold mb-3">FAQ</h4>
        <div v-for="f in faqs" :key="f.q" class="mb-3">
          <div class="font-medium">{{ f.q }}</div>
          <div class="text-sm text-slate-500 mt-1">{{ f.a }}</div>
        </div>
      </div>
    </div>

    <!-- UPGRADE / CHECKOUT MODAL -->
    <dialog ref="upgradeModal" class="rounded-lg p-0" @close="closeUpgradeModal">
      <form method="dialog" class="w-[540px] max-w-full bg-white rounded-lg overflow-hidden shadow-lg">
        <div class="p-5 border-b">
          <h3 class="text-lg font-semibold">Confirm Upgrade</h3>
        </div>

        <div class="p-5">
          <div class="text-sm text-slate-700 mb-3">You're upgrading to <span class="font-semibold">{{ planToBuy?.name }}</span> ({{ billingCycle }})</div>
          <div class="flex items-center justify-between bg-slate-50 p-3 rounded">
            <div>
              <div class="text-sm text-slate-500">Amount</div>
              <div class="text-lg font-bold">{{ displayPrice(planToBuy?.key) }}</div>
            </div>
            <div>
              <div class="text-sm text-slate-500">Billing cycle</div>
              <div class="text-sm">{{ billingCycle }}</div>
            </div>
          </div>

          <div class="mt-4">
            <label class="text-sm font-medium">Payment method</label>
            <div class="mt-2 flex items-center gap-3">
              <div class="flex-1">
                <div class="text-sm font-semibold">Visa •••• 4242</div>
                <div class="text-xs text-slate-400">Expires 08/26</div>
              </div>
              <button class="px-3 py-1 rounded border text-sm" type="button" @click="openPayment">Change</button>
            </div>
          </div>
        </div>

        <div class="p-4 border-t flex justify-end gap-2">
          <button type="button" class="px-4 py-2 rounded border" @click="closeUpgradeModal">Cancel</button>
          <button type="button" class="px-4 py-2 rounded bg-emerald-500 text-white" @click="confirmUpgrade">Confirm Payment</button>
        </div>
      </form>
    </dialog>

    <!-- PAYMENT MODAL (demo) -->
    <dialog ref="paymentModal" class="rounded-lg p-0" @close="closePayment">
      <form method="dialog" class="w-[520px] max-w-full bg-white rounded-lg overflow-hidden shadow-lg">
        <div class="p-5 border-b">
          <h3 class="text-lg font-semibold">Payment Methods</h3>
        </div>

        <div class="p-5 space-y-3">
          <div class="text-sm text-slate-600">Demo payment modal — integrate real gateway (Stripe/ Xendit / Midtrans) on backend.</div>
          <div class="grid grid-cols-1 gap-3">
            <button class="px-4 py-2 rounded border text-left" @click.prevent="selectDemoMethod('visa')">Visa •••• 4242</button>
            <button class="px-4 py-2 rounded border text-left" @click.prevent="selectDemoMethod('paypal')">PayPal (demo)</button>
            <button class="px-4 py-2 rounded border text-left" @click.prevent="selectDemoMethod('manual')">Manual invoice</button>
          </div>
        </div>

        <div class="p-4 border-t flex justify-end gap-2">
          <button type="button" class="px-4 py-2 rounded bg-slate-100" @click="closePayment">Close</button>
        </div>
      </form>
    </dialog>
  </AdminLayout>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import AdminLayout from '@/components/layout/AdminLayout.vue'

/* demo plans */
const plans = [
  { key:'free', name:'Free', subtitle:'For personal use', priceMonthly:0, priceYearly:0, features:['Up to 1 project','Basic support','Community access'], userLimit:'1 seat', badge:'' },
  { key:'pro', name:'Pro', subtitle:'For small teams', priceMonthly:29, priceYearly:290, features:['Unlimited projects','Priority support','Integrations'], userLimit:'10 seats', badge:'Popular' },
  { key:'business', name:'Business', subtitle:'For growing teams', priceMonthly:99, priceYearly:990, features:['SAML SSO','Dedicated support','Advanced analytics'], userLimit:'Unlimited seats', badge:'Enterprise' },
]

/* comparison rows */
const comparison = [
  { feature: 'Projects', available: ['free','pro','business'] },
  { feature: 'Priority Support', available: ['pro','business'] },
  { feature: 'Integrations', available: ['pro','business'] },
  { feature: 'SAML SSO', available: ['business'] },
  { feature: 'Advanced analytics', available: ['business'] },
]

/* state */
const billingCycle = ref('monthly') // 'monthly' or 'yearly'
const currentPlanKey = ref('pro') // demo current
const nextBillingDate = ref(new Date(Date.now() + 14*24*60*60*1000).toISOString().slice(0,10))
const usedSeats = ref(4)

/* modal refs */
const upgradeModal = ref(null)
const paymentModal = ref(null)
const planToBuy = ref(null)

/* faqs */
const faqs = [
  { q:'Can I cancel anytime?', a:'Yes — cancel at any time from billing settings. No hidden fees.' },
  { q:'Do you offer discounts for yearly billing?', a:'Yes — yearly saves ~20% compared to monthly.' },
  { q:'Is there an enterprise plan?', a:'Yes — contact sales for custom pricing and SLAs.' },
]

/* helpers */
function findPlan(key) { return plans.find(p=>p.key===key) }
function displayPrice(key) {
  if (!key) return '—'
  const p = findPlan(key)
  if (!p) return '—'
  const val = billingCycle.value === 'monthly' ? p.priceMonthly : p.priceYearly
  if (val === 0) return 'Free'
  return `$${val}${billingCycle.value === 'monthly' ? '/mo' : '/yr'}`
}

/* upgrade flow */
function openUpgrade(key) {
  planToBuy.value = findPlan(key)
  upgradeModal.value?.showModal?.()
}
function closeUpgradeModal() { upgradeModal.value?.close?.() }
function confirmUpgrade() {
  // demo: simply set as current plan and close; integrate real payment flow
  if (!planToBuy.value) return
  currentPlanKey.value = planToBuy.value.key
  // mimic server update + next billing date
  nextBillingDate.value = new Date(Date.now() + (billingCycle.value === 'monthly' ? 30 : 365)*24*60*60*1000).toISOString().slice(0,10)
  closeUpgradeModal()
  alert(`Upgraded to ${planToBuy.value.name} (${billingCycle.value}) — demo only`)
}

/* payment modal */
function openPayment() { paymentModal.value?.showModal?.() }
function closePayment() { paymentModal.value?.close?.() }
function selectDemoMethod(method) {
  alert(`Selected payment method: ${method} (demo)`)
  closePayment()
}

/* computed planToBuy name */
const currentPlan = computed(()=> findPlan(currentPlanKey.value))

onMounted(()=> {
  // nothing special; modals are refs to <dialog>
})
</script>

<style scoped>
/* basic look */
.bg-white { background: #fff; }

/* billing toggle styles */
button[disabled] { opacity: 0.6; cursor: not-allowed; }

/* dialog backdrop */
dialog::backdrop { background: rgba(0,0,0,0.3); }

/* responsiveness */
@media (max-width: 768px) {
  .grid-cols-1.sm\:grid-cols-3 { grid-template-columns: repeat(1, minmax(0,1fr)); }
}
</style>
