<template>
  <div class="bg-white rounded-lg shadow p-4 flex flex-col items-center justify-center transition-all duration-300 hover:shadow-md border border-gray-100">
    <div class="text-sm font-semibold text-indigo-600 uppercase tracking-wide mb-1">{{ title }}</div>
    
    <!-- <div v-if="isLoading" class="flex items-center justify-center h-16">
      <div class="animate-spin rounded-full h-8 w-8 border-b-2 border-indigo-500"></div>
    </div> -->
    
    <div class="text-center">
      <div class="text-3xl font-bold mb-1 text-gray-800">{{ value || '0' }}</div>
      <div class="text-xs text-gray-500">{{ unit }}</div>
    </div>
    
    <div class="mt-3">
      <div v-if="icon === 'download'" class="w-6 h-6 text-indigo-500">
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4" />
        </svg>
      </div>
      <div v-else-if="icon === 'upload'" class="w-6 h-6 text-indigo-500">
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-8l-4-4m0 0L8 8m4-4v12" />
        </svg>
      </div>
      <div v-else-if="icon === 'activity'" class="w-6 h-6 text-indigo-500">
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 01-2 2h-2a2 2 0 01-2-2z" />
        </svg>
      </div>
    </div>
  </div>
</template>

<script setup>
defineProps({
  title: {
    type: String,
    required: true
  },
  value: {
    type: [Number, String],
    default: 0
  },
  unit: {
    type: String,
    default: ''
  },
  isLoading: {
    type: Boolean,
    default: false
  },
  icon: {
    type: String,
    default: ''
  }
});
</script>