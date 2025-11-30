<script setup lang="ts">
import { ref, onBeforeUnmount } from 'vue';
import type { Ref } from 'vue';

interface Monitor {
  id: number;
  api: string;
  status: 'on' | 'off';
}

const DEFAULT_MONITORS: Monitor[] = [
  { id: 1, api: 'stub/v1/api/super/test', status: 'on' },
  { id: 2, api: 'stub/v31/api/atom/witch', status: 'off' }
]
const monitors: Ref<Monitor[]> = ref(DEFAULT_MONITORS);
const apiUrl = ref('');
const status: Ref<number | null> = ref(null);
const error: Ref<string | null> = ref(null);
const isLoading = ref(false);
const timers: Ref<number[]> = ref([]);
const intervals: Ref<number[]> = ref([]);

const addMonitor = () => {
  const newMonitor: Monitor = { id: monitors.value.length + 1, api: apiUrl.value, status: 'off' }
  monitors.value.push(newMonitor);
  const interval = setInterval(async () => {
    const status = await fetchData(newMonitor.api);
    monitors.value.find((monitor) => monitor.id === newMonitor.id)!.status = status === 200 ? 'on' :
      'off';
  }, 2000)
  intervals.value.push(interval)
};

const ping = async () => {
  try {
    isLoading.value = true;
    const response = await fetch(apiUrl.value);
    status.value = response.status;
    await new Promise((resolve) => setTimeout(resolve, 2000));
    // For error check
    ar;
  } catch (e) {
    if (e instanceof Error) {
      error.value = e.message
    } else {
      error.value = String(e)
    }
  } finally {
    const timerId = setTimeout(() => {
      status.value = null;
      error.value = null;
    }, 3000)
    timers.value.push(timerId);
    isLoading.value = false
  }
}

const fetchData = async (apiUrl: string): Promise<number> => {
  const response = await fetch(apiUrl);
  return response.status;
}

monitors.value.forEach((monitor) => {
  const interval = setInterval(async () => {
    const status = await fetchData(monitor.api);
    monitor.status = status === 200 ? 'on' : 'off';
  }, 2000);

  intervals.value.push(interval);
})

onBeforeUnmount(() => {
  timers.value.forEach((timer) => {
    clearTimeout(timer);
  })
  intervals.value.forEach((interval) => {
    clearInterval(interval);
  })
})
</script>

<template>
  <form @submit.prevent="ping" style="display: flex; gap: 4px; margin-bottom: 2rem"
    :inert="isLoading">
    <label for="api">
      Api
      <input v-model="apiUrl" id="api" type="text" required />
    </label>
    <button type="submit">Check</button>
    <button @click="addMonitor" type="button">Add to auto monitoring</button>
    <div v-if="status">{{ status }}</div>
    <div v-if="error">Error: {{ error }}</div>
  </form>
  <table style="border: 1px solid black">
    <caption>Monitors</caption>
    <thead>
      <tr>
        <th>Api</th>
        <th>Status</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="monitor in monitors" :key="monitor.id">
        <td>{{ monitor.api }}</td>
        <td>{{ monitor.status }}</td>
      </tr>
    </tbody>
  </table>
</template>

<style scoped>
[inert] > * {
  opacity: 0.5;
}
</style>
