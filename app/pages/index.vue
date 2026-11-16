<script setup lang="ts">
import { useWeather } from '../composables/useWeather'

definePageMeta({ title: 'Welcome' })
const user = useAuthUser()
const { fetchWeather } = useWeather()
const weather = ref<Awaited<ReturnType<typeof fetchWeather>> | null>(null)
const weatherError = ref('')
const weatherLoading = ref(true)

onMounted(async () => {
  try {
    weather.value = await fetchWeather()
  } catch {
    weatherError.value = 'Unable to load weather right now.'
  } finally {
    weatherLoading.value = false
  }
})
</script>

<template>
  <div>
    <v-row>
      <v-col cols="12">
        <v-card class="pa-6" rounded="lg">
          <div class="d-flex align-center mb-4" v-if="user">
            <v-avatar size="56" class="mr-4">
              <v-img :src="user.picture" :alt="user.name" />
            </v-avatar>
            <div>
              <div class="text-h5 font-weight-bold">
                Welcome, {{ user.name }} 👋
              </div>
              <div class="text-body-2 text-medium-emphasis">
                Here's a quick overview of what you can do.
              </div>
            </div>
          </div>
        </v-card>
      </v-col>
    </v-row>

    <v-row class="mt-2">
      <v-col cols="12" md="6">
        <v-card to="/scan" hover rounded="lg" class="pa-4" height="100%">
          <v-icon size="36" color="primary" class="mb-2">mdi-qrcode-scan</v-icon>
          <div class="text-h6 font-weight-medium">Scan</div>
          <div class="text-body-2 text-medium-emphasis">
            Quick single-shot scan using your browser's built-in scanner.
          </div>
        </v-card>
      </v-col>

      <v-col cols="12" md="6">
        <v-card to="/weather" hover rounded="lg" class="pa-4" height="100%">
          <div class="d-flex align-center justify-space-between mb-2">
            <div class="text-h6 font-weight-medium">Weather</div>
            <v-icon size="36" color="primary">mdi-weather-partly-cloudy</v-icon>
          </div>
          <div v-if="weatherLoading" class="text-body-2 text-medium-emphasis">
            Loading current weather...
          </div>
          <v-alert v-else-if="weatherError" type="error" variant="tonal">
            {{ weatherError }}
          </v-alert>
          <template v-else-if="weather">
            <div class="text-h4 font-weight-bold">
              {{ Math.round(weather.current.temp_c) }}°C
            </div>
            <div class="text-body-1">{{ weather.current.condition.text }}</div>
            <div class="text-body-2 text-medium-emphasis">
              {{ weather.location.name }}, {{ weather.location.region }}
            </div>
          </template>
        </v-card>
      </v-col>
    </v-row>
  </div>
</template>
