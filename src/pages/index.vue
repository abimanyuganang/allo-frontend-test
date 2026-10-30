
<script setup lang="ts">
import { computed, onMounted, ref } from 'vue'
import { getRockets } from '../api/rockets'
import type { Rocket } from '../types/rocket'
import RocketCard from '../components/RocketCard.vue'
import { useRocketStore } from '../stores/rockets'
import RocketForm from '../components/RocketForm.vue'

const rockets = ref<Rocket[]>([])
const loading = ref(false)
const error = ref<string | null>(null)
const search = ref('')

const rocketStore = useRocketStore()

// Combine locally added rockets with API rockets
const allRockets = computed(() => [
  ...rocketStore.localRockets,
  ...rockets.value,
])

// Filter all rockets by name
const filteredRockets = computed(() => {
  const query = search.value.trim().toLowerCase()

  return allRockets.value.filter(rocket =>
    (rocket.full_name || '').toLowerCase().includes(query)
  )
})

// Fetch rockets from the API
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
    <!-- Header -->
    <div class="d-flex align-center justify-space-between flex-wrap ga-4 mb-8">
      <div>
        <h1 class="text-h4 font-weight-bold mb-2">
          SpaceX Rockets
        </h1>

        <p class="text-body-1 text-medium-emphasis">
          Explore SpaceX rocket configurations.
        </p>
      </div>

      <RocketForm />
    </div>

    <!-- Search -->
    <v-text-field
      v-model="search"
      label="Search rockets"
      prepend-inner-icon="mdi-magnify"
      variant="outlined"
      clearable
      hide-details
      class="mb-6"
    />

    <!-- Loading -->
    <v-row v-if="loading && !allRockets.length">
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

    <template v-else>
      <!-- API error, without hiding local rockets -->
      <v-alert
        v-if="error"
        type="error"
        variant="tonal"
        class="mb-6"
      >
        {{ error }}

        <div class="mt-3">
          <v-btn
            color="error"
            variant="outlined"
            @click="fetchRockets"
          >
            Retry
          </v-btn>
        </div>
      </v-alert>

      <!-- Results counter -->
      <p class="mb-4 text-medium-emphasis">
        Showing {{ filteredRockets.length }} of
        {{ allRockets.length }} rockets
      </p>

      <!-- Rocket cards -->
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

      <!-- Empty state -->
      <v-alert
        v-else-if="!error"
        type="info"
        variant="tonal"
      >
        No rockets found. Try another search.
      </v-alert>
    </template>
  </v-container>
</template>
