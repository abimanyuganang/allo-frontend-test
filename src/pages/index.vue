<script setup lang="ts">
import { computed, onMounted, ref } from 'vue'
import { getRockets } from '../api/rockets'
import type { Rocket } from '../types/rocket'
import RocketCard from '../components/RocketCard.vue'

const rockets = ref<Rocket[]>([])
const loading = ref(false)
const error = ref<string | null>(null)
const search = ref('')

const filteredRockets = computed(() => {
  const query = search.value.trim().toLowerCase()

  return rockets.value.filter((rocket) =>
    (rocket.full_name || '')
      .toLowerCase()
      .includes(query)
  )
})

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
  <v-container class="py-8">
    <div class="mb-8">
      <h1 class="text-h4 font-weight-bold mb-2">
        SpaceX Rockets
      </h1>

      <p class="text-body-1 text-medium-emphasis">
        Explore SpaceX rocket configurations.
      </p>
    </div>

    <v-text-field
      v-model="search"
      label="Search rockets"
      prepend-inner-icon="mdi-magnify"
      variant="outlined"
      clearable
      hide-details
      class="mb-6"
    />

    <div v-if="loading">
      <v-row>
        <v-col
          v-for="n in 6"
          :key="n"
          cols="12"
          sm="6"
          md="4"
        >
          <v-skeleton-loader
            type="image, article, actions"
          />
        </v-col>
      </v-row>
    </div>

    <v-alert
      v-else-if="error"
      type="error"
      variant="tonal"
      class="mb-6"
    >
      {{ error }}
      <div class="mt-3">
        <v-btn color="error" @click="fetchRockets">
          Retry
        </v-btn>
      </div>
    </v-alert>

    <template v-else>
      <p class="mb-4 text-medium-emphasis">
        Showing {{ filteredRockets.length }} of
        {{ rockets.length }} rockets
      </p>

      <v-row v-if="filteredRockets.length">
        <v-col
          v-for="rocket in filteredRockets"
          :key="rocket.id"
          cols="12"
          sm="6"
          md="4"
        >
          <RocketCard :rocket="rocket" />
        </v-col>
      </v-row>

      <v-alert v-else type="info" variant="tonal">
        No rockets found. Try another search.
      </v-alert>
    </template>
  </v-container>
</template>
