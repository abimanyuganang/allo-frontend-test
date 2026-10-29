
<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { getRockets } from '../api/rockets'
import type { Rocket } from '../types/rocket'

const rockets = ref<Rocket[]>([])
const loading = ref(false)
const error = ref<string | null>(null)

async function fetchRockets() {
  loading.value = true
  error.value = null

  try {
    rockets.value = await getRockets()
  } catch (err) {
    error.value = err instanceof Error
      ? err.message
      : 'Something went wrong'
  } finally {
    loading.value = false
  }
}

onMounted(fetchRockets)
</script>

<template>
  <v-container>
    <h1>SpaceX Rockets</h1>

    <v-progress-circular
      v-if="loading"
      indeterminate
      color="primary"
    />

    <v-alert v-else-if="error" type="error">
      {{ error }}
      <v-btn @click="fetchRockets">Retry</v-btn>
    </v-alert>

    <div v-else>
      <p>Found {{ rockets.length }} rockets</p>

      <v-card
        v-for="rocket in rockets"
        :key="rocket.id"
        class="my-3 pa-4"
      >
        <h3>{{ rocket.full_name || 'Unnamed Rocket' }}</h3>
        <p>{{ rocket.description || 'No description available' }}</p>
      </v-card>
    </div>
  </v-container>
</template>
