<script setup lang="ts">
import { ref } from 'vue';
import type { Ref } from 'vue';

const DEFAULT_MONITORS = [
  { id: 1, api: 'stub/v1/api/super/test', status: 'on' },
  { id: 2, api: 'stub/v31/api/atom/witch', status: 'off' }
]
const monitors = ref(DEFAULT_MONITORS);
const api = ref('');
const status: Ref<number | null> = ref(null);

const ping = async () => {
  const response = await fetch(api.value);
  status.value = response.status;
  setTimeout(() => status.value = null, 3000)
}
</script>

<template>
  <form @submit.prevent="ping" style="display: flex; gap: 4px; margin-bottom: 2rem">
    <label for="api">
      Api
      <input v-model="api" id="api" type="text" />
    </label>
    <button type="submit">Check</button>
    <button type="button">Add to auto monitoring</button>
    <div>{{ status }}</div>
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
      <tr v-for="monitor in monitors">
        <td>{{ monitor.api }}</td>
        <td>{{ monitor.status }}</td>
      </tr>
    </tbody>
  </table>
</template>

<style scoped></style>
