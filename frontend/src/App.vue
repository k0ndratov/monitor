<script setup lang="ts">
import { ref, onBeforeUnmount } from 'vue';
import { v4 as uuid } from 'uuid';
import type { Ref } from 'vue';

type ApiStatus = 'on' | 'off' | 'unknown';

interface Monitor {
  id: string;
  api: string;
  status: ApiStatus;
  lastUpdate?: Date;
}

const DEFAULT_MONITORS: Monitor[] = [
  { id: uuid(), api: 'stub/v1/api/super/test', status: 'on' },
  { id: uuid(), api: 'stub/v31/api/atom/witch', status: 'off' }
]
const monitors: Ref<Monitor[]> = ref([]);
const apiUrl = ref('');
const status: Ref<string | null> = ref(null);
const error: Ref<string | null> = ref(null);
const isLoading = ref(false);
const timers: Ref<number[]> = ref([]);

const addMonitor = (url: string) => {
  const newMonitor: Monitor = { id: uuid(), api: url, status: 'unknown' }
  monitors.value.push(newMonitor);
};

const removeMonitor = (id: string) => {
  monitors.value = monitors.value.filter((monitor) => monitor.id !== id)
}

const ping = async () => {
  try {
    isLoading.value = true;
    status.value = await checkApi(apiUrl.value);
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

const statusToString = (status: number | null): ApiStatus => {
  if (!status) return 'unknown'

  return status === 200 ? 'on' : 'off'
}

const checkApi = async (apiUrl: string): Promise<ApiStatus> => {
  const response = await fetch(apiUrl);
  return statusToString(response.status);
}

DEFAULT_MONITORS.forEach((monitor) => {
  addMonitor(monitor.api);
})

const interval = setInterval(async () => {
  const promises = monitors.value.map(async (monitor) => {
    let status: ApiStatus = 'unknown';

    try {
      status = await checkApi(monitor.api);
    } catch (e) {
      console.error(e);
    } finally {
      monitor.status = status;
    }

    monitor.lastUpdate = new Date();
  })

  await Promise.all(promises);
}, 2000)

onBeforeUnmount(() => {
  timers.value.forEach((timer) => {
    clearTimeout(timer);
  })
  clearInterval(interval);
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
    <button @click="addMonitor(apiUrl)" type="button">Add to auto monitoring</button>
    <div v-if="status">{{ status }}</div>
    <div v-if="error">Error: {{ error }}</div>
  </form>
  <table style="border: 1px solid black">
    <caption>Monitors</caption>
    <thead>
      <tr>
        <th>Api</th>
        <th>Status</th>
        <th>Last update</th>
        <th></th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="monitor in monitors" :key="monitor.id">
        <td>{{ monitor.api }}</td>
        <td>{{ monitor.status }}</td>
        <td>{{ monitor.lastUpdate }}</td>
        <td><button @click="removeMonitor(monitor.id)">X</button></td>
      </tr>
    </tbody>
  </table>
</template>

<style scoped>
[inert] > * {
  opacity: 0.5;
}
</style>
