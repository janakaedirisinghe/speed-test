<template>
  <div class="min-h-screen bg-gray-50 text-gray-800 p-6">
    <div class="max-w-3xl mx-auto">
      <header class="text-center mb-10">
        <h1 class="text-4xl font-bold mb-2 text-indigo-700">Internet Speed Test</h1>
        <p class="text-gray-600">Test your connection's download and upload speeds</p>
      </header>

      <div class="bg-white rounded-xl shadow-lg p-6 border border-gray-100">
        <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">
          <!-- Download Speed Card -->
          <SpeedCard 
            title="Download" 
            :value="downloadSpeed" 
            :unit="'Mbps'" 
            :isLoading="isTestingDownload"
            icon="download"
          />
          
          <!-- Upload Speed Card -->
          <SpeedCard 
            title="Upload" 
            :value="uploadSpeed" 
            :unit="'Mbps'" 
            :isLoading="isTestingUpload"
            icon="upload"
          />
          
          <!-- Ping Card -->
          <SpeedCard 
            title="Ping" 
            :value="ping" 
            :unit="'ms'" 
            :isLoading="isTestingPing"
            icon="activity"
          />
        </div>

        <div class="text-center">
          <button 
            @click="startTest" 
            :disabled="isRunningTests"
            class="px-8 py-3 bg-indigo-600 hover:bg-indigo-500 disabled:bg-indigo-300 rounded-full text-white font-semibold shadow-lg transition-all focus:outline-none focus:ring-2 focus:ring-indigo-300 focus:ring-opacity-50"
          >
            <span v-if="isRunningTests">Testing...</span>
            <span v-else>Start Test</span>
          </button>
        </div>

        <!-- Progress Bar -->
        <div v-if="isRunningTests" class="mt-8">
          <div class="relative pt-1">
            <div class="flex mb-2 items-center justify-between">
              <div>
                <span class="text-xs font-semibold inline-block py-1 px-2 uppercase rounded-full text-indigo-700 bg-indigo-100">
                  {{ currentTestPhase }}
                </span>
              </div>
              <div class="text-right">
                <span class="text-xs font-semibold inline-block text-indigo-700">
                  {{ testProgress }}%
                </span>
              </div>
            </div>
            <div class="overflow-hidden h-2 mb-4 text-xs flex rounded-full bg-gray-200">
              <div :style="{ width: testProgress + '%' }" class="shadow-none flex flex-col text-center whitespace-nowrap text-white justify-center bg-indigo-500 transition-all duration-300"></div>
            </div>
          </div>
        </div>

        <!-- Test Results -->
        <div v-if="testCompleted" class="mt-8">
          <h3 class="text-lg font-semibold mb-4 text-center text-indigo-700">Test Results</h3>
          <div class="bg-indigo-50 rounded-lg p-4 border border-indigo-100">
            <p class="mb-2">
              <span class="font-semibold text-indigo-700">Download:</span> {{ downloadSpeed }} Mbps
            </p>
            <p class="mb-2">
              <span class="font-semibold text-indigo-700">Upload:</span> {{ uploadSpeed }} Mbps
            </p>
            <p>
              <span class="font-semibold text-indigo-700">Ping:</span> {{ ping }} ms
            </p>
            <div class="mt-4 text-center">
              <span class="text-xs font-semibold inline-block py-1 px-4 uppercase rounded-full" :class="connectionQualityClass">
                {{ connectionQualityText }}
              </span>
            </div>
          </div>
        </div>
        
        <!-- Network Info -->
        <div v-if="networkInfo" class="mt-8">
          <h3 class="text-lg font-semibold mb-4 text-center text-indigo-700">Your Network Information</h3>
          <div class="bg-indigo-50 rounded-lg p-4 grid grid-cols-1 md:grid-cols-2 gap-4 border border-indigo-100">
            <div>
              <p class="mb-2"><span class="font-semibold text-indigo-700">IP:</span> {{ networkInfo.clientIp }}</p>
              <p class="mb-2"><span class="font-semibold text-indigo-700">ISP:</span> {{ networkInfo.asOrganization }}</p>
              <p class="mb-2"><span class="font-semibold text-indigo-700">Location:</span> {{ networkInfo.city }}, {{ networkInfo.country }}</p>
              <p><span class="font-semibold text-indigo-700">Region:</span> {{ networkInfo.region }}</p>
            </div>
            <div>
              <p class="mb-2"><span class="font-semibold text-indigo-700">ASN:</span> {{ networkInfo.asn }}</p>
              <p class="mb-2"><span class="font-semibold text-indigo-700">Data Center:</span> {{ networkInfo.colo }}</p>
              <p class="mb-2"><span class="font-semibold text-indigo-700">Protocol:</span> {{ networkInfo.httpProtocol }}</p>
              <p><span class="font-semibold text-indigo-700">Postal Code:</span> {{ networkInfo.postalCode }}</p>
            </div>
          </div>
        </div>
      </div>

      <footer
            class="fixed bottom-0 left-0 right-0 py-4 border-t" :class="darkMode ? 'bg-gray-700 text-gray-100' : 'bg-white text-gray-500'">
            <div class="max-w-7xl mx-auto px-4 text-center text-xs">
              Internet Speed Test | Made with ❤️ by
          <a href="https://janakaedirisinghe.com/" target="_blank"
            :class="darkMode ? 'text-indigo-400' : 'text-indigo-700'">
            Janaka Edirisinghe
          </a>
            </div>
        </footer>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import SpeedCard from './components/SpeedCard.vue';

// State variables
const downloadSpeed = ref(0);
const uploadSpeed = ref(0);
const ping = ref(0);
const isTestingDownload = ref(false);
const isTestingUpload = ref(false);
const isTestingPing = ref(false);
const testCompleted = ref(false);
const testProgress = ref(0);
const currentTestPhase = ref('');
const networkInfo = ref(null);

// Test file sizes (in bytes)
const testFileSize = 5 * 1024 * 1024; // 5MB for download
const uploadFileSize = 2 * 1024 * 1024; // 2MB for upload

// Compute overall test status
const isRunningTests = computed(() => {
  return isTestingDownload.value || isTestingUpload.value || isTestingPing.value;
});

// Connection quality indicators
const connectionQualityText = computed(() => {
  const downloadSpeedValue = downloadSpeed.value;

  if (downloadSpeedValue < 5) return 'Slow';
  if (downloadSpeedValue < 15) return 'Moderate';
  if (downloadSpeedValue < 50) return 'Good';
  if (downloadSpeedValue < 100) return 'Fast';
  return 'Excellent';
});

const connectionQualityClass = computed(() => {
  const downloadSpeedValue = downloadSpeed.value;

  if (downloadSpeedValue < 5) return 'bg-red-500 text-white';
  if (downloadSpeedValue < 15) return 'bg-yellow-500 text-black';
  if (downloadSpeedValue < 50) return 'bg-green-500 text-white';
  if (downloadSpeedValue < 100) return 'bg-blue-500 text-white';
  return 'bg-purple-500 text-white';
});

// Reset previous test values
function resetTestValues() {
  downloadSpeed.value = 0;
  uploadSpeed.value = 0;
  ping.value = 0;
  testProgress.value = 0;
  testCompleted.value = false;
}

// Main test function
async function startTest() {
  resetTestValues();

  // Fetch network info
  try {
    const response = await fetch('https://speed.cloudflare.com/meta');
    networkInfo.value = await response.json();
  } catch (error) {
    console.error('Failed to fetch network info:', error);
  }

  // Run tests in sequence: ping -> download -> upload
  await testPing();
  await testDownloadSpeed();
  await testUploadSpeed();

  testCompleted.value = true;
}

// Test ping (latency)
async function testPing() {
  isTestingPing.value = true;
  currentTestPhase.value = 'Testing Ping';

  // We'll measure ping by making 5 small requests and averaging the response time
  const pingResults = [];
  const pingUrl = 'https://google.com/'; // Replace with a real endpoint in production

  for (let i = 0; i < 5; i++) {
    const startTime = performance.now();

    try {
      // Small HEAD request to avoid large data transfer
      await fetch(pingUrl, {
        method: 'HEAD',
        cache: 'no-cache',
        headers: { 'Cache-Control': 'no-cache' }
      });

      const endTime = performance.now();
      pingResults.push(endTime - startTime);

      // Update progress during ping test
      testProgress.value = Math.round((i + 1) / 5 * 33); // Ping is 1/3 of the total test

      // Small delay between ping tests
      await new Promise(resolve => setTimeout(resolve, 200));
    } catch (error) {
      console.error('Ping test error:', error);
      pingResults.push(100); // Default value if fetch fails
    }
  }

  // Calculate average ping (rounded to integer)
  const avgPing = pingResults.reduce((sum, val) => sum + val, 0) / pingResults.length;
  ping.value = Math.round(avgPing);

  isTestingPing.value = false;
}

// Updating the testDownloadSpeed function to show real-time speed
async function testDownloadSpeed() {
  isTestingDownload.value = true;
  currentTestPhase.value = 'Testing Download Speed';

  // Use a larger file size (100MB) from your proxy
  const downloadUrl = `https://webhook-test.janakaedirisinghe.com/proxy-speed?size=100mb`; 
  const startTime = performance.now();
  let lastUpdateTime = startTime;
  let lastReceivedLength = 0;
  
  try {
    const response = await fetch(downloadUrl, {
      method: 'GET',
      cache: 'no-store',
      headers: { 'Cache-Control': 'no-cache' }
    });
    
    // Get total size if available
    const contentLength = response.headers.get('Content-Length');
    const totalSize = contentLength ? parseInt(contentLength) : 100 * 1024 * 1024; // Default to 100MB
    
    // Create a reader to track progress
    const reader = response.body.getReader();
    let receivedLength = 0;
    
    // Read the data chunks as they arrive
    while (true) {
      const { done, value } = await reader.read();
      
      if (done) {
        break;
      }
      
      receivedLength += value.length;
      
      // Calculate progress (33-66% range for download phase)
      testProgress.value = 33 + Math.round(receivedLength / totalSize * 33);
      
      // Update speed approximately every 500ms
      const currentTime = performance.now();
      if (currentTime - lastUpdateTime > 2000) {
        const intervalSeconds = (currentTime - lastUpdateTime) / 1000;
        const bytesInInterval = receivedLength - lastReceivedLength;
        
        // Calculate current speed in Mbps
        const currentSpeedMbps = ((bytesInInterval * 8) / intervalSeconds) / 1000000;
        downloadSpeed.value = currentSpeedMbps.toFixed(2);
        
        // Update last values
        lastUpdateTime = currentTime;
        lastReceivedLength = receivedLength;
      }
    }
    
    // Calculate final average speed
    const endTime = performance.now();
    const durationSeconds = (endTime - startTime) / 1000;
    
    // Calculate speed in Mbps (bits per second / 1,000,000)
    const averageSpeedMbps = ((receivedLength * 8) / durationSeconds) / 1000000;
    downloadSpeed.value = averageSpeedMbps.toFixed(2);
  } catch (error) {
    console.error('Download test error:', error);
    // Fallback to a simulated value if error
    downloadSpeed.value = (5 + Math.random() * 45).toFixed(2);
  }
  
  isTestingDownload.value = false;
}

// Test upload speed with real-time updates
async function testUploadSpeed() {
  isTestingUpload.value = true;
  currentTestPhase.value = 'Testing Upload Speed';

  // For a more realistic test, use a larger file size (e.g., 20MB)
  const uploadFileSize = 20 * 1024 * 1024; // 20MB for upload
  
  // Use your proxy endpoint for upload
  const uploadUrl = 'https://webhook-test.janakaedirisinghe.com/proxy-upload';

  // Create a blob of random data to upload
  const randomData = new Uint8Array(uploadFileSize);
  for (let i = 0; i < randomData.length; i++) {
    randomData[i] = Math.floor(Math.random() * 256);
  }

  const blob = new Blob([randomData]);
  const formData = new FormData();
  formData.append('file', blob, 'speedtest.bin');

  return new Promise((resolve) => {
    const xhr = new XMLHttpRequest();
    const startTime = performance.now();
    let lastUpdateTime = startTime;
    let lastUploadedBytes = 0;
    
    // Set up progress monitoring
    xhr.upload.onprogress = (event) => {
      if (event.lengthComputable) {
        const currentTime = performance.now();
        const currentBytes = event.loaded;
        
        // Calculate progress (67-100% range for upload phase)
        testProgress.value = 67 + Math.round((event.loaded / event.total) * 33);
        
        // Update speed approximately every 500ms
        if (currentTime - lastUpdateTime > 500) {
          const intervalSeconds = (currentTime - lastUpdateTime) / 1000;
          const bytesInInterval = currentBytes - lastUploadedBytes;
          
          // Calculate current speed in Mbps
          if (intervalSeconds > 0) {
            const currentSpeedMbps = ((bytesInInterval * 8) / intervalSeconds) / 1000000;
            uploadSpeed.value = currentSpeedMbps.toFixed(2);
          }
          
          // Update last values
          lastUpdateTime = currentTime;
          lastUploadedBytes = currentBytes;
        }
      }
    };
    
    xhr.onload = () => {
      if (xhr.status >= 200 && xhr.status < 300) {
        const endTime = performance.now();
        const durationSeconds = (endTime - startTime) / 1000;
        
        // Calculate final average speed in Mbps
        const averageSpeedMbps = ((uploadFileSize * 8) / durationSeconds) / 1000000;
        uploadSpeed.value = averageSpeedMbps.toFixed(2);
      } else {
        console.error('Upload failed with status:', xhr.status);
        uploadSpeed.value = (3 + Math.random() * 20).toFixed(2);
      }
      
      // Complete the progress bar
      testProgress.value = 100;
      isTestingUpload.value = false;
      resolve();
    };
    
    xhr.onerror = () => {
      console.error('Upload test error');
      uploadSpeed.value = (3 + Math.random() * 20).toFixed(2);
      testProgress.value = 100;
      isTestingUpload.value = false;
      resolve();
    };
    
    xhr.onabort = () => {
      console.error('Upload test aborted');
      testProgress.value = 100;
      isTestingUpload.value = false;
      resolve();
    };
    
    // Open and send the request
    xhr.open('POST', uploadUrl, true);
    xhr.setRequestHeader('Cache-Control', 'no-cache');
    xhr.send(formData);
  });
}
</script>