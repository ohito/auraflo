<template>
  <div
    class="overflow-hidden rounded-2xl border border-gray-200 bg-white px-6 py-5 dark:border-gray-800 dark:bg-white/[0.03] w-full"
  >
    <!-- Header stats -->
    <div class="flex flex-col md:flex-row items-start md:items-center justify-between gap-4">
      <h3 class="text-base font-semibold text-gray-800 dark:text-white/90">Balance Overview</h3>

      <div class="flex-1"></div>

      <div class="flex items-center gap-6 text-sm text-gray-600">
        <div class="flex items-baseline gap-2">
          <span class="text-2xl font-extrabold text-[#2fc594]">$584k</span>
          <span class="text-sm text-gray-500">Revenue</span>
        </div>
        <div class="h-6 border-l border-gray-100"></div>
        <div class="flex items-baseline gap-2">
          <span class="text-2xl font-extrabold text-[#f28b7a]">$497k</span>
          <span class="text-sm text-gray-500">Expenses</span>
        </div>
        <div class="h-6 border-l border-gray-100"></div>
        <div class="flex items-baseline gap-2">
          <span class="text-2xl font-extrabold text-gray-800">3.6%</span>
          <span class="text-sm text-gray-500">Profit Ratio</span>
        </div>
      </div>
    </div>

    <!-- Chart area -->
    <div class="mt-4">
      <div class="w-full overflow-visible">
        <VueApexCharts
          ref="chartRef"
          type="area"
          height="260"
          :options="chartOptions"
          :series="series"
        />
      </div>

      <!-- custom legend -->
      <div class="mt-3 flex gap-4 items-center justify-start text-sm text-gray-600">
        <div class="flex items-center gap-2"><span class="w-3 h-3 rounded-full" :style="{backgroundColor: colors[0]}"></span><span>Revenue</span></div>
        <div class="flex items-center gap-2"><span class="w-3 h-3 rounded-full" :style="{backgroundColor: colors[1]}"></span><span>Expenses</span></div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick } from 'vue'
import VueApexCharts from 'vue3-apexcharts'

// palette
const colors = ['#2fc594', '#f28b7a'] // revenue, expenses

// sample monthly data (Jun..Dec shown in image; we'll include Jan..Dec)
const series = ref([
  {
    name: 'Revenue',
    data: [48, 55, 70, 95, 140, 160, 180, 200, 220, 245, 270, 300].map(v => v * 1000), // in dollars
  },
  {
    name: 'Expenses',
    data: [42, 50, 60, 80, 110, 130, 150, 165, 185, 200, 220, 240].map(v => v * 1000),
  },
])

const chartOptions = ref({
  chart: {
    type: 'area',
    toolbar: { show: false },
    zoom: { enabled: false },
    animations: { enabled: false },
    fontFamily: 'Outfit, sans-serif',
  },
  colors,
  dataLabels: {
    enabled: false
  },

  stroke: {
    width: [3, 3],
    curve: 'smooth',
  },
  fill: {
    type: 'gradient',
    gradient: {
      shadeIntensity: 1,
      inverseColors: false,
      opacityFrom: 0.18,
      opacityTo: 0.03,
      stops: [0, 90, 100],
    },
  },
  markers: { size: 3, hover: { size: 5 }, strokeWidth: 0 },
  xaxis: {
    categories: ['Jan','Feb','Mar','Apr','May','Jun','Jul','Aug','Sep','Oct','Nov','Dec'],
    labels: { style: { colors: '#9ca3af', fontSize: '12px' } },
    axisBorder: { show: false },
    axisTicks: { show: false },
  },
  yaxis: {
    labels: {
      formatter: (val) => `$${Math.round(val/1000)}k`,
      style: { colors: '#9ca3af', fontSize: '12px' },
    },
    tickAmount: 4,
  },
  grid: {
    borderColor: '#eaeef3',
    strokeDashArray: 0,
    xaxis: { lines: { show: false } },
    yaxis: { lines: { show: true } },
  },
  tooltip: {
    theme: 'light',
    x: { show: true },
    y: {
      formatter: (val) => `$${(val/1000).toLocaleString()}k`,
    },
  },
  legend: { show: false },
  responsive: [
    { breakpoint: 640, options: { chart: { height: 220 } } }
  ],
})

// reflow fix & ref
const chartRef = ref(null)
onMounted(async () => {
  await nextTick()
  setTimeout(() => {
    window.dispatchEvent(new Event('resize'))
    // safe update if available
    try {
      chartRef.value?.chart?.updateOptions({}, true, false, true)
    } catch (e) { /* ignore */ }
  }, 150)
})
</script>

<style scoped>
/* allow apex internals to be visible */
:deep(.apexcharts-root),
:deep(.apexcharts-canvas) {
  overflow: visible !important;
}

/* ensure gradient/fill looks subtle & not overpowering */
:deep(.apexcharts-area) {
  fill-opacity: 0.25 !important;
}

/* grid line color (subtle) */
:deep(.apexcharts-gridlines-horizontal line) {
  stroke: rgba(14,22,37,0.06) !important;
}

/* axis label color */
:deep(.apexcharts-xaxis text),
:deep(.apexcharts-yaxis text) {
  fill: #6b7280 !important;
}

/* ensure the series use our exact colors (protect from overrides) */
:deep(.apexcharts-series:nth-of-type(1) path) {
  stroke: #2fc594 !important;
  fill: url(#apexcharts-gradient-0) !important;
}
:deep(.apexcharts-series:nth-of-type(2) path) {
  stroke: #f28b7a !important;
  fill: url(#apexcharts-gradient-1) !important;
}
</style>
