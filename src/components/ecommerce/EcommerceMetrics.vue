<template>
  <!-- Wrapper: card group (stack di mobile, row di md+) -->
  <div class="w-full">
    <div
      class="bg-white rounded-2xl border border-gray-200 dark:border-gray-800 dark:bg-white/[0.03] overflow-hidden"
    >
      <!-- Use flex column on small screens, row on md, add vertical separators on md+ -->
      <div class="flex flex-col md:flex-row md:divide-x divide-gray-200">

        <!-- Card 1 -->
        <div class="flex-1 px-6 py-6 md:py-5 flex items-center gap-6">
          <div class="flex items-center justify-center w-12 h-12 bg-gray-100 rounded-xl dark:bg-gray-800">
            <svg class="w-6 h-6 text-gray-400 dark:text-gray-300" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
              <!-- outer pin/rocket shape -->
              <path
                d="M12 2C8.5 5.5 6 9 6 12.5C6 16.5 9.5 20 12 22C14.5 20 18 16.5 18 12.5C18 9 15.5 5.5 12 2Z"
                stroke="currentColor"
                stroke-width="1.5"
                stroke-linecap="round"
                stroke-linejoin="round"
                fill="none"
              />
              <!-- inner circle -->
              <circle cx="12" cy="10.5" r="2" fill="currentColor" />
              <!-- small tail detail to mimic pointed bottom -->
              <path d="M10.5 17.5c.7.3 1.5.5 1.5.5s.8-.2 1.5-.5" stroke="currentColor" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round" fill="none"/>
            </svg>
          </div>

          <div class="flex-1 flex items-start justify-between">
            <div>
              <div class="text-xs uppercase text-gray-400 tracking-wider font-semibold">Campaign Sent</div>
              <div class="mt-2 font-bold text-2xl text-gray-800 dark:text-white/90">197</div>
            </div>

            <div>
              <span class="inline-flex items-center gap-1 rounded-full bg-green-50 py-0.5 px-2 text-sm font-medium text-green-600 dark:bg-green-500/15 dark:text-green-400">
                <svg class="w-3 h-3" viewBox="0 0 12 12" fill="none" xmlns="http://www.w3.org/2000/svg">
                  <path d="M2 8l4-5 4 5" stroke="currentColor" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
                11.01%
              </span>
            </div>
          </div>
        </div>

        <!-- Card 2 -->
        <div class="flex-1 px-6 py-6 md:py-5 flex items-center gap-6">
          <div class="flex items-center justify-center w-12 h-12 bg-gray-100 rounded-xl dark:bg-gray-800">
            <svg class="w-6 h-6 text-gray-400 dark:text-gray-300" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <!-- Circular arrows -->
              <path
                d="M3 12a9 9 0 0 1 9-9c2.8 0 5.4 1.3 7.1 3.5M21 12a9 9 0 0 1-9 9c-2.8 0-5.4-1.3-7.1-3.5"
                stroke="currentColor"
                stroke-width="1.5"
                stroke-linecap="round"
                stroke-linejoin="round"
              />
              <path
                d="M16 6h3V3M8 18H5v3"
                stroke="currentColor"
                stroke-width="1.5"
                stroke-linecap="round"
                stroke-linejoin="round"
              />
              <!-- Dollar sign -->
              <path
                d="M12 8.5c-1.5 0-2.5.6-2.5 1.8 0 1.3 1 1.9 2.5 2.2s2.5.9 2.5 2.2c0 1.2-1 1.8-2.5 1.8m0-8v8"
                stroke="currentColor"
                stroke-width="1.5"
                stroke-linecap="round"
                stroke-linejoin="round"
              />
            </svg>

          </div>

          <div class="flex-1 flex items-start justify-between">
            <div>
              <div class="text-xs uppercase text-gray-400 tracking-wider font-semibold">Annual Profit</div>
              <div class="mt-2 font-bold text-2xl text-gray-800 dark:text-white/90">$489.4k</div>
            </div>

            <div>
              <span class="inline-flex items-center gap-1 rounded-full bg-green-50 py-0.5 px-2 text-sm font-medium text-green-600 dark:bg-green-500/15 dark:text-green-400">
                <svg class="w-3 h-3" viewBox="0 0 12 12" fill="none" xmlns="http://www.w3.org/2000/svg">
                  <path d="M2 8l4-5 4 5" stroke="currentColor" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
                7.8%
              </span>
            </div>
          </div>
        </div>

        <!-- Card 3 -->
        <div class="flex-1 px-6 py-6 md:py-5 flex items-center gap-6">
          <div class="flex items-center justify-center w-12 h-12 bg-gray-100 rounded-xl dark:bg-gray-800">
            <svg class="w-6 h-6 text-gray-400 dark:text-gray-300" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M3 12h3l2-6 3 12 2-6h3" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
            </svg>
          </div>

          <div class="flex-1 flex items-start justify-between">
            <div>
              <div class="text-xs uppercase text-gray-400 tracking-wider font-semibold">Lead Conversion</div>
              <div class="mt-2 font-bold text-2xl text-gray-800 dark:text-white/90">32.89%</div>
            </div>

            <div>
              <span class="inline-flex items-center gap-1 rounded-full bg-red-50 py-0.5 px-2 text-sm font-medium text-red-600 dark:bg-red-500/15 dark:text-red-400">
                <svg class="w-3 h-3" viewBox="0 0 12 12" fill="none" xmlns="http://www.w3.org/2000/svg">
                  <path d="M2 4l4 4 4-4" stroke="currentColor" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
                1.9%
              </span>
            </div>
          </div>
        </div>

        <!-- Card 4 -->
        <div class="flex-1 px-6 py-6 md:py-5 flex items-center gap-6">
          <div class="flex items-center justify-center w-12 h-12 bg-gray-100 rounded-xl dark:bg-gray-800">
            <svg class="w-6 h-6 text-gray-400 dark:text-gray-300" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M7 3h10v4H7zM9 7v5a3 3 0 0 0 6 0V7" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
            </svg>
          </div>

          <div class="flex-1 flex items-start justify-between">
            <div>
              <div class="text-xs uppercase text-gray-400 tracking-wider font-semibold">Daily Avg Income</div>
              <div class="mt-2 font-bold text-2xl text-gray-800 dark:text-white/90">$1,596.5</div>
            </div>

            <div>
              <span class="inline-flex items-center gap-1 rounded-full bg-green-50 py-0.5 px-2 text-sm font-medium text-green-600 dark:bg-green-500/15 dark:text-green-400">
                <svg class="w-3 h-3" viewBox="0 0 12 12" fill="none" xmlns="http://www.w3.org/2000/svg">
                  <path d="M2 8l4-5 4 5" stroke="currentColor" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
                2.1%
              </span>
            </div>
          </div>
        </div>

        <!-- Card 5 -->
        <div class="flex-1 px-6 py-6 md:py-5 flex items-center gap-6">
          <div class="flex items-center justify-center w-12 h-12 bg-gray-100 rounded-xl dark:bg-gray-800">
            <svg class="w-6 h-6 text-gray-400 dark:text-gray-300" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M12 21s-7-4.35-9-7.2C1.5 11 3 6 8 6c2.5 0 3.9 1.9 4 2 .1-.1 1.5-2 4-2 5 0 6.5 5 5 7.8-2 2.85-9 7.2-9 7.2z" stroke="currentColor" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round" fill="none"/>
            </svg>
          </div>

          <div class="flex-1 flex items-start justify-between">
            <div>
              <div class="text-xs uppercase text-gray-400 tracking-wider font-semibold">Annual Deals</div>
              <div class="mt-2 font-bold text-2xl text-gray-800 dark:text-white/90">2,659</div>
            </div>

            <div>
              <span class="inline-flex items-center gap-1 rounded-full bg-red-50 py-0.5 px-2 text-sm font-medium text-red-600 dark:bg-red-500/15 dark:text-red-400">
                <svg class="w-3 h-3" viewBox="0 0 12 12" fill="none" xmlns="http://www.w3.org/2000/svg">
                  <path d="M2 4l4 4 4-4" stroke="currentColor" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
                4.5%
              </span>
            </div>
          </div>
        </div>

      </div> <!-- end flex row -->
    </div> <!-- end outer card -->
  </div>
</template>
