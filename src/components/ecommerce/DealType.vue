<template>
  <div
    class="overflow-hidden rounded-2xl border border-gray-200 bg-white px-5 pt-5 dark:border-gray-800 dark:bg-white/[0.03] sm:px-6 sm:pt-6 w-full max-w-[360px]"
  >
    <!-- Header -->
    <div class="flex items-center justify-between border-b border-gray-100 pb-3 mb-4">
      <h3 class="text-base font-semibold text-gray-800 dark:text-white/90">Deal Type</h3>

      <div class="flex items-center gap-2 text-sm text-gray-500">
        <span class="font-semibold">SORT BY:</span>
        <button class="flex items-center gap-1 font-medium text-gray-700 dark:text-gray-300 hover:text-gray-900">
          Monthly
          <svg class="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"
            stroke-linejoin="round">
            <polyline points="6 9 12 15 18 9" />
          </svg>
        </button>
      </div>
    </div>

    <!-- Radar Chart area -->
    <div class="flex flex-col items-center">
      <div class="w-full max-w-[300px] mx-auto">
        <VueApexCharts ref="radarRef" type="radar" height="260" :options="chartOptions" :series="series" />
      </div>

      <!-- Legend (custom; Apex may also render one — we hide apex legend and use our own) -->
      <div class="mt-4 flex gap-6 justify-center text-xs">
        <div class="flex items-center gap-2 text-gray-600">
          <span class="w-3 h-3 rounded-full" :style="{ backgroundColor: colors[0] }"></span>
          Pending
        </div>
        <div class="flex items-center gap-2 text-gray-600">
          <span class="w-3 h-3 rounded-full" :style="{ backgroundColor: colors[1] }"></span>
          Loss
        </div>
        <div class="flex items-center gap-2 text-gray-600">
          <span class="w-3 h-3 rounded-full" :style="{ backgroundColor: colors[2] }"></span>
          Won
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick } from 'vue'
import VueApexCharts from 'vue3-apexcharts'

// colors follow the dashboard palette (yellow, red, green)
const colors = ['#f4c24b', '#f28b7a', '#2fc594']

// sample series data (replace with API values if you have)
const series = ref([
  {
    name: 'Pending',
    data: [20, 40, 60, 30, 50, 40],
  },
  {
    name: 'Loss',
    data: [10, 30, 50, 70, 30, 55],
  },
  {
    name: 'Won',
    data: [5, 20, 30, 40, 60, 45],
  },
])

const chartOptions = ref({
  chart: {
    toolbar: { show: false },
    animations: { enabled: true },
    fontFamily: 'Outfit, sans-serif',
  },
  colors,
  stroke: {
    width: 2,
  },
  fill: {
    opacity: 0.25,
  },
  markers: {
    size: 4,
    strokeWidth: 2,
    hover: { size: 6 },
  },
  labels: ['2016', '2017', '2018', '2019', '2020', '2021'],
  yaxis: {
    show: false,
  },
  xaxis: {
    labels: {
      style: { colors: '#6b7280' }, // subtle year labels color
    },
  },
  legend: {
    show: false, // we use custom legend under chart
  },
  tooltip: {
    y: {
      formatter: (val) => `${val}`,
    },
  },
  responsive: [
    {
      breakpoint: 640,
      options: {
        chart: { height: 220 },
        markers: { size: 3 },
      },
    },
  ],
})

// small reflow fix so apex positions things accurately on initial render
const radarRef = ref(null)
onMounted(async () => {
  await nextTick()
  setTimeout(() => {
    window.dispatchEvent(new Event('resize'))
    if (radarRef.value?.chart) {
      // force redraw if needed
      // apex internals: updateSeries can trigger re-layout — we don't change data here, but safe to call updateOptions empty
      try {
        radarRef.value.chart.updateOptions({}, true, false, true)
      } catch (e) {
        // ignore if not available
      }
    }
  }, 120)
})
</script>

<style scoped>
/* ensure apex internals not clipped by parent */
:deep(.apexcharts-root),
:deep(.apexcharts-canvas) {
  overflow: visible !important;
}

/* subtle grid lines color like example */
:deep(.apexcharts-gridlines-horizontal line) {
  stroke: rgba(14, 22, 37, 0.04) !important;
}

/* labels color */
:deep(.apexcharts-radar .apexcharts-datalabel text),
:deep(.apexcharts-legends text) {
  fill: #374151 !important; /* gray-700 */
}

/* keep stroke opacity consistent */
:deep(.apexcharts-area) {
  fill-opacity: 0.25 !important;
}
</style>
