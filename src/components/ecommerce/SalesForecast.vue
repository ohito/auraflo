<template>
  <div
    class="overflow-hidden rounded-2xl border border-gray-200 bg-white px-5 pt-5 dark:border-gray-800 dark:bg-white/[0.03] sm:px-6 sm:pt-6 w-full max-w-[360px]"
  >
    <!-- Header -->
    <div class="flex items-center justify-between border-b border-gray-100 pb-3 mb-4">
      <h3 class="text-base font-semibold text-gray-800 dark:text-white/90">
        Sales Forecast
      </h3>

      <div class="flex items-center gap-2 text-sm text-gray-500">
        <span class="font-semibold">SORT BY:</span>
        <button
          class="flex items-center gap-1 font-medium text-gray-700 dark:text-gray-300 hover:text-gray-900"
        >
          Nov 2021
          <svg
            class="w-4 h-4"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
          >
            <polyline points="6 9 12 15 18 9" />
          </svg>
        </button>
      </div>
    </div>

    <!-- Chart -->
    <div>
      <div class="mx-auto w-[300px] overflow-visible">
        <VueApexCharts
          ref="chartRef"
          type="bar"
          height="250"
          :options="chartOptions"
          :series="series"
        />
      </div>

      <!-- Label bawah chart -->
      <div class="text-center mt-2 text-sm text-gray-500">
        Total Forecasted Value
      </div>

      <!-- Legend -->
      <div class="mt-4 flex justify-center gap-6 text-xs">
        <div class="flex items-center gap-2 text-gray-600">
          <span class="w-3 h-3 rounded-sm" :style="{ backgroundColor: colors[0] }"></span>
          Goal
        </div>
        <div class="flex items-center gap-2 text-gray-600">
          <span class="w-3 h-3 rounded-sm" :style="{ backgroundColor: colors[1] }"></span>
          Pending Forecast
        </div>
        <div class="flex items-center gap-2 text-gray-600">
          <span class="w-3 h-3 rounded-sm" :style="{ backgroundColor: colors[2] }"></span>
          Revenue
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick } from 'vue'
import VueApexCharts from 'vue3-apexcharts'

const colors = ['#465fff', '#2fc594', '#f4c24b']

const series = ref([
  { name: 'Goal', data: [37000] },
  { name: 'Pending Forecast', data: [12000] },
  { name: 'Revenue', data: [18000] },
])

const chartOptions = ref({
  chart: {
    type: 'bar',
    toolbar: { show: false },
    animations: { enabled: false },
    fontFamily: 'Outfit, sans-serif',
  },
  colors,
  plotOptions: {
    bar: {
      horizontal: false,
      columnWidth: '28%',   // lebih kecil biar ada ruang
      borderRadius: 0,      // kotak
      endingShape: 'flat',
      barGap: 0.25,         // antar bar jarak 25% dari bar width (v4+)
      barSpacing: 6,        // jaga-jaga (untuk versi lama)
      dataLabels: {
        position: 'center',
      },
    },
  },


  dataLabels: {
    enabled: true,
    enabledOnSeries: [0, 1, 2],
    offsetY: 0,
    formatter: (val) => Math.round(val / 1000).toString(),
    style: {
      colors: ['#ffffff'],
      fontSize: '14px',
      fontWeight: '700',
    },
    background: { enabled: false },
  },

  stroke: { show: false },

  xaxis: {
    categories: [' '],
    labels: { show: false },
    axisBorder: { show: false },
    axisTicks: { show: false },
  },

  yaxis: {
    min: 0,
    max: 40000,
    tickAmount: 4,
    labels: {
      formatter: (v) => `$${Math.round(v / 1000)}k`,
      style: { colors: '#9ca3af', fontSize: '12px' },
    },
  },

  grid: {
    borderColor: '#eaeef3',
    strokeDashArray: 0,
    xaxis: { lines: { show: false } },
    yaxis: { lines: { show: true } },
    padding: {
      left: 10,
      right: 10,
      top: 0,
      bottom: 0,
    },
  },

  fill: { opacity: 1 },

  tooltip: {
    y: { formatter: (v) => `$${(v / 1000).toFixed(1)}k` },
  },

  legend: { show: false },

  responsive: [
    {
      breakpoint: 640,
      options: { plotOptions: { bar: { columnWidth: '40%' } }, chart: { height: 200 } },
    },
  ],
})

// ref to chart
const chartRef = ref(null)

onMounted(async () => {
  await nextTick()
  // small delay then trigger resize to force apex to recalc positions (fixes initial clipping/placement)
  setTimeout(() => {
    window.dispatchEvent(new Event('resize'))
    // if needed, call updateOptions:
    // if (chartRef.value?.chart) chartRef.value.chart.updateOptions({}, true, false, true)
  }, 120)
})
</script>


<style scoped>
/* Allow apex labels to show even when parent styles might hide overflow */
.apexcharts-root,
.apexcharts-canvas {
  overflow: visible !important;
}

/* If you want the yellow bar label darker for contrast, uncomment below:
.apexcharts-series:nth-of-type(3) .apexcharts-data-label text {
  fill: #222 !important;
}
*/

:deep(.apexcharts-bar-area .apexcharts-series path) {
  fill-opacity: 1 !important;
  stroke-width: 0 !important;
}

:deep(.apexcharts-data-label text) {
  text-anchor: middle !important;
  dominant-baseline: central !important;
  font-weight: 700 !important;
}

/* Putih untuk bar biru & hijau */
:deep(.apexcharts-series:nth-of-type(1) .apexcharts-data-label text),
:deep(.apexcharts-series:nth-of-type(2) .apexcharts-data-label text) {
  fill: #ffffff !important;
}

/* Hitam untuk bar kuning */
:deep(.apexcharts-series:nth-of-type(3) .apexcharts-data-label text) {
  fill: #111827 !important;
}

</style>
